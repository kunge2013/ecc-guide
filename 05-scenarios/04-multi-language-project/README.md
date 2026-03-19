# 多语言项目

这个场景展示了如何使用 Claude Code 管理和开发包含多种编程语言的项目。

## 🎯 场景概述

### 适用情况

- 前端（JavaScript/TypeScript）+ 后端（Python/Go/Java）
- 微服务架构，不同服务使用不同语言
- 需要跨语言的代码审查
- 共享库和工具开发

### 核心挑战

- 语言特定的最佳实践
- 跨语言集成测试
- 统一的 CI/CD 流程
- 语言特定的依赖管理

## 📋 使用场景

### 场景 1: 全栈应用开发

**语言组合：** React + Python Flask
**挑战：** 类型定义共享、API 契约同步

**Agent 使用：**
- 各自语言的 TDD Guide Agent
- 语言特定的 Code Reviewer Agent
- E2E 测试验证集成

---

### 场景 2: 微服务架构

**语言组合：** Go + Python + Node.js
**挑战：** 服务间通信、协议定义、错误处理一致

**Agent 使用：**
- Planner Agent 设计服务架构
- 各服务的独立开发
- 集成测试验证

---

### 场景 3: 跨语言共享库

**语言组合：** Rust + Python 绑定
**挑战：** FFI 接口、内存安全、类型转换

**Agent 使用：**
- Rust Reviewer Agent 验证内存安全
- Python Reviewer Agent 验证绑定
- E2E 测试验证功能

## 🚀 管理策略

### 1. 语言特定配置

```yaml
# .github/workflows/ci.yml
jobs:
  typescript:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm test
      - run: npx tsc --noEmit

  python:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-python@v3
      - run: pip install -r requirements.txt
      - run: pytest
      - run: mypy src/

  go:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-go@v3
      - run: go mod tidy
      - run: go test ./...
      - run: go vet ./...
```

### 2. 统一测试标准

所有语言遵循：
- 测试覆盖率 > 80%
- 代码通过 linting
- 类型检查通过
- 集成测试通过

### 3. 共享类型定义

使用 OpenAPI/Swagger 或 GraphQL Schema 作为接口契约：
- 自动生成类型定义
- 跨语言验证
- 版本控制

## 📊 语言特性对比

| 特性 | TypeScript | Python | Go |
|------|-----------|--------|-----|
| 静态类型 | ✅ | 可选 | ✅ |
| 性能 | 中等 | 中等 | 高 |
| 并发 | Promise | asyncio | goroutine |
| 部署 | Node.js | WSGI/ASGI | Binary |

## 💡 最佳实践

### 1. 语言边界清晰

- 定义明确的 API 契约
- 使用版本控制
- 文档化所有接口

### 2. 统一错误处理

- 定义错误码规范
- 跨语言错误传播
- 统一错误日志格式

### 3. 共享测试数据

- 使用 JSON/YAML 定义测试数据
- 多语言使用相同测试用例
- 集成测试覆盖所有语言边界

---

**提示：** 多语言项目需要明确的语言边界和统一的接口契约。使用适当的 Agent 处理每种语言的特定问题。
