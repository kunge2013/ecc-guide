# 场景 2: 异步错误处理改进

这个场景展示如何通过 Code Review 识别和改进异步代码中的错误处理。

## 原始代码（错误处理不完善）

```javascript
// file: src/api/user-service.js

class UserService {
  constructor(userRepository, emailService) {
    this.userRepository = userRepository;
    this.emailService = emailService;
  }

  // ❌ 问题：没有处理 Promise 拒绝
  async createUser(userData) {
    const user = await this.userRepository.create(userData);
    await this.emailService.sendWelcomeEmail(user.email);
    return user;
  }

  // ❌ 问题：catch 中只有 console.log
  async deleteUser(userId) {
    try {
      await this.userRepository.delete(userId);
      return { success: true };
    } catch (error) {
      console.log('删除用户失败', error); // 丢失错误
      return { success: false };
    }
  }

  // ❌ 问题：错误处理过于宽泛
  async updateUser(userId, updates) {
    try {
      const user = await this.userRepository.findById(userId);
      if (!user) {
        throw new Error('用户不存在');
      }

      const updatedUser = await this.userRepository.update(userId, updates);
      return updatedUser;
    } catch (error) {
      // 捕获所有错误，包括系统错误
      throw new Error('更新用户失败');
    }
  }

  // ❌ 问题：多个异步操作没有错误边界
  async batchCreateUsers(usersData) {
    const results = [];
    for (const userData of usersData) {
      const user = await this.userRepository.create(userData);
      results.push(user);
    }
    return results;
  }

  // ❌ 问题：没有清理资源
  async processLargeFile(filePath) {
    const reader = new FileReader(filePath);
    const data = await reader.read();
    const processed = await this.processData(data);
    return processed;
  }
}
```

## Code Review 发现

### 🔴 CRITICAL 问题

**问题 1: createUser 没有错误处理**

```markdown
文件: src/api/user-service.js:9
级别: CRITICAL
类型: 错误处理

问题:
createUser 方法完全没有错误处理。如果数据库操作或邮件发送失败，
错误会冒泡到调用者，可能导致未捕获的 Promise rejection。

影响:
- 数据库和邮件服务不一致（用户创建但邮件未发送）
- 错误信息不清晰
- 难以追踪问题根因

建议:
添加 try-catch 块，确保错误被正确处理和记录。

修复方案:
async createUser(userData) {
  try {
    const user = await this.userRepository.create(userData);
    try {
      await this.emailService.sendWelcomeEmail(user.email);
    } catch (emailError) {
      // 邮件发送失败不应阻止用户创建
      this.logger.error('发送欢迎邮件失败', {
        userId: user.id,
        error: emailError.message
      });
      // 可以考虑重试或队列处理
    }
    return user;
  } catch (error) {
    this.logger.error('创建用户失败', {
      error: error.message,
      userData: { ...userData, password: '[REDACTED]' }
    });
    throw new CreateUserError('无法创建用户', { cause: error });
  }
}
```

**问题 2: deleteUser 静默吞掉错误**

```markdown
文件: src/api/user-service.js:18
级别: HIGH
类型: 错误处理

问题:
deleteUser 捕获错误后只记录到 console，然后返回 success: false。
调用者无法知道失败的具体原因，无法进行适当的错误处理。

影响:
- 无法区分不同类型的失败（权限不足、数据库错误等）
- 错误信息丢失
- 难以调试和监控

建议:
重新抛出错误，或返回详细的错误信息。

修复方案:
async deleteUser(userId) {
  try {
    await this.userRepository.delete(userId);
    return { success: true };
  } catch (error) {
    this.logger.error('删除用户失败', {
      userId,
      error: error.message
    });
    // 方案 1: 重新抛出具体错误
    throw new DeleteUserError('删除用户失败', { cause: error });

    // 方案 2: 返回详细错误信息
    // return {
    //   success: false,
    //   error: {
    //     message: error.message,
    //     code: error.code,
    //     userId
    //   }
    // };
  }
}
```

**问题 3: updateUser 丢失原始错误信息**

```markdown
文件: src/api/user-service.js:30
级别: HIGH
类型: 错误处理

问题:
catch 块捕获所有错误并抛出一个通用的错误消息，丢失了原始错误信息。
这使得调试非常困难，不知道是数据库连接问题、权限问题还是其他问题。

影响:
- 调试困难
- 错误监控无法正确分类
- 用户看不到有意义的错误信息

建议:
保留原始错误信息，或创建特定的错误类型。

修复方案:
async updateUser(userId, updates) {
  try {
    const user = await this.userRepository.findById(userId);
    if (!user) {
      throw new UserNotFoundError(`用户 ${userId} 不存在`);
    }

    return await this.userRepository.update(userId, updates);
  } catch (error) {
    // 如果是已知的业务错误，直接传递
    if (error instanceof UserNotFoundError) {
      throw error;
    }

    // 否则包装为系统错误
    this.logger.error('更新用户失败', {
      userId,
      error: error.message,
      stack: error.stack
    });

    throw new UpdateUserError('更新用户失败', { cause: error });
  }
}
```

### 🟡 MEDIUM 问题

**问题 4: batchCreateUsers 没有错误隔离**

```markdown
文件: src/api/user-service.js:48
级别: MEDIUM
类型: 错误处理

问题:
批量创建用户时，如果中途失败，已经创建的用户不会被回滚。
这可能导致部分成功的状态，不一致的数据。

影响:
- 数据不一致
- 难以重试
- 用户体验差

建议:
(1) 使用数据库事务
(2) 记录失败的项目
(3) 提供重试机制

修复方案:
async batchCreateUsers(usersData) {
  const results = [];
  const failures = [];

  for (const userData of usersData) {
    try {
      const user = await this.userRepository.create(userData);
      results.push({ success: true, user });
    } catch (error) {
      failures.push({
        success: false,
        userData: { ...userData, password: '[REDACTED]' },
        error: error.message
      });
      this.logger.error('批量创建用户失败', {
        error: error.message
      });
    }
  }

  return {
    total: usersData.length,
    success: results.length,
    failures: failures.length,
    results,
    failures
  };
}

// 或者使用事务和回滚
async batchCreateUsersWithTransaction(usersData) {
  const transaction = await this.userRepository.beginTransaction();

  try {
    const users = [];
    for (const userData of usersData) {
      const user = await this.userRepository.create(userData, transaction);
      users.push(user);
    }
    await transaction.commit();
    return users;
  } catch (error) {
    await transaction.rollback();
    throw error;
  }
}
```

**问题 5: processLargeFile 资源泄漏**

```markdown
文件: src/api/user-service.js:57
级别: MEDIUM
类型: 资源管理

问题:
FileReader 没有被正确关闭，可能导致文件句柄泄漏。
如果读取过程中出错，文件可能保持打开状态。

影响:
- 资源泄漏
- 性能下降
- 系统稳定性问题

建议:
使用 finally 块确保资源释放，或使用 try-with-resources 模式。

修复方案:
async processLargeFile(filePath) {
  const reader = new FileReader(filePath);

  try {
    const data = await reader.read();
    return await this.processData(data);
  } catch (error) {
    this.logger.error('处理文件失败', {
      filePath,
      error: error.message
    });
    throw error;
  } finally {
    // ✅ 确保文件被关闭
    reader.close();
  }
}
```

## 修复后的代码

```javascript
// file: src/api/user-service.js (已修复)

// 自定义错误类
class CreateUserError extends Error {
  constructor(message, { cause } = {}) {
    super(message, { cause });
    this.name = 'CreateUserError';
  }
}

class DeleteUserError extends Error {
  constructor(message, { cause } = {}) {
    super(message, { cause });
    this.name = 'DeleteUserError';
  }
}

class UserNotFoundError extends Error {
  constructor(message) {
    super(message);
    this.name = 'UserNotFoundError';
    this.statusCode = 404;
  }
}

class UpdateUserError extends Error {
  constructor(message, { cause } = {}) {
    super(message, { cause });
    this.name = 'UpdateUserError';
  }
}

class UserService {
  constructor(userRepository, emailService, logger) {
    this.userRepository = userRepository;
    this.emailService = emailService;
    this.logger = logger || console;
  }

  async createUser(userData) {
    try {
      const user = await this.userRepository.create(userData);

      // 邮件发送失败不应阻止用户创建
      try {
        await this.emailService.sendWelcomeEmail(user.email);
      } catch (emailError) {
        this.logger.error('发送欢迎邮件失败', {
          userId: user.id,
          error: emailError.message,
          stack: emailError.stack
        });

        // 可以添加到重试队列
        await this.emailService.queueWelcomeEmail(user.id);
      }

      return user;
    } catch (error) {
      this.logger.error('创建用户失败', {
        error: error.message,
        userData: { ...userData, password: '[REDACTED]' },
        stack: error.stack
      });
      throw new CreateUserError('无法创建用户', { cause: error });
    }
  }

  async deleteUser(userId) {
    try {
      await this.userRepository.delete(userId);
      return { success: true };
    } catch (error) {
      this.logger.error('删除用户失败', {
        userId,
        error: error.message,
        stack: error.stack
      });
      throw new DeleteUserError('删除用户失败', { cause: error });
    }
  }

  async updateUser(userId, updates) {
    try {
      const user = await this.userRepository.findById(userId);
      if (!user) {
        throw new UserNotFoundError(`用户 ${userId} 不存在`);
      }

      return await this.userRepository.update(userId, updates);
    } catch (error) {
      if (error instanceof UserNotFoundError) {
        throw error;
      }

      this.logger.error('更新用户用户失败', {
        userId,
        error: error.message,
        stack: error.stack
      });

      throw new UpdateUserError('更新用户失败', { cause: error });
    }
  }

  async batchCreateUsers(usersData) {
    const transaction = await this.userRepository.beginTransaction();

    try {
      const users = [];
      for (const userData of usersData) {
        const user = await this.userRepository.create(userData, transaction);
        users.push(user);
      }
      await transaction.commit();
      return users;
    } catch (error) {
      await transaction.rollback();
      this.logger.error('批量创建用户失败', {
        count: usersData.length,
        error: error.message
      });
      throw error;
    }
  }

  async processLargeFile(filePath) {
    const reader = new FileReader(filePath);

    try {
      const data = await reader.read();
      return await this.processData(data);
    } catch (error) {
      this.logger.error('处理文件失败', {
        filePath,
        error: error.message
      });
      throw error;
    } finally {
      reader.close();
    }
  }
}
```

## 测试用例

```javascript
// file: test/api/user-service.test.js

describe('UserService', () => {
  let service;
  let mockUserRepo;
  let mockEmailService;
  let mockLogger;

  beforeEach(() => {
    mockUserRepo = {
      create: jest.fn(),
      findById: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
      beginTransaction: jest.fn()
    };
    mockEmailService = {
      sendWelcomeEmail: jest.fn(),
      queueWelcomeEmail: jest.fn()
    };
    mockLogger = {
      error: jest.fn(),
      info: jest.fn()
    };

    service = new UserService(mockUserRepo, mockEmailService, mockLogger);
  });

  describe('createUser', () => {
    it('应该成功创建用户并发送欢迎邮件', async () => {
      const userData = { name: 'John', email: 'john@example.com' };
      const createdUser = { id: 1, ...userData };

      mockUserRepo.create.mockResolvedValue(createdUser);
      mockEmailService.sendWelcomeEmail.mockResolvedValue();

      const result = await service.createUser(userData);

      expect(result).toEqual(createdUser);
      expect(mockEmailService.sendWelcomeEmail).toHaveBeenCalledWith(userData.email);
    });

    it('应该处理邮件发送失败但用户创建成功', async () => {
      const userData = { name: 'John', email: 'john@example.com' };
      const createdUser = { id: 1, ...userData };

      mockUserRepo.create.mockResolvedValue(createdUser);
      mockEmailService.sendWelcomeEmail.mockRejectedValue(new Error('SMTP error'));
      mockEmailService.queueWelcomeEmail.mockResolvedValue();

      const result = await service.createUser(userData);

      expect(result).toEqual(createdUser);
      expect(mockEmailService.queue.queueWelcomeEmail).toHaveBeenCalledWith(1);
    });

    it('应该抛出 CreateUserError 当创建失败', async () => {
      const userData = { name: 'John', email: 'john@example.com' };
      mockUserRepo.create.mockRejectedValue(new Error('DB error'));

      await expect(service.createUser(userData))
        .rejects.toThrow(CreateUserError);

      expect(mockLogger.error).toHaveBeenCalled();
    });
  });

  describe('deleteUser', () => {
    it('应该成功删除用户', async () => {
      mockUserRepo.delete.mockResolvedValue();

      const result = await service.deleteUser(1);

      expect(result.success).toBe(true);
    });

    it('应该抛出 DeleteUserError 当删除失败', async () => {
      mockUserRepo.delete.mockRejectedValue(new Error('Not found'));

      await expect(service.deleteUser(1))
        .rejects.toThrow(DeleteUserError);

      expect(mockLogger.error).toHaveBeenCalled();
    });
  });

  describe('updateUser', () => {
    it('应该成功更新用户', async () => {
      const existingUser = { id: 1, name: 'John', email: 'john@example.com' };
      const updatedUser = { ...existingUser, name: 'Jane' };

      mockUserRepo.findById.mockResolvedValue(existingUser);
      mockUserRepo.update.mockResolvedValue(updatedUser);

      const result = await service.updateUser(1, { name: 'Jane' });

      expect(result).toEqual(updatedUser);
    });

    it('应该抛出 UserNotFoundError 当用户不存在', async () => {
      mockUserRepo.findById.mockResolvedValue(null);

      await expect(service.updateUser(1, { name: 'Jane' }))
        .rejects.toThrow(UserNotFoundError);
    });

    it('应该抛出 UpdateUserError 当更新失败', { async () => {
      const existingUser = { id: 1, name: 'John' };
      mockUserRepo.findById.mockResolvedValue(existingUser);
      mockUserRepo.update.mockRejectedValue(new Error('DB error'));

      await expect(service.updateUser(1, { name: 'Jane' }))
        .rejects.toThrow(UpdateUserError);
    }});
  });
});
```

## 学习要点

### 1. 异步错误处理模式

**最佳实践：**
- ✅ 每个异步操作都有 try-catch
- ✅ 使用自定义错误类
- ✅ 保留原始错误信息（cause）
- ✅ 记录完整的错误上下文
- ✅ 区分业务错误和系统错误

### 2. 错误传播策略

| 场景 | 策略 | 说明 |
|------|------|------|
| 用户输入错误 | 抵达层 | 验证并返回友好消息 |
| 业务逻辑错误 | 抛出特定错误 | 如 UserNotFoundError |
| 系统错误 | 包装并抛出 | 保留 cause |
| 可恢复错误 | 记录并继续 | 如邮件发送失败 |

### 3. 资源管理

```javascript
// ✅ 使用 finally 确保清理
async withResource(resource) {
  try {
    await resource.open();
    return await useResource(resource);
  } finally {
    await resource.close();
  }
}

// ✅ 使用上下文管理器（如果支持）
async withResourceUsingManager(resource) {
  await using r = await resource.open();
  return await useResource(r);
}
```

### 4. 错误监控和日志

```javascript
// ✅ 结构化错误日志
this.logger.error('操作失败', {
  operation: 'createUser',
  userId: user.id,
  error: {
    name: error.name,
    message: error.message,
    code: error.code,
    stack: error.stack
  },
  metadata: {
    // 相关的上下文信息
  }
});
```

---

**关键教训：** 异步代码的错误处理至关重要。每个 async 函数都应该明确处理错误，或者抛出有意义的错误。使用自定义错误类和结构化日志可以显著提高可调试性。
