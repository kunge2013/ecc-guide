# 场景 1: SQL 注入漏洞修复

这个场景展示如何通过 Code Review 发现并修复严重的安全漏洞。

## 原始代码（有 SQL 注入漏洞）

```javascript
// file: src/user/user-repository.js

class UserRepository {
  constructor(db) {
    this.db = db;
  }

  // ❌ 危险：直接拼接 SQL，存在 SQL 注入风险
  async findByEmail(email) {
    const query = `SELECT * FROM users WHERE email = '${email}'`;
    return await this.db.execute(query);
  }

  // ❌ 危险：用户输入直接拼接到 SQL
  async searchUsers(searchTerm) {
    const query = `
      SELECT * FROM users
      WHERE name LIKE '%${searchTerm}%'
    `;
    return await this.db.execute(query);
  }

  // ❌ 危险：多个参数都直接拼接
  async findByNameAndRole(name, role) {
    const query = `
      SELECT * FROM users
      WHERE name = '${name}' AND role = '${role}'
    `;
    return await this.db.execute(query);
  }
}
```

## Code Review 发现

### 🔴 CRITICAL 问题

**问题 1: findByEmail 存在 SQL 注入**

```markdown
文件: src/user/user-repository.js:8
级别: CRITICAL
类型: 安全漏洞

问题:
直接拼接用户输入到 SQL 查询中，存在严重的 SQL 注入风险。攻击者可以通过构造特殊的 email 值执行任意 SQL 命令。

攻击示例:
email = "admin' OR '1'='1' --"
生成的 SQL: SELECT * FROM users WHERE email = 'admin' OR '1'='1' --'
结果: 返回所有用户数据

建议:
使用参数化查询或 ORM 库。大多数数据库驱动和 ORM 库都提供安全的参数化方式。

修复方案:
async findByEmail(email) {
  // ✅ 使用参数化查询
  const query = 'SELECT * FROM users WHERE email = ?';
  return await this.db.execute(query, [email]);
}
```

**问题 2: searchUsers 存在 SQL 注入**

```markdown
文件: src/user/user-repository.js:15
级别: CRITICAL
类型: 安全漏洞

问题:
LIKE 查询直接拼接用户输入，同样存在 SQL 注入风险。虽然使用 % 通配符，但仍然不安全。

攻击示例:
searchTerm = "%' OR '1'='1' --"
生成的 SQL: SELECT * FROM users WHERE name LIKE '%%' OR '1'='1' --%'
结果: 返回所有用户

建议:
使用参数化查询，在参数中添加通配符。

修复方案:
async searchUsers(searchTerm) {
  // ✅ 在参数中包含通配符
  const query = 'SELECT * FROM users WHERE name LIKE ?';
  return await this.db.execute(query, [`%${searchTerm}%`]);
}
```

**问题 3: findByNameAndRole 存在 SQL 注入**

```markdown
文件: src/user/user-repository.js:24
级别: CRITICAL
类型: 安全漏洞

问题:
两个参数都直接拼接，攻击面更大。可以同时注入两个参数。

建议:
使用命名参数或位置参数。

修复方案:
async findByNameAndRole(name, role) {
  // ✅ 使用多个参数
  const query = `
    SELECT * FROM users
    WHERE name = ? AND role = ?
  `;
  return await this.db.execute(query, [name, role]);
}
```

## 修复后的代码

```javascript
// file: src/user/user-repository.js (已修复)

class UserRepository {
  constructor(db) {
    this.db = db;
  }

  // ✅ 使用参数化查询
  async findByEmail(email) {
    const query = 'SELECT * FROM users WHERE email = ?';
    return await this.db.execute(query, [email]);
  }

  // ✅ 在参数中包含通配符
  async searchUsers(searchTerm) {
    const query = 'SELECT * FROM users WHERE name LIKE ?';
    return await this.db.execute(query, [`%${searchTerm}%`]);
  }

  // ✅ 使用多个参数
  async findByNameAndRole(name, role) {
    const query = `
      SELECT * FROM users
      WHERE name = ? AND role = ?
    `;
    return await this.db.execute(query, [name, role]);
  }

  // ✅ 使用 ORM 库（示例：使用 Prisma）
  async findById(id) {
    return await this.db.user.findUnique({
      where: { id }
    });
  }

  // ✅ 使用查询构建器（示例：使用 Knex）
  async findActiveUsers() {
    return await this.db('users')
      .where({ is_active: true })
      .select('*');
  }
}
```

## 测试用例

```javascript
// file: test/user/user-repository.test.js

describe('UserRepository', () => {
  let repository;
  let mockDb;

  beforeEach(() => {
    mockDb = {
      execute: jest.fn()
    };
    repository = new UserRepository(mockDb);
  });

  describe('findByEmail', () => {
    it('应该使用参数化查询', async () => {
      const email = 'user@example.com';
      await repository.findByEmail(email);

      expect(mockDb.execute).toHaveBeenCalledWith(
        'SELECT * FROM users WHERE email = ?',
        [email]
      );
    });

    it('应该正确处理包含特殊字符的 email', async () => {
      const email = "admin' OR '1'='1";
      await repository.findByEmail(email);

      // 验证 SQL 不被污染
      const [query] = mockDb.execute.mock.calls[0];
      expect(query).not.toContain('OR');
      expect(query).toContain('?');
    });
  });

  describe('searchUsers', () => {
    it('应该安全处理搜索词', async () => {
      const searchTerm = "test' OR '1'='1";
      await repository.searchUsers(searchTerm);

      const [query] = mockDb.execute.mock.calls[0];
      expect(query).not.toContain('OR');
      expect(query).toContain('?');

      const [params] = mockDb.execute.mock.calls[0];
      expect(params[0]).toBe('%test\' OR \'1\'=\'1%');
    });
  });
});
```

## 学习要点

### 1. 识别 SQL 注入

**危险信号：**
- 字符串模板拼接 SQL（`'...${userInput}...'`）
- 字符串拼接 SQL（`'...' + userInput + '...'`）
- 用户输入直接插入 SQL

### 2. 防御 SQL 注入

**安全方法：**
- ✅ 参数化查询（推荐）
- ✅ 使用 ORM 库
- ✅ 使用查询构建器
- ✅ 输入验证和清理

**永远不要：**
- ❌ 直接拼接 SQL
- ❌ 使用字符串格式化
- ❌ 相信任何用户输入

### 3. Code Review 检查清单

- [ ] 所有数据库查询都使用参数化
- [ ] 没有字符串拼接 SQL
- [ ] 输入验证在查询之前
- [ ] 错误处理不泄露敏感信息
- [ ] 测试覆盖安全场景

### 4. 自动化检测

**工具推荐：**
- **ESLint:** eslint-plugin-security
- **Snyk:** 依赖安全扫描
- **SonarQube:** 代码质量分析
- **SQLMap:** 自动化 SQL 注入测试

### 5. 紧急修复流程

如果发现 SQL 注入漏洞：

1. **立即评估影响范围**
   ```bash
   git grep "SELECT" -- *.js
   git grep "INSERT" -- *.js
   git grep "UPDATE" -- *.js
   git grep "DELETE" -- *.js
   ```

2. **优先修复用户输入处理**
   - 认证相关
   - 权限检查
   - 数据修改

3. **添加测试用例**
   - 正常输入
   - 特殊字符
   - SQL 注入尝试

4. **安全发布**
   - 经过充分测试
   - 通知安全团队
   - 监控异常请求

## 进阶建议

### 1. 使用类型安全查询

```javascript
// ✅ 使用 TypeScript + ORM
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async findUserByEmail(email: string) {
  return await prisma.user.findUnique({
    where: { email }
  });
}
```

### 2. 添加审计日志

```javascript
async findByEmail(email) {
  // ✅ 记录查询日志
  await this.logQuery('find_user_by_email', { email });

  const query = 'SELECT * FROM users WHERE email = ?';
  return await this.db.execute(query, [email]);
}
```

### 3. 使用安全配置

```javascript
// 数据库连接配置
const dbConfig = {
  host: process.env.DB_HOST,
  // ✅ 禁用多语句执行
  multipleStatements: false,
  // ✅ 启用 SSL
  ssl: process.env.NODE_ENV === 'production',
  // ✅ 设置连接池限制
  connectionLimit: 10
};
```

---

**关键教训：** 永远不要信任用户输入。使用参数化查询是防止 SQL 注入的最有效方法。在 Code Review 中，任何数据库查询都应该使用安全的方式处理用户输入。
