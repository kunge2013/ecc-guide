# Language Reviewers Agent 示例集合

这个目录包含了 3 个语言特定的代码审查和构建错误场景。

## 📚 示例列表

### 1. [TypeScript 类型错误](./01-typescript-errors.md)
**语言：** TypeScript
**工具：** tsc
**优先级：** HIGH (阻止编译)

**错误类型：**
- 属性不存在 (TS2339)
- 类型不匹配 (TS2345)
- 隐式 any (TS7006)
- 可能为 null (TS2663)

**关键修复：**
- 使用明确类型接口
- 添加可选属性
- 类型守卫
- 避免 any 类型

---

### 2. [Python 类型检查](./02-python-type-checking.md)
**语言：** Python
**工具：** mypy
**优先级：** HIGH (类型安全)

**错误类型：**
- 缺少返回类型注解
- 参数类型不明确
- 返回类型与注解不匹配
- 复杂类型未定义

**关键修复：**
- 完整类型注解
- 使用 typing 模块
- 处理 Optional 类型
- 使用 dataclasses

---

### 3. [Go 编译错误](./03-go-compilation-errors.md)
**语言：** Go
**工具：** go build / go vet
**优先级：** HIGH (阻止编译)

**错误类型：**
- 类型不匹配
- 未导入包
- 格式化参数错误
- 未使用变量

**关键修复：**
- 正确的错误处理
- 类型断言安全
- 指针和值正确使用
- 处理未使用变量

---

## 🎯 使用建议

### 学习路径

1. **TypeScript 开发者** → 示例 1
   - 类型系统基础
   - 接口和类型定义
   - 泛型使用

2. **Python 开发者** → 示例 2
   - 类型注解
   - mypy 配置
   - 类型别名和泛型

3. **Go 开发者** → 示例 3
   - 静态类型检查
   - 错误处理模式
   - 指针语义

### 错误修复策略

| 语言 | 工具 | 命令 |
|------|------|------|
| TypeScript | tsc | `npx tsc --noEmit` |
| Python | mypy | `mypy src/ --strict` |
| Go | go build | `go build ./...` |

### CI/CD 集成

```yaml
# .github/workflows/type-check.yml

name: Type Check

on: [push, pull_request]

jobs:
  typescript:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm install
      - run: npx tsc --noEmit

  python:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
      - run: pip install mypy
      - run: mypy src/ --strict

  go:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-go@v4
      - run: go build ./...
      - run: go vet ./...
```

---

## 📊 语言特性对比

| 特性 | TypeScript | Python (mypy) | Go |
|------|-----------|---------------|-----|
| 静态类型 | ✅ | 可选 | ✅ |
| 类型推断 | ✅ | ✅ | ✅ |
| 泛型 | ✅ | ✅ | ✅ |
| 联合类型 | ✅ | Union | interface{} |
| 可选类型 | ? \| undefined | Optional | * |
| 类型守卫 | ✅ | ✅ | 类型断言 |

---

## 🔗 相关资源

- [Language Reviewers Agent](../README.md) - Agent 使用指南
- [TypeScript 文档](https://www.typescriptlang.org/docs/) - TS 官方文档
- [mypy 文档](https://mypy.readthedocs.io/) - Python 类型检查
- [Go 语言规范](https://golang.org/ref/spec) - Go 官方规范

---

**提示：** 语言特定的类型检查和编译错误是构建失败的主要原因。及时修复这些错误能保证构建成功。
