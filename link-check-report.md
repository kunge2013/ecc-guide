# 链接检查报告

生成时间: 2026-03-20

## 主 README.md 链接检查结果

### ✅ 存在的链接

以下链接对应的文件/目录存在：

| 链接 | 路径 | 状态 |
|------|------|------|
| [ECC 使用手册](./ecc使用手册.md) | /home/fk/workspace/github/ecc-guide/ecc使用手册.md | ✅ 存在 |
| [学习路线图](./学习路线图.md) | /home/fk/workspace/github/ecc-guide/学习路线图.md | ✅ 存在 |
| [命令总览](./01-commands/README.md) | /home/fk/workspace/github/ecc-guide/01-commands/README.md | ✅ 存在 |
| [01-commands](./01-commands/) | /home/fk/workspace/github/ecc-guide/01-commands/ | ✅ 存在 |
| [02-agents](./02-agents/) | /home/fk/workspace/github/ecc-guide/02-agents/ | ✅ 存在 |
| [03-skills](./03-skills/) | /home/fk/workspace/github/ecc-guide/03-skills/ | ✅ 存在 |
| [03-skills/01-java](./03-skills/01-java/) | /home/fk/workspace/github/ecc-guide/03-skills/01-java/ | ✅ 存在 |
| [04-practice/java-springboot-demo](./04-practice/java-springboot-demo/) | /home/fk/workspace/github/ecc-guide/04-practice/java-springboot-demo/ | ✅ 存在 |
| [05-scenarios](./05-scenarios/) | /home/fk/workspace/github/ecc-guide/05-scenarios/ | ✅ 存在 |
| [05-scenarios/01-new-feature-development](./05-scenarios/01-new-feature-development/) | /home/fk/workspace/github/ecc-guide/05-scenarios/01-new-feature-development/ | ✅ 存在 |
| [06-workflows](./06-workflows/) | /home/fk/workspace/github/ecc-guide/06-workflows/ | ✅ 存在 |
| [07-exercises](./07-exercises/) | /home/fk/workspace/github/ecc-guide/07-exercises/ | ✅ 存在 |

### ❌ 不存在的链接

以下链接对应的文件/目录 **不存在**，需要创建或修复：

| 链接 | 问题描述 | 建议操作 |
|------|----------|----------|
| [01-exercises/beginner](./07-exercises/beginner/) | 07-exercises 下没有 beginner 目录 | 创建 `07-exercises/beginner/` 目录 |
| [初级练习](./07-exercises/beginner/) | 07-exercises 下没有 beginner 目录 | 创建 `07-exercises/beginner/` 目录 |
| [中级练习](./07-exercises/intermediate/) | 07-exercises 下没有 intermediate 目录 | 创建 `07-exercises/intermediate/` 目录 |
| [Python Django Demo](./04-practice/python-django-demo/) | 04-practice 下没有 python-django-demo 目录 | 创建 `04-practice/python-django-demo/` 目录或移除链接 |
| [Go API Demo](./04-practice/go-api-demo/) | 04-practice 下没有 go-api-demo 目录 | 创建 `04-practice/go-api-demo/` 目录或移除链接 |
| [Rust API Demo](./04-practice/rust-api-demo/) | 04-practice 下没有 rust-api-demo 目录 | 创建 `04-practice/rust-api-demo/` 目录或移除链接 |

### ⚠️ 目录结构与 README 不一致的警告

README.md 中描述的项目结构与实际目录结构存在差异：

**README.md 声明应该存在的目录：**
- 07-exercises/beginner/
- 07-exercises/intermediate/
- 07-exercises/advanced/
- 04-practice/python-django-demo/
- 04-practice/go-api-demo/
- 04-practice/rust-api-demo/
- 06-workflows/feature-development.md
- 06-workflows/bug-fix.md
- 06-workflows/refactoring.md
- 06-workflows/decision-tree.md
- 06-workflows/best-practices.md

**实际存在的目录：**
- 07-exercises/ (只有 README.md)
- 04-practice/java-springboot-demo/ (只有这个)
- 06-workflows/ (只有 README.md 和 examples/ 子目录)

## 建议修复方案

### 方案 1：创建缺失的目录结构（推荐）

创建以下目录并添加基础的 README.md 文件：

```bash
# 创建练习目录
mkdir -p 07-exercises/beginner
mkdir -p 07-exercises/intermediate
mkdir -p 07-exercises/advanced

# 创建其他实践项目目录（可选）
mkdir -p 04-practice/python-django-demo
mkdir -p 04-practice/go-api-demo
mkdir -p 04-practice/rust-api-demo

# 创建工作流程文档（或者从 examples/ 复制）
# 06-workflows/ 下已经有 examples/ 目录，可以参考
```

### 方案 2：更新 README.md，移除不存在的内容

如果这些目录不打算创建，可以更新 README.md，移除或注释掉相关链接。

## 总结

- ✅ 正常工作的链接: 12 个
- ❌ 断开的链接: 6 个
- ⚠️ 目录结构不一致: 6 个

**建议**: 优先创建缺失的练习目录结构（07-exercises/beginner/、intermediate/、advanced/），因为这些在文档中被多次引用。
