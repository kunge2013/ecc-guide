# 场景 3: API 设计一致性改进

这个场景展示如何通过 Code Review 发现和改进 API 设计问题。

## 原始代码（API 设计不一致）

```javascript
// file: src/api/user-controller.js

class UserController {
  constructor(userService) {
    this.userService = userService;
  }

  // ❌ 问题 1: 返回格式不一致（有时返回数据，有时返回包装对象）
  async getUser(req, res) {
    const user = await this.userService.findById(req.params.id);
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    return res.json(user); // 直接返回用户对象
  }

  // ❌ 问题 2: 错误响应格式不统一
  async createUser(req, res) {
    try {
      const user = await this.userService.create(req.body);
      return res.status(201).json({
        data: user,
        message: 'User created successfully'
      });
    } catch (error) {
      return res.status(400).json({
        success: false,
        error: error.message
      });
    }
  }

  // ❌ 问题 3: 状态码使用不正确
  async updateUser(req, res) {
    try {
      const user = await this.userService.update(req.params.id, req.body);
      return res.json(user);
    } catch (error) {
      // 500 状态码太宽泛
      return res.status(500).json({
        error: 'Internal server error'
      });
    }
  }

  // ❌ 问题 4: 没有分页参数验证
  async listUsers(req, res) {
    const { page, limit } = req.query;

    const result = await this.userService.list({
      page: parseInt(page) || 1,
      limit: parseInt(limit) || 10
    });

    return res.json(result);
  }

  // ❌ 问题 5: 没有内容协商，总是返回 JSON
  async exportUsers(req, res) {
    const users = await this.userService.listAll();
    return res.json(users);
  }
}

// file: src/api/product-controller.js

class ProductController {
  constructor(productService) {
    this.productService = productService;
  }

  // ❌ 问题 6: 与 UserController 的返回格式完全不同
  async getProduct(req, res) {
    const product = await this.productService.findById(req.params.id);
    if (!product) {
      return res.status(404).send('Product not found'); // 返回字符串而不是 JSON
    }
    return res.json({
      status: 'ok',
      result: product,
      timestamp: Date.now() // 不必要的元数据
    });
  }

  // ❌ 问题 7: 使用不同的分页参数名
  async listProducts(req, res) {
    const { pageNum, pageSize } = req.query; // 不同的参数名

    const result = await this.productService.list({
      page: parseInt(pageNum) || 1,
      limit: parseInt(pageSize) || 10
    });

    return res.json({
      items: result.data, // 不同的字段名
      total: result.total,
      page: result.page,
      pageSize: result.limit
    });
  }
}
```

## Code Review 发现

### 🔴 CRITICAL 问题

**问题 1: API 响应格式不一致**

```markdown
文件: src/api/user-controller.js:7
级别: CRITICAL
类型: API 设计

问题:
getUser 返回原始对象，而 createUser 返回包装对象。
这导致客户端需要处理不同的响应格式，增加了复杂度。

影响:
- 客户端代码复杂
- 难以统一处理错误
- API 文档难以维护
- 客户端需要检查每个端点的返回格式

建议:
建立统一的 API 响应格式标准，所有端点都遵循相同模式。

标准格式:
成功响应:
{
  "data": <实际数据>,
  "meta": { <可选的元数据> }
}

错误响应:
{
  "error": {
    "code": "ERROR_CODE",
    "message": "人类可读的错误消息",
    "details": { <可选的详细信息> }
  }
}

修复方案:
// 创建响应工具类
class ApiResponse {
  static success(data, meta = {}) {
    return {
      data,
      meta
    };
  }

  static error(code, message, details = {}) {
    return {
      error: {
        code,
        message,
        details
      }
    };
  }
}

// 在控制器中使用
async getUser(req, res) {
  const user = await this.userService.findById(req.params.id);
  if (!user) {
    return res.status(404).json(
      ApiResponse.error('USER_NOT_FOUND', 'User not found')
    );
  }
  return res.json(ApiResponse.success(user));
}
```

**问题 2: HTTP 状态码使用不当**

```markdown
文件: src/api/user-controller.js:33
级别: CRITICAL
类型: API 设计

问题:
updateUser 在所有错误情况下都返回 500 状态码。
这使得客户端无法区分不同类型的错误（如验证错误、权限错误等）。

HTTP 状态码语义:
- 400: 客户端请求错误（如验证失败）
- 401: 未认证
- 403: 已认证但无权限
- 404: 资源不存在
- 409: 资源冲突
- 422: 验证失败
- 500: 服务器内部错误

建议:
根据错误类型返回适当的 HTTP 状态码。

修复方案:
async updateUser(req, res) {
  try {
    const user = await this.userService.update(req.params.id, req.body);
    return res.json(ApiResponse.success(user));
  } catch (error) {
    if (error instanceof ValidationError) {
      return res.status(400).json(
        ApiResponse.error('VALIDATION_ERROR', error.message, {
          fields: error.fields
        })
      );
    }

    if (error instanceof NotFoundError) {
      return res.status(404).json(
        ApiResponse.error('USER_NOT_FOUND', error.message)
      );
    }

    if (error instanceof AuthError) {
      return res.status(401).json(
        ApiResponse.error('UNAUTHORIZED', error.message)
      );
    }

    // 其他错误才是真正的服务器错误
    this.logger.error('Update user error', { error });
    return res.status(500).json(
      ApiResponse.error('INTERNAL_ERROR', 'Internal server error')
    );
  }
}
```

### 🟡 MEDIUM 问题

**问题 3: 控制器之间不一致**

```markdown
文件: src/api/product-controller.js:8
级别: MEDIUM
类型: API 设计

问题:
ProductController 使用完全不同的 API 设计风格：
- 返回格式不同
- 分页参数名不同
- 成功响应包含不必要的元数据

影响:
- API 使用体验不一致
- 客户端需要处理多种格式
- 难以维护

建议:
在团队中建立 API 设计规范，所有控制器都遵循相同模式。

修复方案:
// 创建基类控制器
class BaseController {
  constructor(service, logger) {
    this.service = service;
    this.logger = logger;
  }

  async getById(req, res) {
    try {
      const item = await this.service.findById(req.params.id);
      if (!item) {
        return this.notFound(res, 'Resource not found');
      }
      return this.success(res, item);
    } catch (error) {
      return this.handleError(res, error);
    }
  }

  async list(req, res) {
    try {
      const pagination = this.parsePagination(req.query);
      const result = await this.service.list(pagination);
      return this.success(res, result.data, {
        total: result.total,
        page: result.page,
        limit: result.limit
      });
    } catch (error) {
      return this.handleError(res, error);
    }
  }

  success(res, data, meta = {}) {
    return res.json(ApiResponse.success(data, meta));
  }

  notFound(res, message) {
    return res.status(404).json(
      ApiResponse.error('NOT_FOUND', message)
    );
  }

  handleError(res, error) {
    if (error instanceof ValidationError) {
      return res.status(400).json(
        ApiResponse.error('VALIDATION_ERROR', error.message)
      );
    }

    if (error instanceof NotFoundError) {
      return this.notFound(res, error.message);
    }

    this.logger.error('Controller error', { error });
    return res.status(500).json(
      ApiResponse.error('INTERNAL_ERROR', 'Internal server error')
    );
  }

  parsePagination(query) {
    return {
      page: Math.max(1, parseInt(query.page) || 1),
      limit: Math.min(100, Math.max(1, parseInt(query.limit) || 10))
    };
  }
}

// 用户控制器继承基类
class UserController extends BaseController {
  async getById(req, res) {
    return super.getById(req).catch(error =>
      this.handleError(res, error)
    );
  }

  async list(req, res) {
    return super.list(req, res);
  }
}
```

**问题 4: 缺少输入验证**

```markdown
文件: src/api/user-controller.js:45
级别: MEDIUM
类型: 输入验证

问题:
listUsers 直接使用 req.query 的值，没有验证或清理。
parseInt 可能返回 NaN，导致意外行为。

建议:
验证和清理所有输入参数。

修复方案:
async listUsers(req, res) {
  const pagination = this.parsePagination(req.query);
  const result = await this.userService.list(pagination);
  return this.success(res, result.data, {
    total: result.total,
    page: result.page,
    limit: result.limit
  });
}

// 在基类中实现 parsePagination
parsePagination(query) {
  // ✅ 验证和限制分页参数
  const page = parseInt(query.page);
  const limit = parseInt(query.limit);

  return {
    page: isNaN(page) ? 1 : Math.max(1, page),
    limit: isNaN(limit) ? 10 : Math.min(100, Math.max(1, limit))
  };
}
```

**问题 5: 没有内容协商**

```markdown
文件: src/api/user-controller.js:62
级别: LOW
类型: API 增强功能

问题:
exportUsers 总是返回 JSON，没有根据 Accept header 决定格式。

建议:
支持多种响应格式（JSON、CSV、XML）。

修复方案:
async exportUsers(req, res) {
  const users = await this.userService.listAll();
  const format = this.parseAcceptFormat(req);

  switch (format) {
    case 'csv':
      res.setHeader('Content-Type', 'text/csv');
      return res.send(this.toCsv(users));

    case 'json':
    default:
      return res.json(ApiResponse.success(users));
  }
}

parseAcceptFormat(req) {
  const accept = req.headers.accept || '';
  if (accept.includes('text/csv')) return 'csv';
  if (accept.includes('application/json')) return 'json';
  return 'json';
}

toCsv(data) {
  // 实现 CSV 转换
}
```

## 修复后的代码

```javascript
// file: src/api/api-response.js

class ApiResponse {
  static success(data, meta = {}) {
    return {
      data,
      meta: Object.keys(meta).length > 0 ? meta : undefined
    };
  }

  static error(code, message, details = {}) {
    return {
      error: {
        code,
        message,
        details: Object.keys(details).length > 0 ? details : : undefined
      }
    };
  }
}

// file: src/api/base-controller.js

class BaseController {
  constructor(service, logger) {
    this.service = service;
    this.logger = logger;
  }

  async getById(req, res) {
    try {
      const item = await this.service.findById(req.params.id);
      if (!item) {
        return this.notFound(res, this.resourceName + ' not found');
      }
      return this.success(res, item);
    } catch (error) {
      return this.handleError(res, error);
    }
  }

  async create(req, res) {
    try {
      const data = await this.validateCreate(req.body);
      const item = await this.service.create(data);
      return res.status(201).json(
        ApiResponse.success(item, { message: this.resourceName + ' created' })
      );
    } catch (error) {
      return this.handleError(res, error);
    }
  }

  async update(req, res) {
    try {
      const data = await this.validateUpdate(req.body);
      const item = await this.service.update(req.params.id, data);
      return this.success(res, item);
    } catch (error) {
      return this.handleError(res, error);
    }
  }

  async delete(req, res) {
    try {
      await this.service.delete(req.params.id);
      return res.status(204).send();
    } catch (error) {
      return this.handleError(res, error);
    }
  }

  async list(req, res) {
    try {
      const pagination = this.parsePagination(req.query);
      const result = await this.service.list(pagination);
      return this.success(res, result.data, {
        total: result.total,
        page: result.page,
        limit: result.limit
      });
    } catch (error) {
      return this.handleError(res, error);
    }
  }

  success(res, data, meta = {}) {
    return res.json(ApiResponse.success(data, meta));
  }

  notFound(res, message) {
    return res.status(404).json(
      ApiResponse.error('NOT_FOUND', message)
    );
  }

  handleError(res, error) {
    if (error instanceof ValidationError) {
      return res.status(400).json(
        ApiResponse.error('VALIDATION_ERROR', error.message, {
          fields: error.fields
        })
      );
    }

    if (error instanceof NotFoundError) {
      return this.notFound(res, error.message);
    }

    if (error instanceof AuthError) {
      return res.status(401).json(
        ApiResponse.error('UNAUTHORIZED', error.message)
      );
    }

    if (error instanceof ForbiddenError) {
      return res.status(403).json(
        ApiResponse.error('FORBIDDEN', error.message)
      );
    }

    this.logger.error('Controller error', {
      resource: this.resourceName,
      error: error.message,
      stack: error.stack
    });

    return res.status(500).json(
      ApiResponse.error('INTERNAL_ERROR', 'Internal server error')
    );
  }

  parsePagination(query) {
    const page = parseInt(query.page);
    const limit = parseInt(query.limit);

    return {
      page: isNaN(page) ? 1 : Math.max(1, page),
      limit: isNaN(limit) ? 10 : Math.min(100, Math.max(1, limit))
    };
  }

  async validateCreate(data) {
    return data; // 子类可以覆盖
  }

  async validateUpdate(data) {
    return data; // 子类可以覆盖
  }
}

// file: src/api/user-controller.js

class UserController extends BaseController {
  constructor(userService, logger) {
    super(userService, logger);
    this.resourceName = 'User';
  }

  async validateCreate(data) {
    // 特定的用户验证逻辑
    if (!data.email || !data.password) {
      throw new ValidationError('Email and password are required', {
        fields: ['email', 'password']
      });
    }
    return data;
  }
}

// file: src/api/product-controller.js

class ProductController extends BaseController {
  constructor(productService, logger) {
    super(productService, logger);
    this.resourceName = 'Product';
  }

  async list(req, res) {
    try {
      const pagination = this.parsePagination(req.query);
      const filters = this.parseFilters(req.query);
      const result = await this.service.list({ ...pagination, ...filters });
      return this.success(res, result.data, {
        total: result.total,
        page: result.page,
        limit: result.limit
      });
    } catch (error) {
      return this.handleError(res, error);
    }
  }

  parseFilters(query) {
    return {
      category: query.category,
      minPrice: parseFloat(query.minPrice),
      maxPrice: parseFloat(query.maxPrice)
    };
  }
}
```

## API 设计规范文档

```markdown
# API 设计规范

## 响应格式

### 成功响应

```json
{
  "data": <数据>,
  "meta": {
    "total": 100,
    "page": 1,
    "limit": 10
  }
}
```

### 错误响应

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "人类可读的消息",
    "details": {
      "field": "具体错误信息"
    }
  }
}
```

## HTTP 状态码

| 状态码 | 使用场景 | 错误码 |
|--------|----------|--------|
| 200 | 成功 GET/PUT/PATCH | - |
| 201 | 成功创建 POST | - |
| 204 | 成功删除 DELETE | - |
| 400 | 客户端错误 | VALIDATION_ERROR, INVALID_INPUT |
| 401 | 未认证 | UNAUTHORIZED |
| 403 | 已认证但无权限 | FORBIDDEN |
| 404 | 资源不存在 | NOT_FOUND |
| 409 | 资源冲突 | CONFLICT, DUPLICATE |
| 422 | 验证失败 | VALIDATION_ERROR |
| 500 | 服务器错误 | INTERNAL_ERROR |

## 分页参数

标准分页参数名：
- `page`: 页码（从 1 开始，默认 1）
- `limit`: 每页数量（默认 10，最大 100）

响应包含：
- `meta.total`: 总记录数
- `meta.page`: 当前页
- `meta.limit`: 每页数量

## 命名约定

- 路径: kebab-case (`/users/`, `/products/`)
- 查询参数: kebab-case (`?sort-by=name&order=asc`)
- JSON 字段: camelCase (`firstName`, `lastName`)
- 错误码: SCREAMING_SNAKE_CASE (`USER_NOT_FOUND`)
```

## 学习要点

### 1. API 一致性的重要性

**好处：**
- 客户端开发更容易
- 减少文档复杂度
- 降低集成错误
- 提高用户体验

### 2. HTTP 状态码最佳实践

**正确使用：**
- ✅ 使用语义化的状态码
- ✅ 4xx 表示客户端错误
- ✅ 5xx 表示服务器错误
- ✅ 错误响应包含可操作的信息

**避免：**
- ❌ 总是返回 200，错误放在响应体中
- ❌ 所有错误都返回 500
- ❌ 使用非标准的状态码

### 3. API 版本控制

```javascript
// URL 版本控制
app.use('/api/v1/users', userRoutesV1);
app.use('/api/v2/users', userRoutesV2);

// Header 版本控制
app.use('/api/users', (req, res, next) => {
  const version = req.headers['api-version'] || 'v1';
  if (version === 'v2') {
    return userRoutesV2(req, res, next);
  }
  return userRoutesV1(req, res, next);
});
```

---

**关键教训：** API 设计的一致性对系统的长期维护和客户端开发至关重要。建立明确的 API 设计规范并使用代码模式（如基类控制器）确保一致性。
