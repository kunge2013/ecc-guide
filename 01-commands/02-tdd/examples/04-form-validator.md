# 场景 4: 数据验证 - 表单验证器

这个场景展示了 TDD 如何实现复杂的数据验证逻辑。

。

## 需求

实现用户注册表单验证器：
- 用户名验证（长度、字符、非法词）
- 密码验证（强度、常见密码检测）
- 邮箱验证（格式、域名检查）
- 手机号验证（格式、运营商）
- 返回详细的错误信息

## 测试驱动开发过程

### Step 1: 写测试（RED）- validator_test.ts

```typescript
import { UserValidator, ValidationResult } from './validator';

describe('UserValidator', () => {
  describe('用户名验证', () => {
    it('应该接受有效的用户名', () => {
      const result = UserValidator.validateUsername('john_doe123');
      expect(result.valid).toBe(true);
      expect(result.errors).toEqual([]);
    });

    it('应该拒绝过短的用户名', () => {
      const result = UserValidator.validateUsername('ab');
      expect(result.valid).toBe(false);
      expect(result.errors).toContain('用户名至少3个字符');
    });

    it('应该拒绝过长的用户名', () => {
      const result = UserValidator.validateUsername('a'.repeat(31));
      expect(result.valid).toBe(false);
      expect(result.errors).toContain('用户名最多30个字符');
    });

    it('应该拒绝非法字符', () => {
      const result = UserValidator.validateUsername('john@doe');
      expect(result.valid).toBe(false);
      expect(result.errors).toContain('用户名只能包含字母、数字和下划线');
    });

    it('应该拒绝保留词', () => {
      const result = UserValidator.validateUsername('admin');
      expect(result.valid).toBe(false);
      expect(result.errors).toContain('该用户名已被保留');
    });

    it('应该拒绝空字符串', () => {
      const result = UserValidator.validateUsername('');
      expect(result.valid).toBe(false);
      expect(result.errors.length).toBeGreaterThan(0);
    });
  });

  describe('密码验证', () => {
    it('应该接受强密码', () => {
      const result = UserValidator.validatePassword('SecurePass123!');
      expect(result.valid).toBe(true);
      expect(result.errors).toEqual([]);
    });

    it('应该检测弱密码：长度不足', () => {
      const result = UserValidator.validatePassword('Short1!');
      expect(result.valid).toBe(false);
      expect(result.errors).toContain('密码至少8个字符');
    });

    it('应该检测弱密码：缺少大写字母', () => {
      const result = UserValidator.validatePassword('securepass123!');
      expect(result.valid).toBe(false);
      expect(result.errors).toContain('密码必须包含大写字母');
    });

    it('应该检测弱密码：缺少小写字母', () => {
      const result = UserValidator.validatePassword('SECUREPASS123!');
      expect(result.valid).toBe(false);
      expect(result.errors).toContain('密码必须包含小写字母');
    });

    it('应该检测弱密码：缺少数字', () => {
      const result = UserValidator.validatePassword('SecurePass!');
      expect(result.valid).toBe(false);
      expect(result.errors).toContain('密码必须包含数字');
    });

    it('应该检测弱密码：缺少特殊字符', () => {
      const result = UserValidator.validatePassword('SecurePass123');
      expect(result.valid).toBe(false);
      expect(result.errors).toContain('密码必须包含特殊字符');
    });

    it('应该拒绝常见密码', () => {
      const result = UserValidator.validatePassword('Password123!');
      expect(result.valid).toBe(false);
      expect(result.errors).toContain('该密码过于常见，请选择更强的密码');
    });

    it('应该检测密码中的用户名', () => {
      const result = UserValidator.validatePassword('John123!', 'john');
      expect(result.valid).toBe(false);
      expect(result.errors).toContain('密码不能包含用户名');
    });
  });

  describe('邮箱验证', () => {
    it('应该接受有效的邮箱', () => {
      const result = UserValidator.validateEmail('user@example.com');
      expect(result.valid).toBe(true);
    });

    it('应该接受带有子域的邮件', () => {
      const result = UserValidator.validateEmail('user@mail.example.com');
      expect(result.valid).toBe(true);
    });

    it('应该拒绝无效的邮箱格式', () => {
      const result = UserValidator.validateEmail('invalid-email');
      expect(result.valid).toBe(false);
    });

    it('应该拒绝缺少域名的邮箱', () => {
      const result = UserValidator.validateEmail('user@');
      expect(result.valid).toBe(false);
    });

    it('应该拒绝一次性邮箱', () => {
      const result = UserValidator.validateEmail('user@tempmail.com');
      expect(result.valid).toBe(false);
      expect(result.errors).toContain('不支持一次性邮箱');
    });
  });

  describe('手机号验证', () => {
    it('应该接受有效的中国大陆手机号', () => {
      const result = UserValidator.validatePhoneNumber('13812345678', 'CN');
      expect(result.valid).toBe(true);
    });

    it('应该拒绝无效的手机号', () => {
      const result = UserValidator.validatePhoneNumber('12345678901', 'CN');
      expect(result.valid).toBe(false);
    });

    it('应该拒绝过短的手机号', () => {
      const result = UserValidator.validatePhoneNumber('1381234', 'CN');
      expect(result.valid).toBe(false);
    });

    it('应该接受美国的手机号', () => {
      const result = UserValidator.validatePhoneNumber('+14155552671', 'US');
      expect(result.valid).toBe(true);
    });
  });

  describe('完整表单验证', () => {
    it('应该通过完整的有效表单', () => {
      const data = {
        username: 'john_doe123',
        email: 'john@example.com',
        password: 'SecurePass123!',
        phone: '13812345678',
        country: 'CN'
      };

      const result = UserValidator.validateRegistration(data);
      expect(result.valid).toBe(true);
      expect(result.errors).toEqual({});
    });

    it('应该返回所有字段错误', () => {
      const data = {
        username: 'ab',
        email: 'invalid',
        password: 'weak',
        phone: 'invalid',
        country: 'CN'
      };

      const result = UserValidator.validateRegistration(data);
      expect(result.valid).toBe(false);
      expect(Object.keys(result.errors)).toContain('username');
      expect(Object.keys(result.errors)).toContain('email');
      expect(Object.keys(result.errors)).toContain('password');
      expect(Object.keys(result.errors)).toContain('phone');
    });

    it('应该通过部分验证（仅必填字段）', () => {
      const data = {
        username: 'john_doe123',
        email: 'john@example.com',
        password: 'SecurePass123!'
      };

      const result = UserValidator.validateRegistration(data);
      expect(result.valid).toBe(true);
    });
  });
});
```

### Step 2: 实现代码（GREEN）- validator.ts

```typescript
interface ValidationResult {
  valid: boolean;
  errors: string[];
}

interface RegistrationData {
  username: string;
  email: string;
  password: string;
  phone?: string;
  country?: string;
}

interface FormValidationResult {
  valid: boolean;
  errors: Record<string, string[]>;
}

const RESERVED_USERNAMES = ['admin', 'root', 'system', 'api', 'www'];
const COMMON_PASSWORDS = ['password', '123456', 'qwerty', 'admin', 'welcome'];
const DISPOSABLE_EMAIL_DOMAINS = ['tempmail.com', 'guerrillamail.com', 'mailinator.com'];
const EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

class UserValidator {
  private static readonly USERNAME_MIN_LENGTH = 3;
  private static readonly USERNAME_MAX_LENGTH = 30;
  private static readonly USERNAME_REGEX = /^[a-zA-Z0-9_]+$/;
  private static readonly PASSWORD_MIN_LENGTH = 8;

  static validateUsername(username: string): ValidationResult {
    const errors: string[] = [];

    if (!username || username.trim().length === 0) {
      errors.push('用户名不能为空');
    } else {
      if (username.length < this.USERNAME_MIN_LENGTH) {
        errors.push(`用户名至少${this.USERNAME_MIN_LENGTH}个字符`);
      }

      if (username.length > this.USERNAME_MAX_LENGTH) {
        errors.push(`用户名最多${this.USERNAME_MAX_LENGTH}个字符`);
      }

      if (!this.USERNAME_REGEX.test(username)) {
        errors.push('用户名只能包含字母、数字和下划线');
      }

      if (RESERVED_USERNAMES.includes(username.toLowerCase())) {
        errors.push('该用户名已被保留');
      }
    }

    return {
      valid: errors.length === 0,
      errors
    };
  }

  static validatePassword(password: string, username?: string): ValidationResult {
    const errors: string[] = [];

    if (!password || password.length === 0) {
      errors.push('密码不能为空');
    } else {
      if (password.length < this.PASSWORD_MIN_LENGTH) {
        errors.push(`密码至少${this.PASSWORD_MIN_LENGTH}个字符`);
      }

      if (!/[A-Z]/.test(password)) {
        errors.push('密码必须包含大写字母');
      }

      if (!/[a-z]/.test(password)) {
        errors.push('密码必须包含小写字母');
      }

      if (!/[0-9]/.test(password)) {
        errors.push('密码必须包含数字');
      }

      if (!/[!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?]/.test(password)) {
        errors.push('密码必须包含特殊字符');
      }

      if (COMMON_PASSWORDS.includes(password.toLowerCase())) {
        errors.push('该密码过于常见，请选择更强的密码');
      }

      if (username && password.toLowerCase().includes(username.toLowerCase())) {
        errors.push('密码不能包含用户名');
      }
    }

    return {
      valid: errors.length === 0,
      errors
    };
  }

  static validateEmail(email: string): ValidationResult {
    const errors: string[] = [];

    if (!email || email.trim().length === 0) {
      errors.push('邮箱不能为空');
    } else {
      if (!EMAIL_REGEX.test(email)) {
        errors.push('邮箱格式无效');
      } else {
        const domain = email.split('@')[1];
        if (DISPOSABLE_EMAIL_DOMAINS.includes(domain)) {
          errors.push('不支持一次性邮箱');
        }
      }
    }

    return {
      valid: errors.length === 0,
      errors
    };
  }

  static validatePhoneNumber(phone: string, country: string = 'CN'): ValidationResult {
    const errors: string[] = [];

    if (!phone || phone.trim().length === 0) {
      return { valid: true, errors: [] }; // 可选字段
    }

    switch (country) {
      case 'CN':
        // 中国大陆手机号：1开头，第二位3-9，共11位
        if (!/^1[3-9]\d{9}$/.test(phone)) {
          errors.push('手机号格式无效');
        }
        break;
      case 'US':
        // 美国手机号：+1开头，共11位数字
        if (!/^\+1\d{10}$/.test(phone)) {
          errors.push('手机号格式无效');
        }
        break;
      default:
        errors.push('不支持该国家的手机号格式');
    }

    return {
      valid: errors.length === 0,
      errors
    };
  }

  static validateRegistration(data: RegistrationData): FormValidationResult {
    const errors: Record<string, string[]> = {};

    const usernameResult = this.validateUsername(data.username);
    if (!usernameResult.valid) {
      errors.username = usernameResult.errors;
    }

    const emailResult = this.validateEmail(data.email);
    if (!emailResult.valid) {
      errors.email = emailResult.errors;
    }

    const passwordResult = this.validatePassword(data.password, data.username);
    if (!passwordResult.valid) {
      errors.password = passwordResult.errors;
    }

    if (data.phone && data.country) {
      const phoneResult = this.validatePhoneNumber(data.phone, data.country);
      if (!phoneResult.valid) {
        errors.phone = phoneResult.errors;
      }
    }

    return {
      valid: Object.keys(errors).length === 0,
      errors
    };
  }
}

export { UserValidator, ValidationResult, RegistrationData, FormValidationResult };
```

### Step 3: 重构（REFACTOR）

提取验证规则配置：

```typescript
interface ValidationRule {
  minLength?: number;
  maxLength?: number;
  pattern?: RegExp;
  forbiddenValues?: string[];
  allowedValues?: string[];
}

class ConfigurableValidator {
  private rules: Map<string, ValidationRuleRule>;

  constructor(rules: Record<string, ValidationRule>) {
    this.rules = new Map(Object.entries(rules));
  }

  validate(fieldName: string, value: string): ValidationResult {
    const rule = this.rules.get(fieldName);
    if (!rule) {
      return { valid: true, errors: [] };
    }

    const errors: string[] = [];

    if (rule.minLength && value.length < rule.minLength) {
      errors.push(`${fieldName}至少${rule.minLength}个字符`);
    }

    if (rule.maxLength && value.length > rule.maxLength) {
      errors.push(`${fieldName}最多${rule.maxLength}个字符`);
    }

    if (rule.pattern && !rule.pattern.test(value)) {
      errors.push(`${fieldName}格式无效`);
    }

    if (rule.forbiddenValues && rule.forbiddenValues.includes(value)) {
      errors.push(`${fieldName}值无效`);
    }

    if (rule.allowedValues && !rule.allowedValues.includes(value)) {
      errors.push(`${fieldName}值不在允许范围内`);
    }

    return { valid: errors.length === 0, errors };
  }
}

// 使用配置化的验证器
const validationRules = {
  username: {
    minLength: 3,
    maxLength: 30,
    pattern: /^[a-zA-Z0-9_]+$/,
    forbiddenValues: RESERVED_USERNAMES
  },
  // ... 其他规则
};

const configurableValidator = new ConfigurableValidator(validationRules);
```

## 关键学习点

1. **错误信息本地化** - 清晰的错误消息便于前端显示
2. **规则配置化** - 验证规则可配置，便于维护和扩展
3. **组合验证** - 单个字段验证 + 整体表单验证
4. **国际化支持** - 错误消息支持多语言（通过配置）
