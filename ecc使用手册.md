# Everything Claude Code (ECC) 使用手册

> **ECC** 是一个 Claude Code 插件，提供生产就绪的代理、技能、命令、规则和 MCP 配置，用于专业的软件开发工作流程。

---

## 目录

- [概述](#概述)
- [核心概念](#核心概念)
- [命令 (Commands)](#命令-commands)
- [代理 (Agents)](#代理-agents)
- [技能 (Skills)](#技能-skills)
- [工作流程指南](#工作流程指南)
- [最佳实践](#最佳实践)

---

## 概述

ECC 提供以下核心组件：

- **🤖 Agents（代理）**：专用的子代理，可以处理特定的开发任务
- **🎯 Commands（命令）**：用户可以直接调用的斜杠命令（`/command`）
- **📚 Skills（技能）**：包含领域知识和最佳实践的文档
- **📋 Rules（规则）**：项目级和全局的开发规范
- **🔌 MCP 配置**：外部服务的集成配置

### 项目结构

```
everything-claude-code/
├── agents/          # 专用子代理（planner, tdd-guide, code-reviewer等）
├── skills/          # 领域知识文档（python-patterns, frontend-patterns等）
├── commands/        # 斜杠命令（/tdd, /plan, /code-review等）
├── rules/           # 开发规范和最佳实践
├── mcp-configs/     # MCP服务器配置
├── hooks/           # 自动化触发器
└── scripts/         # 跨平台工具脚本
```

---

## 核心概念

### Agents（代理）vs Commands（命令）vs Skills（技能）

| 组件 | 触发方式 | 用途 | 示例 |
|-------|---------|------|------|
| **Agents** | 由 `Agent` 工具或命令自动调用 | 执行特定的自动化任务 | `tdd-guide`、`planner`、`code-reviewer` |
| **Commands** | 用户输入 `/command` | 启动特定的开发工作流 | `/tdd`、`/plan`、`e2e` |
| **Skills** | 被 Claude 自动加载 | 提供领域知识和最佳实践指导 | `python-patterns`、`frontend-patterns` |

### 工作流程

典型开发流程：

```
1. /plan     → 创建实现计划
2. /tdd      → 使用测试驱动开发实现
3. /e2e      → 端到端测试
4. /code-review → 代码审查
```

---

## 命令 (Commands)

命令是用户可以直接调用的`/command`格式指令。

### 开发工作流命令

#### `/plan` - 实现计划

**用途**：在编写任何代码之前创建详细的实现计划

**何时使用**：
- 开始新功能开发
- 进行重大架构变更
- 复杂的重构工作
- 需求不明确时

**工作原理**：
1. 分析需求并重述
2. 分解为多个阶段和步骤
3. 识别依赖关系和风险
4. 等待用户确认后再执行

**示例**：

```
用户: /plan 我需要添加实时通知功能，当市场结算时通知用户

Agent (planner):
# 实现计划：实时市场结算通知

## 需求重述
- 向关注市场的用户发送通知
- 支持多种通知渠道（应用内、邮件、webhook）
- 确保通知可靠送达
- 包含市场结果和用户头寸信息

## 实现阶段

### 阶段 1：数据库架构
- 添加 notifications 表
- 添加 user_notification_preferences 表
- 创建索引以优化性能

### 阶段 2：通知服务
- 创建 notification service
- 实现通知队列
- 添加重试逻辑
- 创建通知模板

### 阶段 3：集成点
- 钩入市场结算逻辑
- 查询所有有头寸的用户
- 为每个用户排队通知

### 阶段 4：前端组件
- 创建通知铃铛组件
- 添加通知列表模态框
- 实现实时更新

## 依赖
- Redis（队列）
- 邮件服务
- Supabase 实时订阅

## 风险
- HIGH: 邮件送达性（需要 SPF/DKIM）
- MEDIUM: 1000+ 用户时的性能问题
- MEDIUM: 通知垃圾邮件

**等待确认**：是否继续此计划？（yes/no/modify）
```

**相关代理**：`planner`

---

#### `/tdd` - 测试驱动开发

**用途**：强制执行测试优先的开发流程

**何时使用**：
- 实现新功能
- 添加新函数/组件
- 修复 bug（先写测试重现 bug）
- 重构现有代码
- 构建关键业务逻辑

**工作原理**：
1. 定义接口/类型
2. 先编写失败的测试（RED）
3. 编写最小实现代码（GREEN）
4. 重构代码（REFACTOR）
5. 验证 80%+ 测试覆盖率

**TDD 循环**：

```
RED → GREEN → REFACTOR → REPEAT

RED:      编写失败的测试
GREEN:    编写最小代码使测试通过
REFACTOR: 改进代码，保持测试通过
REPEAT:   下一个功能/场景
```

**示例**：

```
用户: /tdd 我需要一个计算市场流动性分数的函数

Agent (tdd-guide):
# TDD 会话：市场流动性分数计算器

## 步骤 1：定义接口

```typescript
// lib/liquidity.ts
export interface MarketData {
  totalVolume: number
  bidAskSpread: number
  activeTraders: number
  lastTradeTime: Date
}

export function calculateLiquidityScore(market: MarketData): number {
  throw new Error('Not implemented')
}
```

## 步骤 2：编写失败测试

```typescript
// lib/liquidity.test.ts
describe('calculateLiquidityScore', () => {
  it('应为流动性高的市场返回高分', () => {
    const market = {
      totalVolume: 100000,
      bidAskSpread: 0.01,
      activeTraders: 500,
      lastTradeTime: new Date()
    }
    const score = calculateLiquidityScore(market)
    expect(score).toBeGreaterThan(80)
    expect(score).toBeLessThanOrEqual(100)
  })

  it('应为流动性低的市场返回低分', () => {
    const market = {
      totalVolume: 100,
      bidAskSpread: 0.5,
      activeTraders: 2,
      lastTradeTime: new Date(Date.now() - 86400000)
    }
    const score = calculateLiquidityScore(market)
    expect(score).toBeLessThan(30)
  })

  it('应处理边界情况：零成交量', () => {
    const market = {
      totalVolume: 0,
      bidAskSpread: 0,
      activeTraders: 0,
      lastTradeTime: new Date()
    }
    const score = calculateLiquidityScore(market)
    expect(score).toBe(0)
  })
})
```

## 步骤 3：运行测试 - 验证失败

```bash
npm test lib/liquidity.test.ts

FAIL lib/liquidity.test.ts
  ✕ should return high score for liquid market
    Error: Not implemented

1 test failed, 0 passed
```

✅ 测试按预期失败。准备实现。

## 步骤 4：实现最小代码

```typescript
// lib/liquidity.ts
export function calculateLiquidityScore(market: MarketData): number {
  if (market.totalVolume === 0) {
    return 0
  }

  const volumeScore = Math.min(market.totalVolume / 1000, 100)
  const spreadScore = Math.max(100 - (market.bidAskSpread * 1000), 0)
  const traderScore = Math.min(market.activeTraders / 10, 100)

  const hoursSinceLastTrade = (Date.now() - market.lastTradeTime.getTime()) / (1000 * 60 * 60)
  const recencyScore = Math.max(100 - (hoursSinceLastTrade * 10), 0)

  const score = (
    volumeScore * 0.4 +
    spreadScore * 0.3 +
    traderScore * 0.2 +
    recencyScore * 0.1
  )

  return Math.min(Math.max(score, 0), 100)
}
```

## 步骤 5：运行测试 - 验证通过

```bash
npm test lib/liquidity.test.ts

PASS lib/liquidity.test.ts
  ✓ should return high score for liquid market
  ✓ should return low score for illiquid market
  ✓ should handle edge case: zero volume

3 tests passed
```

✅ 所有测试通过！

## 步骤 6：重构（改进）

[使用常量、提取辅助函数等重构代码]

## 步骤 7：验证测试仍然通过
## 步骤 8：检查覆盖率

```bash
npm test -- --coverage

File           | % Stmts | % Branch | % Funcs | % Lines
---------------|---------|----------|---------|--------
liquidity.ts   |   100   |   100    |   100   |   100

Coverage: 100% ✅ (目标: 80%)
```

✅ TDD 会话完成！
```

**相关代理**：`tdd-guide`

---

#### `/e2e` - 端到端测试

**用途**：使用 Playwright 生成、维护和执行端到端测试

**何时使用**：
- 测试关键用户流程（登录、交易、支付）
- 验证多步骤流程
- 测试 UI 交互和导航
- 验证前后端集成
- 准备生产部署

**工作原理**：
1. 分析用户流程并识别测试场景
2. 使用页面对象模型模式生成 Playwright 测试
3. 跨多个浏览器运行测试
4. 捕获失败时的截图、视频和跟踪
5. 生成报告和结果

**示例**：

```
用户: /e2e 测试市场搜索和查看流程

Agent (e2e-runner):
# E2E 测试生成：市场搜索和查看流程

## 测试场景识别

**用户流程**：市场搜索 → 查看结果 → 点击市场 → 查看详情

## 生成的测试代码

```typescript
// tests/e2e/markets/search-and-view.spec.ts
import { test, expect } from '@playwright/test'
import { MarketsPage } from '../../pages/MarketsPage'
import { MarketDetailsPage } from '../../pages/MarketDetailsPage'

test.describe('Market Search and View Flow', () => {
  test('user can search markets and view details', async ({ page }) => {
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // 执行语义搜索
    await marketsPage.searchMarkets('election')

    // 等待 API 响应
    await page.waitForResponse(resp =>
      resp.url().includes('/api/markets/search') && resp.status() === 200
    )

    // 验证搜索结果
    const marketCards = marketsPage.marketCards
    await expect(marketCards.first()).toBeVisible()
    const resultCount = await marketCards.count()
    expect(resultCount).toBeGreaterThan(0)

    // 点击第一个结果
    const firstMarketTitle = await marketCards.first().textContent()
    await marketCards.first().click()

    // 验证市场详情页面加载
    await expect(page).toHaveURL(/\/markets\/[a-z0-9-]+/)

    const detailsPage = new MarketDetailsPage(page)
    await expect(detailsPage.marketName).toBeVisible()
    await expect(detailsPage.priceChart).toBeVisible()
  })
})
```

## 运行测试

```bash
npx playwright test tests/e2e/markets/search-and-view.spec.ts

Running 3 tests using 3 workers

  ✓  [chromium] › user can search markets and view details (4.2s)
  ✓  [chromium] › search with no results shows empty state (1.8s)
  ✓  [chromium] › can clear search and see all markets again (2.9s)

  3 passed (9.1s)
```

## 测试报告

```
╔══════════════════════════════════════════════════════════════╗
║                    E2E Test Results                          ║
╠══════════════════════════════════════════════════════════════╣
║ Status:     ✅ ALL TESTS PASSED                              ║
║ Total:      3 tests                                          ║
║ Passed:     3 (100%)                                         ║
║ Failed:     0                                                ║
║ Duration:   9.1s                                             ║
╚══════════════════════════════════════════════════════════════╝
```
```

**相关代理**：`e2e-runner`

---

#### `/code-review` - 代码审查

**用途**：对未提交的更改进行全面的安全和质量审查

**何时使用**：
- 编写或修改代码后
- 提交代码前
- PR 创建前

**审查内容**：

**安全问题（CRITICAL）**：
- 硬编码凭证、API 密钥、令牌
- SQL 注入漏洞
- XSS 漏洞
- 缺少输入验证
- 不安全的依赖项

**代码质量（HIGH）**：
- 函数超过 50 行
- 文件超过 800 行
- 嵌套深度超过 4 层
- 缺少错误处理
- console.log 语句
- TODO/FIXME 注释

**最佳实践（MEDIUM）**：
- 变异模式（应使用不可变操作）
- 缺少测试的新代码
- 可访问性问题

**示例输出**：

```
[CRITICAL] Hardcoded API key in source
File: src/api/client.ts:42
Issue: API key "sk-abc..." exposed in source code. This will be committed to git history.
Fix: Move to environment variable and add to .gitignore/.env.example

  const apiKey = "sk-abc123";           // BAD
  const apiKey = process.env.API_KEY;   // GOOD

[HIGH] Function too long (75 lines)
File: src/utils/dataProcessor.ts:10
Issue: Function exceeds 50-line limit, making it hard to understand and test.
Fix: Extract helper functions to reduce complexity

## Review Summary

| Severity | Count | Status |
|----------|-------|--------|
| CRITICAL | 1     | block  |
| HIGH     | 2     | warn   |
| MEDIUM   | 3     | info   |
| LOW      | 1     | note   |

Verdict: BLOCK — 1 CRITICAL issue must be fixed before merge.
```

**相关代理**：`code-reviewer`

---

### 学习和提取命令

#### `/learn` - 提取可重用模式

**用途**：分析当前会话并提取值得保存为技能的模式

**何时使用**：
- 解决了一个非平凡问题后
- 发现了有用的调试技术
- 找到了项目特定的约定

**提取内容**：

1. **错误解决模式**
   - 发生了什么错误？
   - 根本原因是什么？
   - 如何修复的？
   - 这对类似错误可重用吗？

2. **调试技术**
   - 非显而易见的调试步骤
   - 有效的工具组合
   - 诊断模式

3. **变通方法**
   - 库的怪癖
   - API 限制
   - 版本特定的修复

4. **项目特定模式**
   - 发现的代码库约定
   - 架构决策
   - 集成模式

**输出格式**：

创建技能文件：`~/.claude/skills/learned/[pattern-name].md`

```markdown
# [描述性模式名称]

**提取日期**: [日期]
**上下文**: [此模式适用时的简要描述]

## 问题
[此模式解决的问题 - 具体说明]

## 解决方案
[模式/技术/变通方法]

## 示例
[适用的代码示例]

## 何时使用
[触发条件 - 什么应该激活此技能]
```

---

### 构建和修复命令

#### `/build-fix` - 修复构建错误

**用途**：以最小、安全的更改逐步修复构建和类型错误

**工作流程**：

**步骤 1：检测构建系统**

| 指示器 | 构建命令 |
|---------|----------|
| `package.json` | `npm run build` 或 `pnpm build` |
| `tsconfig.json` | `npx tsc --noEmit` |
| `Cargo.toml` | `cargo build` |
| `pom.xml` | `mvn compile` |
| `build.gradle` | `./gradlew compileJava` |
| `go.mod` | `go build ./...` |
| `pyproject.toml` | `python -m py_compile` |

**步骤 2：解析和分组错误**
- 运行构建命令并捕获错误输出
- 按文件路径分组错误
- 按依赖顺序排序

**步骤 3：修复循环（一次修复一个错误）**
1. 读取文件 - 查看错误上下文
2. 诊断 - 识别根本原因
3. 最小修复 - 使用最小更改解决错误
4. 重新运行构建 - 验证错误已消失
5. 移至下一个错误

**步骤 4：保护措施**
如果以下情况则停止并询问用户：
- 修复引入的错误比解决的多
- 同一错误在 3 次尝试后仍然存在
- 修复需要架构更改
- 构建错误源于缺少依赖项

---

### 语言特定命令

#### Kotlin 相关命令

| 命令 | 用途 |
|-------|------|
| `/kotlin-build` | 修复 Kotlin/Gradle 构建错误 |
| `/kotlin-review` | Kotlin 和 Android/KMP 代码审查 |
| `/kotlin-test` | 强制 Kotlin 项目的 TDD 工作流 |

#### C++ 相关命令

| 命令 | 用途 |
|-------|------|
| `/cpp-build` | 修复 C++ 构建、CMake 和链接器问题 |
| `/cpp-review` | C++ 代码审查（内存安全、现代 C++） |
| `/cpp-test` | 强制 C++ 项目的 TDD 工作流 |

#### Go 相关命令

| 命令 | 用途 |
|-------|------|
| `/go-build` | 修复 Go 构建、go vet 和 linter 警告 |
| `/go-review` | Go 代码审查（惯用模式、并发安全） |
| `/go-test` | 强制 Go 项目的 TDD 工作流 |

#### Rust 相关命令

| 命令 | 用途 |
|-------|------|
| `/rust-build` | 修复 Rust 构建、借用检查器问题 |
| `/rust-review` | Rust 代码审查（所有权、生命周期） |
| `/rust-test` | 强制 Rust 项目的 TDD 工作流 |

#### Python 相关命令

| 命令 | 用途 |
|-------|------|
| `/python-review` | Python 代码审查（PEP 8、类型提示） |

#### Java 相关命令

| 命令 | 用途 |
|-------|------|
| `/gradle-build` | 修复 Android 和 KMP 项目的 Gradle 构建错误 |

---

### 高级命令

#### `/multi-plan` - 多模型协作规划

**用途**：使用多个模型协作进行规划和设计

#### `/multi-execute` - 多模型协作执行

**用途**：使用多个模型协作执行任务

#### `/multi-workflow` - 多模型协作开发

**用途**：工作流 - 多模型协作开发

#### `/multi-frontend` - 前端专注开发

**用途**：前端 - 前端专注开发

#### `/multi-backend` - 后端专注开发

**用途**：后端 - 后端专注开发

#### `/devfleet` - 并行 Claude Code 代理

**用途**：通过 Claude DevFleet 协调并行代理

#### `/orchestrate` - 顺序和 tmux/worktree 编排

**用途**：多代理工作流的顺序和 tmux/worktree 编排指导

#### `/verify` - 验证命令

**用途**：验证实施内容

#### `/quality-gate` - 质量门禁

**用途**：质量门禁命令

---

### 会话管理命令

#### `/save-session` - 保存会话

**用途**：将当前会话状态保存到日期文件，以便在未来会话中恢复工作

#### `/resume-session` - 恢复会话

**用途**：从 `~/.claude/sessions/` 加载最近的会话文件并恢复工作

#### `/sessions` - 管理会话

**用途**：管理 Claude Code 会话历史、别名和会话元数据

#### `/checkpoint` - 检查点

**用途**：检查点命令

---

### 文档和学习命令

#### `/docs` - 文档查找

**用途**：通过 Context7 查找库或主题的当前文档

#### `/update-docs` - 更新文档

**用途**：更新文档

#### `/update-codemaps` - 更新代码图

**用途**：更新代码图

---

### Instinct 管理命令

#### `/instinct-status` - 显示学习到的本能

**用途**：显示学习到的本能（项目 + 全局）及置信度

#### `/instinct-import` - 导入本能

**用途**：从文件或 URL 导入本能到项目/全局范围

#### `/instinct-export` - 导出本能

**用途**：从项目/全局范围导出本能到文件

#### `/promote` - 提升项目作用域本能

**用途**：将项目作用域的本能提升到全局范围

#### `/evolve` - 分析并建议或生成演进结构

**用途**：分析本能并建议或生成演进结构

---

### 测试和评估命令

#### `/test-coverage` - 测试覆盖率

**用途**：测试覆盖率

#### `/eval` - 评估命令

**用途**：评估命令

---

### 工具和配置命令

#### `/harness-audit` - Harness 审计

**用途**：Harness 审计命令

#### `/setup-pm` - 设置包管理器

**用途**：设置包管理器

#### `/pm2` - PM2 初始化

**用途**：PM2 初始化

#### `/model-route` - 模型路由

**用途**：模型路由命令

#### `/loop-start` - 循环开始

**用途**：循环开始命令

#### `/loop-status` - 循环状态

**用途**：循环状态命令

---

### 杂项命令

| 命令 | 用途 |
|-------|------|
| `/aside` | 在不中断或丢失上下文的情况下回答快速问题 |
| `/claw` | 启动 NanoClaw v2 |
| `/skill-create` | 从 git 历史生成技能 |
| `/skill-health` | 显示技能组合健康状况仪表板 |
| `/refactor-clean` | 重构清理 |
| `/prompt-optimize` | 分析草稿提示并输出优化版本 |
| `/projects` | 列出已知项目及其本能统计 |

---

## 代理 (Agents)

代理是专用的子代理，可以被命令或 `Agent` 工具调用。每个代理都有特定的工具集和专长。

### 开发工作流代理

#### tdd-guide

**用途**：测试驱动开发专家，强制执行测试优先的方法

**工具**：`Read`, `Write`, `Edit`, `Bash`, `Grep`

**模型**：Sonnet 4.6

**核心职责**：
- 强制测试优先代码方法
- 指导 Red-Green-Refactor 循环
- 确保 80%+ 测试覆盖率
- 编写全面的测试套件（单元、集成、E2E）
- 在实现前捕获边界情况

**TDD 工作流程**：

1. **编写测试优先（RED）**
   - 编写描述预期行为的失败测试

2. **运行测试 - 验证失败**
   ```bash
   npm test
   ```

3. **编写最小实现（GREEN）**
   - 仅编写足够的代码使测试通过

4. **运行测试 - 验证通过**

5. **重构（IMPROVE）**
   - 删除重复、改进名称、优化 - 测试必须保持绿色

6. **验证覆盖率**
   ```bash
   npm run test:coverage
   # 要求：80%+ 分支、函数、行、语句
   ```

**必须测试的边界情况**：
1. Null/Undefined 输入
2. 空数组/字符串
3. 无效类型传入
4. 边界值（最小/最大）
5. 错误路径（网络失败、数据库错误）
6. 竞争条件（并发操作）
7. 大数据（10k+ 项的性能）
8. 特殊字符（Unicode、表情符号、SQL 字符）

**质量检查清单**：
- [ ] 所有公共函数都有单元测试
- [ ] 所有 API 端点都有集成测试
- [ ] 关键用户流程有 E2E 测试
- [ ] 边界情况已覆盖（null、空、无效）
- [ ] 错误路径已测试（不仅是快乐路径）
- [ ] 使用了外部依赖的模拟
- [ ] 测试是独立的（无共享状态）
- [ ] 断言是具体且有意义的
- [ ] 覆盖率是 80%+

---

#### planner

**用途**：规划专家，为复杂功能和重构创建详细的实施计划

**工具**：`Read`, `Grep`, `Glob`

**模型**：Opus 4.6（最深度推理）

**核心职责**：
- 分析需求并创建详细的实施计划
- 将复杂功能分解为可管理的步骤
- 识别依赖关系和潜在风险
- 建议最佳实施顺序
- 考虑边界情况和错误场景

**规划过程**：

**1. 需求分析**
- 完全理解功能请求
- 如需要，提出澄清问题
- 识别成功标准
- 列出假设和约束

**2. 架构审查**
- 分析现有代码库结构
- 识别受影响的组件
- 审查类似实现
- 考虑可重用模式

**3. 步骤分解**
创建详细步骤，包括：
- 清晰、具体的操作
- 文件路径和位置
- 步骤之间的依赖关系
- 预估复杂度
- 潜在风险

**4. 实施顺序**
- 按依赖关系排序
- 分组相关更改
- 最小化上下文切换
- 启用增量测试

**计划格式**：

```markdown
# 实施计划：[功能名称]

## 概述
[2-3 句话总结]

## 需求
- [需求 1]
- [需求 2]

## 架构变更
- [变更 1：文件路径和描述]
- [变更 2：文件路径和描述]

## 实施步骤

### 阶段 1：[阶段名称]
1. **[步骤名称]** (文件: path/to/file.ts)
   - 操作：要采取的具体操作
   - 原因：此步骤的原因
   - 依赖：无 / 需要步骤 X
   - 风险：低/中/高

### 阶段 2：[阶段名称]
...

## 测试策略
- 单元测试：[要测试的文件]
- 集成测试：[要测试的流程]
- E2E 测试：[要测试的用户流程]

## 风险和缓解措施
- **风险**: [描述]
  - 缓解: [如何解决]

## 成功标准
- [ ] 标准 1
- [ ] 标准 2
```

**最佳实践**：
1. **具体化**：使用确切的文件路径、函数名、变量名
2. **考虑边界情况**：考虑错误场景、null 值、空状态
3. **最小化更改**：优先扩展现有代码而非重写
4. **维护模式**：遵循现有项目约定
5. **启用测试**：构建易于测试的更改
6. **增量思考**：每个步骤都应可验证
7. **记录决策**：解释原因，而不仅仅是什么

---

#### code-reviewer

**用途**：专家代码审查专家，主动审查代码质量、安全性和可维护性

**工具**：`Read`, `Grep`, `Glob`, `Bash`

**模型**：Sonnet 4.6

**核心职责**：
- 对所有代码更改进行质量审查
- 识别安全问题
- 建议改进和最佳实践
- 确保代码符合项目标准

**审查流程**：

1. **收集上下文** - 运行 `git diff --staged` 和 `git diff` 查看所有更改
2. **理解范围** - 识别哪些文件更改，它们与什么功能/修复相关
3. **阅读周围代码** - 不要孤立地审查更改
4. **应用审查检查清单** - 从 CRITICAL 到 LOW 逐一处理每个类别
5. **报告发现** - 使用输出格式。只报告你有信心的问题（>80% 确定）

**置信度过滤**：

- **报告** 如果你 >80% 确定这是真正的问题
- **跳过** 风格偏好，除非违反项目约定
- **跳过** 未更改代码中的问题，除非是 CRITICAL 安全问题
- **合并** 类似问题（例如，"5 个函数缺少错误处理"而不是 5 个单独发现）
- **优先** 可能导致错误、安全漏洞或数据丢失的问题

**审查检查清单**：

**安全（CRITICAL）**：
- 硬编码凭证 — API 密钥、密码、令牌、连接字符串
- SQL 注入 — 查询中的字符串拼接而非参数化查询
- XSS 漏洞 — 在 HTML/JSX 中渲染未转义的用户输入
- 路径遍历 — 未经清理的用户控制文件路径
- CSRF 漏洞 — 无 CSRF 保护的更改状态端点
- 身份验证绕过 — 受保护路由上缺少身份验证检查
- 不安全的依赖项 — 已知易受攻击的包
- 日志中暴露的秘密 — 记录敏感数据（令牌、密码、PII）

**代码质量（HIGH）**：
- 大函数（>50 行）
- 大文件（>800 行）
- 深度嵌套（>4 层）
- 缺少错误处理
- 变异模式 — 优先使用不可变操作
- console.log 语句
- 缺少测试的新代码
- 死代码 — 注释掉的代码、未使用的导入、无法到达的分支

**React/Next.js 模式（HIGH）**：
- 缺少依赖数组 — `useEffect`/`useMemo`/`useCallback` 的依赖不完整
- 渲染中的状态更新 — 在渲染期间调用 setState 导致无限循环
- 列表中缺少键 — 当项目可以重新排序时使用数组索引作为键
- Prop 钻取 — 通过 3+ 层传递 props
- 不必要的重渲染 — 缺少昂贵计算的内存优化
- 客户端/服务器边界 — 在服务器组件中使用 `useState`/`useEffect`
- 缺少加载/错误状态 — 没有回退 UI 的数据获取
- 停滞闭包 — 捕获停滞状态值的事件处理程序

**Node.js/后端模式（HIGH）**：
- 未验证的输入 — 请求体/参数使用时未经架构验证
- 缺少速率限制 — 无节流的公共端点
- 无界查询 — 用户界面端点上的 `SELECT *` 或无 LIMIT 的查询
- N+1 查询 — 在循环中而不是连接/批处理中获取相关数据
- 缺少超时 — 无超时配置的外部 HTTP 调用
- 错误消息泄漏 — 向客户端发送内部错误详细信息
- 缺少 CORS 配置 — 可从非预期源访问的 API

**性能（MEDIUM）**：
- 低效算法 — 当 O(n log n) 或 O(n) 可能时使用 O(n^2)
- 不必要的重渲染
- 大包大小 — 当可使用 tree-shakeable 替代方案时导入整个库
- 缺少缓存 — 无内存化的重复昂贵计算
- 未优化的图像 — 无压缩或延迟加载的大图像
- 同步 I/O — 异步上下文中的阻塞操作

**最佳实践（LOW）**：
- 无工单的 TODO/FIXME — TODO 应引用工单号
- 公共 API 缺少 JSDoc — 导出函数无文档
- 命名不佳 — 非平凡上下文中的单字母变量（x、tmp、data）
- 魔法数字 — 未解释的数字常量
- 格式不一致 — 混合的分号、引号样式、缩进

**批准标准**：
- **批准**：无 CRITICAL 或 HIGH 问题
- **警告**：仅 HIGH 问题（可谨慎合并）
- **阻止**：发现 CRITICAL 问题 — 合并前必须修复

---

#### e2e-runner

**用途**：端到端测试专家，使用 Playwright 生成和维护测试

**工具**：所有工具（包括 Playwright 相关工具）

**模型**：Sonnet 4.6

**核心职责**：
- 生成 Playwright 测试用于用户流程
- 运行 E2E 测试
- 捕获截图/视频/跟踪
- 识别不稳定测试

**测试工件**：

**所有测试**：
- 带时间线和结果的 HTML 报告
- CI 集成的 JUnit XML

**仅失败时**：
- 失败状态的截图
- 测试的视频录制
- 用于调试的跟踪文件（逐步重放）
- 网络日志
- 控制台日志

---

### 语言特定审查代理

#### python-reviewer

**用途**：Python 代码审查专家，专注于 PEP 8 合规、类型提示、安全性和 Pythonic 惯用

**模型**：Sonnet 4.6

**关注领域**：
- PEP 8 标准合规
- 类型提示和类型安全
- Pythonic 惯用模式
- 安全漏洞
- 性能问题

---

#### go-reviewer

**用途**：Go 代码审查专家，专注于惯用模式、并发安全和错误处理

**模型**：Sonnet 4.6

**关注领域**：
- 惯用 Go 模式
- 并发安全性
- 错误处理
- 性能
- 安全

---

#### rust-reviewer

**用途**：Rust 代码审查专家，专注于所有权、生命周期、错误处理和惯用模式

**模型**：Sonnet 4.6

**关注领域**：
- 所有权和借用
- 生命周期
- 错误处理
- Unsafe 用法
- 惯用模式

---

#### cpp-reviewer

**用途**：C++ 代码审查专家，专注于内存安全、现代 C++ 惯用、并发和性能

**模型**：Sonnet 4.6

**关注领域**：
- 内存安全
- 现代 C++ 惯用
- 并程
- 性能
- 安全

---

#### kotlin-reviewer

**用途**：Kotlin 和 Android/KMP 代码审查专家

**模型**：Sonnet 4.6

**关注领域**：
- Kotlin 惯用模式
- 空安全
- 协程安全
- Compose 最佳实践
- 架构违规

---

#### java-reviewer

**用途**：Java 和 Spring Boot 代码审查专家

**模型**：Sonnet 4.6

**关注领域**：
- 分层架构
- JPA 模式
- 安全
- 并程

---

### 构建错误解决代理

#### build-error-resolver

**用途**：修复构建和 TypeScript 错误

**关注**：TypeScript、JavaScript、构建配置

---

#### go-build-resolver

**用途**：修复 Go 构建、go vet 和 linter 警告

---

#### rust-build-resolver

**用途**：修复 Rust 构建、借用检查器问题和依赖项

---

#### kotlin-build-resolver

**用途**：修复 Kotlin/Gradle 构建错误、编译器警告和依赖项

---

#### cpp-build-resolver

**用途**：修复 C++ 构建、CMake 和链接器问题

---

#### java-build-resolver

**用途**：修复 Java/Maven/Gradle 构建、编译错误和依赖项

---

### 其他专业代理

#### architect

**用途**：系统设计专家，用于可扩展性、性能和技术决策

**何时使用**：
- 进行架构决策
- 设计新系统
- 评估技术选择

---

#### security-reviewer

**用途**：安全漏洞检测和修复专家

**何时使用**：
- **主动使用**编写处理用户输入、身份验证、API 端点或敏感数据的代码后
- 发现安全问题必须立即修复

**关注领域**：
- OWASP Top 10 漏洞
- 密钥管理
- 输入验证
- 输出编码
- 身份验证和授权

---

#### database-reviewer

**用途**：PostgreSQL 数据库专家

**关注领域**：
- 查询优化
- 架构设计
- 安全性
- 性能

---

#### refactor-cleaner

**用途**：死代码清理和整合专家

**何时使用**：
- **主动使用**删除未使用的代码、重复项和重构

---

#### docs-lookup

**用途**：通过 Context7 MCP 查找文档

---

#### doc-updater

**用途**：更新代码图和文档

---

#### harness-optimizer

**用途**：分析和改进本地代理 harness 配置

---

#### loop-operator

**用途**：操作自主代理循环，监控进度并在循环停顿时安全介入

---

#### chief-of-staff

**用途**：管理来自多个渠道（Slack、LINE、邮件、Messenger）的通信工作流

---

---

## 技能 (Skills)

技能是包含领域知识和最佳实践的文档，为 Claude 提供指导。技能按语言和框架组织。

### Python 开发技能

#### python-patterns

**用途**：Pythonic 惯用、PEP 8 标准、类型提示和构建健壮、高效、可维护应用程序的最佳实践

**何时激活**：
- 编写新 Python 代码
- 审查 Python 代码
- 重构现有 Python 代码
- 设计 Python 包/模块

**核心原则**：

**1. 可读性优先**

```python
# 好的：清晰且可读
def get_active_users(users: list[User]) -> list[User]:
    """仅返回提供的列表中的活动用户。"""
    return [user for user in users if user.is_active]

# 坏的：聪明但混乱
def get_active_users(u):
    return [x for x in u if x.a]
```

**2. 显式优于隐式**

```python
# 好的：显式配置
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

# 坏的：隐藏副作用
import some_module
some_module.setup()  # 这有什么作用？
```

**3. EAFP - 宽恕比许可更容易**

```python
# 好的：EAFP 风格
def get_value(dictionary: dict, key: str) -> Any:
    try:
        return dictionary[key]
    except KeyError:
        return default_value

# 坏的：LBYL（三思而后行）风格
def get_value(dictionary: dict, key: str) -> Any:
    if key in dictionary:
        return dictionary[key]
    else:
        return default_value
```

**类型提示**：

```python
from typing import Optional, List, Dict, Any

def process_user(
    user_id: str,
    data: Dict[str, Any],
    active: bool = True
) -> Optional[User]:
    """处理用户并返回更新的 User 或 None。"""
    if not active:
        return None
    return User(user_id, data)
```

**现代类型提示（Python 3.9+）**：

```python
# Python 3.9+ - 使用内置类型
def process_items(items: list[str]) -> dict[str, int]:
    return {item: len(item) for item in items}

# Python 3.8 及更早版本 - 使用 typing 模块
from typing import List, Dict

def process_items(items: List[str]) -> Dict[str, int]:
    return {item: len(item) for item in items}
```

**错误处理模式**：

```python
# 好的：捕获特定异常
def load_config(path: str) -> Config:
    try:
        with open(path) as f:
            return Config.from_json(f.read())
    except FileNotFoundError as e:
        raise ConfigError(f"配置文件未找到：{path}") from e
    except json.JSONDecodeError as e:
        raise ConfigError(f"配置中的 JSON 无效：{path}") from e

# 坏的：裸 except
def load_config(path: str) -> Config:
    try:
        with open(path) as f:
            return Config.from_json(f.read())
    except:
        return None  # 静默失败！
```

**上下文管理器**：

```python
# 好的：使用上下文管理器
def process_file(path: str) -> str:
    with open(path, 'r') as f:
        return f.read()

# 坏的：手动资源管理
def process_file(path: str) -> str:
    f = open(path, 'r')
    try:
        return f.read()
    finally:
        f.close()
```

**数据类**：

```python
from dataclasses import dataclass
from datetime import datetime

@dataclass
class User:
    """用户实体，自动生成 __init__、__repr__ 和 __eq__。"""
    id: str
    name: str
    email: str
    created_at: datetime = field(default_factory=datetime.now)
    is_active: bool = True

# 使用
user = User(
    id="123",
    name="Alice",
    email="alice@example.com"
)
```

**反模式避免**：

```python
# 坏的：可变默认参数
def append_to(item, items=[]):
    items.append(item)
    return items

# 好的：使用 None 并创建新列表
def append_to(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items

# 坏的：使用 type() 检查类型
if type(obj) == list:
    process(obj)

# 好的：使用 isinstance
if isinstance(obj, list):
    process(obj)

# 坏的：与 None 比较
if value == None:
    process()

# 好的：使用 is
if value is None:
    process()
```

---

#### python-testing

**用途**：使用 pytest、TDD 方法、fixtures、模拟、参数化和覆盖率要求的 Python 测试策略

---

### 前端开发技能

#### frontend-patterns

**用途**：React、Next.js、状态管理、性能优化和 UI 最佳实践的前端开发模式

**何时激活**：
- 构建 React 组件（组合、props、渲染）
- 管理状态（useState、useReducer、Zustand、Context）
- 实现数据获取（SWR、React Query、服务器组件）
- 优化性能（内存化、虚拟化、代码拆分）
- 处理表单（验证、受控输入、Zod 架构）
- 处理客户端路由和导航
- 构建可访问、响应式 UI 模式

**组件模式**：

**组合优于继承**：

```typescript
// ✅ 好的：组件组合
interface CardProps {
  children: React.ReactNode
  variant?: 'default' | 'outlined'
}

export function Card({ children, variant = 'default' }: CardProps) {
  return <div className={`card card-${variant}`}>{children}</div>
}

export function CardHeader({ children }: { children: React.ReactNode }) {
  return <div className="card-header">{children}</div>
}

export function CardBody({ children }: { children: React.ReactNode }) {
  return <div className="card-body">{children}</div>
}

// 使用
<Card>
  <CardHeader>标题</CardHeader>
  <CardBody>内容</CardBody>
</Card>
```

**复合组件**：

```typescript
interface TabsContextValue {
  activeTab: string
  setActiveTab: (tab: string) => void
}

const TabsContext = createContext<TabsContextValue | undefined>(undefined)

export function Tabs({ children, defaultTab }: {
  children: React.ReactNode
  defaultTab: string
}) {
  const [activeTab, setActiveTab] = useState(defaultTab)

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      {children}
    </TabsContext.Provider>
  )
}

export function TabList({ children }: { children: React.ReactNode }) {
  return <div className="tab-list">{children}</div>
}

export function Tab({ id, children }: { id: string, children: React.ReactNode }) {
  const context = useContext(TabsContext)
  if (!context) throw new Error('Tab must be used within Tabs')

  return (
    <button
      className={context.activeTab === id ? 'active' : ''}
      onClick={() => context.setActiveTab(id)}
    >
      {children}
    </button>
  )
}

// 使用
<Tabs defaultTab="overview">
  <TabList>
    <Tab id="overview">概览</Tab>
    <Tab id="details">详情</Tab>
  </TabList>
</Tabs>
```

**渲染 Props 模式**：

```typescript
interface DataLoaderProps<T> {
  url: string
  children: (data: T | null, loading: boolean, error: Error | null) => React.ReactNode
}

export function DataLoader<T>({ url, children }: DataLoaderProps<T>) {
  const [data, setData] = useState<T | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<Error | null>(null)

  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false))
  }, [url])

  return <>{children(data, loading, error)}</>
}

// 使用
<DataLoader<Market[]> url="/api/markets">
  {(markets, loading, error) => {
    if (loading) return <Spinner />
    if (error) return <Error error={error} />
    return <MarketList markets={markets!} />
  }}
</DataLoader>
```

**自定义 Hooks 模式**：

```typescript
export function useToggle(initialValue = false): [boolean, () => void] {
  const [value, setValue] = useState(initialValue)

  const toggle = useCallback(() => {
    setValue(v => !v)
  }, [])

  return [value, toggle]
}

// 使用
const [isOpen, toggleOpen] = useToggle()
```

**状态管理模式**：

```typescript
interface State {
  markets: Market[]
  selectedMarket: Market | null
  loading: boolean
}

type Action =
  | { type: 'SET_MARKETS'; payload: Market[] }
  | { type: 'SELECT_MARKET'; payload: Market }
  | { type: 'SET_LOADING'; payload: boolean }

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'SET_MARKETS':
      return { ...state, markets: action.payload }
    case 'SELECT_MARKET':
      return { ...state, selectedMarket: action.payload }
    case 'SET_LOADING':
      return { ...state, loading: action.payload }
    default:
      return state
  }
}

const MarketContext = createContext<{
  state: State
  dispatch: Dispatch<Action>
} | undefined>(undefined)

export function MarketProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(reducer, {
    markets: [],
    selectedMarket: null,
    loading: false
  })

  return (
    <MarketContext.Provider value={{ state, dispatch }}>
      {children}
    </MarketContext.Provider>
  )
}

export function useMarkets() {
  const context = useContext(MarketContext)
  if (!context) throw new Error('useMarkets must be used within MarketProvider')
  return context
}
```

**性能优化**：

```typescript
// ✅ useMemo 用于昂贵计算
const sortedMarkets = useMemo(() => {
  return markets.sort((a, b) => b.volume - a.volume)
}, [markets])

// ✅ useCallback 用于传递给子组件的函数
const handleSearch = useCallback((query: string) => {
  setSearchQuery(query)
}, [])

// ✅ React.memo 用于纯组件
export const MarketCard = React.memo<MarketCardProps>(({ market }) => {
  return (
    <div className="market-card">
      <h3>{market.name}</h3>
      <p>{market.description}</p>
    </div>
  )
})
```

**代码拆分和延迟加载**：

```typescript
import { lazy, Suspense } from 'react'

// ✅ 延迟加载重组件
const HeavyChart = lazy(() => import('./HeavyChart'))
const ThreeJsBackground = lazy(() => import('./ThreeJsBackground'))

export function Dashboard() {
  return (
    <div>
      <Suspense fallback={<ChartSkeleton />}>
        <HeavyChart data={data} />
      </Suspense>

      <Suspense fallback={null}>
        <ThreeJsBackground />
      </Suspense>
    </div>
  )
}
```

**表单处理模式**：

```typescript
interface FormData {
  name: string
  description: string
  endDate: string
}

interface FormErrors {
  name?: string
  description?: string
  endDate?: string
}

export function CreateMarketForm() {
  const [formData, setFormData] = useState<FormData>({
    name: '',
    description: '',
    endDate: ''
  })

  const [errors, setErrors] = useState<FormErrors>({})

  const validate = (): boolean => {
    const newErrors: FormErrors = {}

    if (!formData.name.trim()) {
      newErrors.name = '名称是必需的'
    } else if (formData.name.length > 200) {
      newErrors.name = '名称必须小于 200 个字符'
    }

    if (!formData.description.trim()) {
      newErrors.description = '描述是必需的'
    }

    if (!formData.endDate) {
      newErrors.endDate = '结束日期是必需的'
    }

    setErrors(newErrors)
    return Object.keys(newErrors).length === 0
  }

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()

    if (!validate()) return

    try {
      await createMarket(formData)
    } catch (error) {
      // 错误处理
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={formData.name}
        onChange={e => setFormData(prev => ({ ...prev, name: e.target.value }))}
        placeholder="市场名称"
      />
      {errors.name && <span className="error">{errors.name}</span>
      {/* 其他字段 */}
      <button type="submit">创建市场</button>
    </form>
  )
}
```

**错误边界模式**：

```typescript
interface ErrorBoundaryState {
  hasError: boolean
  error: Error | null
}

export class ErrorBoundary extends React.Component<
  { children: React.ReactNode },
  ErrorBoundaryState
> {
  state: ErrorBoundaryState = {
    hasError: false,
    error: null
  }

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error }
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Error boundary caught:', error, errorInfo)
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-fallback">
          <h2>出了点问题</h2>
          <p>{this.state.error?.message}</p>
          <button onClick={() => this.setState({ hasError: false })}>
            再试一次
          </button>
        </div>
      )
    }

    return this.props.children
  }
}

// 使用
<ErrorBoundary>
  <App />
</ErrorBoundary>
```

---

#### frontend-slides

**用途**：从头开始或通过转换 PowerPoint 文件创建令人惊叹、富含动画的 HTML 演示文稿

---

#### frontend-design

**用途**：创建独特、生产级的前端界面，具有高设计质量

---

### Go 开发技能

#### golang-patterns

**用途**：惯用 Go 模式、最佳实践和构建健壮、高效、可维护 Go 应用程序的约定

---

#### golang-testing

**用途**：使用表驱动测试、子测试、基准测试、fuzzing 和测试覆盖率的 Go 测试模式

---

### Rust 开发技能

#### rust-patterns

**用途**：惯用 Rust 模式、所有权、错误处理、traits、并发和构建安全、高效、可维护应用程序的最佳实践

---

#### rust-testing

**用途**：Rust 测试模式，包括单元测试、集成测试、异步测试、基于属性的测试、模拟和覆盖率

---

### Kotlin 开发技能

#### kotlin-patterns

**用途**：惯用 Kotlin 模式、最佳实践和构建健壮、高效、可维护 Kotlin 应用程序的约定

---

#### kotlin-testing

**用途**：使用 Kotest、MockK、coroutine 测试、基于属性的测试和 Kover 覆盖率的 Kotlin 测试模式

---

#### kotlin-coroutines-flows

**用途**：Android 和 KMP 的 Kotlin Coroutines 和 Flow 模式 — 结构化并发、Flow 操作符、StateFlow、共享流

---

#### kotlin-ktor-patterns

**用途**：Ktor 服务器模式，包括路由 DSL、插件、身份验证、Koin DI、kotlinx.serialization、WebSocket

---

#### kotlin-exposed-patterns

**用途**：JetBrains Exposed ORM 模式，包括 DSL 查询、DA O 模式、事务、HikariCP 连接池

---

#### compose-multiplatform-patterns

**用途**：Compose Multiplatform 和 Jetpack Compose 模式，用于 KMP 项目 — 状态管理、导航、主题、性能

---

#### android-clean-architecture

**用途**：Android 和 Kotlin Multiplatform 项目的整洁架构模式 — 模块结构、依赖规则

---

### C++ 开发技能

#### cpp-coding-standards

**用途**：基于 C++ Core Guidelines (isocpp.github.io) 的 C++ 编码标准

---

#### cpp-testing

**用途**：使用 GoogleTest/CTest 编写/更新/修复 C++ 测试、配置测试、诊断失败或不稳定测试

---

### Java 开发技能

#### java-coding-standards

**用途**：Spring Boot 服务的 Java 编码标准：命名、不可变性、Optional 使用、streams、异常、泛型

---

#### springboot-patterns

**用途**：Spring Boot 架构模式、REST API 设计、分层服务、数据访问、缓存、异步处理

---

#### springboot-tdd

**用途**：使用 JUnit 5、Mockito、MockMvc、Testcontainers 和 JaCoCo 的 Spring Boot 测试驱动开发

---

#### springboot-verification

**用途**：Spring Boot 项目的验证循环：构建、静态分析、带覆盖率的测试、安全扫描

---

### Django 开发技能

#### django-patterns

**用途**：Django 架构模式、使用 DRF 的 REST API 设计、ORM 最佳实践、缓存、信号、中间件

---

#### django-tdd

**用途**：使用 pytest-django、TDD 方法、factory_boy、模拟和覆盖率要求的 Django 测试驱动开发

---

#### django-verification

**用途**：Django 项目的验证循环：迁移、linting、带覆盖率的测试、安全扫描、部署验证

---

### Laravel 开发技能

#### laravel-patterns

**用途**：Laravel 架构模式、路由/控制器、Eloquent ORM、服务层、队列、事件、缓存

---

#### laravel-tdd

**用途**：使用 PHPUnit 和 Pest、factories、数据库测试、假对象和覆盖率要求的 Laravel 测试驱动开发

---

#### laravel-verification

**用途**：Laravel 项目的验证循环：环境检查、linting、静态分析、带覆盖率的测试、安全扫描

---

### 后端开发技能

#### backend-patterns

**用途**：Node.js、Python、Go、Rust 的后端架构模式、API 设计、数据库优化和服务端最佳实践

---

#### api-design

**用途**：REST API 设计模式，包括资源命名、状态代码、分页、过滤、错误响应、版本控制

---

#### docker-patterns

**用途**：Docker、Docker Compose 和容器化应用程序的生产就绪模式

最佳实践：
- 多阶段构建
- 非 root 用户
- .dockerignore 优化
- 健康检查
- 资源限制
- 卷管理
- 网络配置

---

#### deployment-patterns

**用途**：CI/CD、GitOps 和生产部署策略

---

### 数据库技能

#### postgres-patterns

**用途**：PostgreSQL 查询优化、架构设计、安全性、性能

---

#### database-migrations

**用途**：数据库迁移管理和最佳实践

---

### 安全技能

#### security-review

**用途**：安全漏洞检测和修复

**何时使用**：
- 主动用于处理用户输入、身份验证、API 端点或敏感数据的代码

---

#### security-scan

**用途**：安全扫描工具和方法

---

### AI 和机器学习技能

#### claude-api

**用途**：使用 Claude API 或 Anthropic SDK 构建应用程序

**何时触发**：
- 代码导入 `anthropic`/`@anthropic-ai/sdk`/`claude_agent_sdk`
- 用户请求使用 Claude API、Anthropic SDK 或 Agent SDK

---

#### ai-regression-testing

**用途**：AI 辅助开发的回归测试策略

---

#### ai-first-engineering

**用途**：AI 优先的工程模式和方法

---

#### cost-aware-llm-pipeline

**用途**：具有成本意识的 LLM 管道模式

---

### 测试技能

#### tdd-workflow

**用途**：测试驱动开发工作流模式

---

#### e2e-testing

**用途**：使用 Playwright 的 E2E 测试模式、页面对象模型、配置、CI/CD 集成

---

### 文档和内容技能

#### article-writing

**用途**：技术文章写作模式

---

#### content-engine

**用途**：内容生成和管理模式

---

### 其他技能

| 技能 | 用途 |
|-------|------|
| `coding-standards` | TypeScript、JavaScript、React 和 Node.js 开发的通用编码标准、最佳实践和模式 |
| `tdd-workflow` | 测试驱动开发工作流 |
| `verification-loop` | Claude Code 会话的综合验证系统 |
| `configure-ecc` | Everything Claude Code 的交互式安装程序 |
| `skill-creator` | 创建有效技能的指南 |
| `skill-stocktake` | 审计 Claude 技能和命令的质量 |
| `continuous-learning` | 从 Claude Code 会话自动提取可重用模式并保存为学习技能 |
| `continuous-learning-v2` | 基于本能的自动学习系统，通过钩子观察会话，创建具有置信度评分的原子本能 |
| `project-guidelines-example` | 基于真实生产应用程序的项目特定技能模板示例 |
| `agent-harness-construction` | 构建代理 harness |
| `agentic-engineering` | 代理工程模式 |
| `autonomous-loops` | 自主循环模式 |
| `blueprint` | 蓝图和模式 |
| `content-hash-cache-pattern` | 内容哈希缓存模式 |
| `search-first` | 搜索优先模式 |
| `deep-research` | 深度研究模式 |
| `exa-search` | Exa 搜索模式 |
| `strategic-compact` | 建议在逻辑间隔进行手动上下文压缩 |
| `plankton-code-quality` | 使用 Plankton 的写时代码质量执行 |
| `eval-harness` | Claude Code 会话的正式评估框架 |
| `mcp-server-patterns` | 使用 Node/TypeScript SDK 构建 MCP 服务器 — 工具、资源、提示、Zod 验证 |
| `iterative-retrieval` | 用于解决子代理上下文问题的渐进式上下文检索模式 |
| `regex-vs-llm-structured-text` | 正则表达式 vs LLM 结构化文本处理 |
| `prompt-optimizer` | 提示优化模式 |
| `deployment-patterns` | 部署模式 |
| `nanoclaw-repl` | NanoClaw v2 REPL |
| `dmux-workflows` | DMux 工作流 |
| `enterprise-agent-ops` | 企业代理操作 |
| `foundation-models-on-device` | 设备上的基础模型 |
| `jpa-patterns` | JPA 模式 |
| `webapp-testing` | 使用 Playwright 与本地 Web 应用程序交互和测试的工具包 |

---

## 详细使用场景

本节提供真实世界的使用示例，展示如何在实际项目中应用 ECC 的各种组件。

### 场景 1：新功能开发 - 用户认证系统

#### 背景
需要为 Web 应用添加完整的用户认证系统，包括：
- 邮箱注册和登录
- OAuth 社交媒体登录
- 密码重置
- 会话管理

#### 使用流程

```
# 步骤 1：使用 /plan 创建实施计划

用户: /plan 我需要为应用添加完整的用户认证系统，支持邮箱注册/登录、OAuth 登录（Google、GitHub）、密码重置和安全的会话管理

Agent (planner):
# 实施计划：用户认证系统

## 概述
实现完整的用户认证系统，包括邮箱认证、OAuth 社交媒体登录、密码重置和安全的会话管理。

## 需求
- 邮箱注册（验证邮箱）
- 邮箱/密码登录
- OAuth 登录（Google、GitHub）
- 密码重置流程
- 安全会话管理（JWT + Redis）
- 防止暴力破解（速率限制）
- 密码强度验证

## 架构变更
- 新表：users, user_sessions, password_reset_tokens, oauth_accounts
- 新 API 路由：/api/auth/* （注册、登录、登出、OAuth 回调）
- 新前端组件：LoginForm, RegisterForm, ForgotPassword, AuthProvider
- 新中间件：authMiddleware（验证 JWT）

## 实施步骤

### 阶段 1：数据库和后端核心（3 文件）
1. **创建用户表和迁移** (File: supabase/migrations/001_users.sql)
   - 操作：CREATE TABLE users with RLS policies
   - 原因：存储用户信息，启用行级安全
   - 依赖：无
   - 风险：低

2. **创建认证服务** (File: src/lib/auth/authService.ts)
   - 操作：实现 JWT 签发、密码哈希（bcrypt）、OAuth 逻辑
   - 原因：核心认证逻辑集中管理
   - 依赖：步骤 1
   - 风险：高 - 密码哈希和 JWT 签发必须正确

3. **创建认证 API 路由** (File: src/app/api/auth/login/route.ts 等)
   - 操作：实现 /api/auth/register, /login, /logout, /oauth/callback 端点
   - 原因：提供 REST API 端点供前端调用
   - 依赖：步骤 2
   - 风险：中 - 必须验证所有输入

### 阶段 2：前端组件（4 文件）
4. **创建 AuthProvider** (File: src/contexts/AuthContext.tsx)
   - 操作：实现 React Context 提供认证状态和方法
   - 原因：全局管理认证状态
   - 依赖：步骤 3
   - 风险：低

5. **创建登录表单** (File: src/components/auth/LoginForm.tsx)
   - 操作：实现邮箱登录和社交登录按钮
   - 原因：用户登录界面
   - 依赖：步骤 4
   - 风险：低

6. **创建注册表单** (File: src/components/auth/RegisterForm.tsx)
   - 操作：实现邮箱注册表单，包含验证
   - 原因：用户注册界面
   - 依赖：步骤 4
   - 风险：低

7. **创建密码重置表单** (File: src/components/auth/ForgotPassword.tsx)
   - 操作：实现"忘记密码"流程
   - 原因：密码恢复用户体验
   - 依赖：步骤 3
   - 风险：中 - 需要邮件服务

### 阶段 3：安全增强（2 文件）
8. **添加速率限制中间件** (File: src/middleware/rateLimit.ts)
   - 操作：实现 IP 级速率限制，防止暴力破解
   - 原因：安全最佳实践
   - 依赖：步骤 3
   - 风险：低

9. **添加认证中间件** (File: src/middleware/auth.ts)
   - 操作：验证 JWT，附加用户信息到请求
   - 原因：保护需要认证的路由
   - 依赖：步骤 2
   - 风险：中 - JWT 验证必须安全

## 测试策略
- 单元测试：密码哈希、JWT 签发、验证逻辑
- 集成测试：完整的登录/注册流程、OAuth 回调
- E2E 测试：用户可以完成完整的注册到登录流程

## 风险和缓解措施
- **风险**: OAuth 提供商更改 API
  - 缓解: 使用成熟的 OAuth 库（NextAuth.js）
- **风险**: 邮件服务不稳定
  - 缓解: 实现邮件队列和重试机制
- **风险**: 会话劫持
  - 缓解: 使用 HttpOnly cookies, 实现 CSRF 保护

## 成功标准
- [ ] 用户可以注册并收到验证邮件
- [ ] 用户可以使用邮箱密码登录
- [ ] 用户可以使用 Google/GitHub OAuth 登录
- [ ] 密码重置流程完整工作
- [ ] 会话安全（JWT, HttpOnly）
- [ ] 速率限制防止暴力破解
- [ ] 所有测试通过，覆盖率 80%+

**等待确认**：是否继续此计划？（yes/no/modify）

用户: yes
```

```
# 步骤 2：使用 /tdd 实现认证服务

用户: /tdd 实现 authService.ts，包含密码哈希、JWT 签发和验证函数

Agent (tdd-guide):
# TDD 会话：authService.ts

## 步骤 1：定义接口

```typescript
// src/lib/auth/authService.ts
export interface User {
  id: string
  email: string
  name: string
  verified: boolean
  createdAt: Date
}

export interface AuthTokens {
  accessToken: string
  refreshToken: string
}

export interface LoginCredentials {
  email: string
  password: string
}

export interface RegisterData {
  email: string
  password: string
  name: string
}

export async function hashPassword(password: string): Promise<string> {
  // TODO: 实现
  throw new Error('Not implemented')
}

export async function verifyPassword(password: string, hash: string): Promise<boolean> {
  throw new Error('Not implemented')
}

export async function generateAccessToken(user: User): Promise<string> {
  throw new Error('Not implemented')
}

export async function generateRefreshToken(user: User): Promise<string> {
  throw new Error('Not implemented')
}

export async function verifyAccessToken(token: string): Promise<User | null> {
  throw new Error('Not implemented')
}
```

## 步骤 2：编写测试

```typescript
// src/lib/auth/__tests__/authService.test.ts
import * as bcrypt from 'bcrypt'
import * as jwt from 'jsonwebtoken'
import {
  hashPassword,
  verifyPassword,
  generateAccessToken,
  verifyAccessToken
} from '../authService'

jest.mock('bcrypt')
jest.mock('jsonwebtoken')

describe('authService', () => {
  describe('hashPassword', () => {
    it('应该使用 bcrypt 哈希密码', async () => {
      const password = 'testPassword123'
      const hash = await hashPassword(password)

      expect(bcrypt.hash).toHaveBeenCalledWith(password, 10)
      expect(hash).toBeDefined()
      expect(hash).not.toBe(password)
    })

    it('应该抛出错误如果密码为空', async () => {
      await expect(hashPassword('')).rejects.toThrow('Password is required')
    })
  })

  describe('verifyPassword', () => {
    it('应该验证正确的密码', async () => {
      const password = 'testPassword123'
      const hash = 'hashedPassword123'

      ;(bcrypt.compare as jest.Mock).mockResolvedValueOnce(true)

      const result = await verifyPassword(password, hash)

      expect(bcrypt.compare).toHaveBeenCalledWith(password, hash)
      expect(result).toBe(true)
    })

    it('应该拒绝错误的密码', async () => {
      const password = 'wrongPassword'
      const hash = 'hashedPassword123'

      ;(bcrypt.compare as jest.Mock).mockResolvedValueOnce(false)

      const result = await verifyPassword(password, hash)

      expect(result).toBe(false)
    })
  })

  describe('generateAccessToken', () => {
    it('应该生成带有用户信息的 JWT', async () => {
      const user: User = {
        id: 'user123',
        email: 'test@example.com',
        name: 'Test User',
        verified: true,
        createdAt: new Date()
      }

      ;(jwt.sign as jest.Mock).mockReturnValueOnce('mockedToken')

      const token = await generateAccessToken(user)

      expect(jwt.sign).toHaveBeenCalledWith(
        { userId: user.id, email: user.email },
        expect.any(String),
        expect.any(Object)
      )
      expect(token).toBe('mockedToken')
    })
  })

  describe('verifyAccessToken', () => {
    it('应该验证有效的 token', async () => {
      const token = 'validToken123'
      const decodedUser = { userId: 'user123', email: 'test@example.com' }

      ;(jwt.verify as jest.Mock).mockReturnValueOnce(decodedUser)

      const user = await verifyAccessToken(token)

      expect(jwt.verify).toHaveBeenCalledWith(token, expect.any(String))
      expect(user).not.toBeNull()
    })

    it('应该拒绝无效的 token', async () => {
      const token = 'invalidToken'

      ;(jwt.verify as jest.Mock).mockImplementationOnce(() => {
        throw new Error('Invalid token')
      })

      const user = await verifyAccessToken(token)

      expect(user).toBeNull()
    })
  })
})
```

## 步骤 3：运行测试 - 验证失败

```bash
npm test src/lib/auth/__tests__/authService.test.ts

FAIL authService.test.ts
  ✕ hashPassword should hash password using bcrypt
    ReferenceError: hashPassword is not defined
  ...更多失败
```

## 步骤 4：实现最小代码

```typescript
// src/lib/auth/authService.ts
import * as bcrypt from 'bcrypt'
import * as jwt from 'jsonwebtoken'

const JWT_SECRET = process.env.JWT_SECRET || 'dev-secret'
const JWT_EXPIRY = '15m'
const REFRESH_TOKEN_EXPIRY = '7d'

export async function hashPassword(password: string): Promise<string> {
  if (!password) {
    throw new Error('Password is required')
  }
  return bcrypt.hash(password, 10)
}

export async function verifyPassword(password: string, hash: string): Promise<boolean> {
  return bcrypt.compare(password, hash)
}

export async function generateAccessToken(user: User): Promise<string> {
  const payload = {
    userId: user.id,
    email: user.email
  }

  return jwt.sign(payload, JWT_SECRET, {
    expiresIn: JWT_EXPIRY,
    issuer: 'your-app'
  })
}

export async function generateRefreshToken(user: User): Promise<string> {
  const payload = {
    userId: user.id,
    type: 'refresh'
  }

  return jwt.sign(payload, JWT_SECRET, {
    expiresIn: REFRESH_TOKEN_EXPIRY,
    issuer: 'your-app'
  })
}

export async function verifyAccessToken(token: string): Promise<User | null> {
  try {
    const decoded = jwt.verify(token, JWT_SECRET, {
      issuer: 'your-app'
    }) as any

    if (!decoded.userId || !decoded.email) {
      return null
    }

    // 在实际应用中，这里应该从数据库获取完整用户信息
    return {
      id: decoded.userId,
      email: decoded.email,
      name: '', // 从数据库获取
      verified: true,
      createdAt: new Date()
    }
  } catch (error) {
    return null
  }
}
```

## 步骤 5-8：运行测试、重构、验证覆盖率...

✅ TDD 会话完成！
```

```
# 步骤 3：使用 /code-review 审查代码

用户: /code-review

Agent (code-reviewer):
## Code Review Results

[CRITICAL] Hardcoded JWT secret in source code
File: src/lib/auth/authService.ts:3
Issue: JWT_SECRET uses fallback value 'dev-secret' which could be used in production
Fix: Remove fallback, throw error if JWT_SECRET is not set in production

  const JWT_SECRET = process.env.JWT_SECRET!;  // Throw in production if missing

[HIGH] Missing token type validation
File: src/lib/auth/authService.ts:45
Issue: verifyAccessToken doesn't validate token structure before returning User
Fix: Add proper type checking and validation

[MEDIUM] Missing JSDoc for public API
File: src/lib/auth/authService.ts:8
Issue: Exported functions lack documentation
Fix: Add JSDoc comments explaining parameters and return values

## Review Summary

| Severity | Count | Status |
|----------|-------|--------|
| CRITICAL | 1     | block  |
| HIGH     | 1     | warn   |
| MEDIUM   | 1     | info   |

Verdict: BLOCK — 1 CRITICAL issue must be fixed before merge.

[建议操作：使用 /build-fix 修复这些问题]
```

```
# 步骤 4：使用 /e2e 测试完整登录流程

用户: /e2e 测试用户注册到登录的完整流程

Agent (e2e-runner):
# E2E 测试：用户认证流程

## 生成的测试

```typescript
// tests/e2e/auth/complete-auth-flow.spec.ts
import { test, expect } from '@playwright/test'
import { LoginPage } from '../../pages/LoginPage'
import { RegisterPage } from '../../pages/RegisterPage'

test.describe('Complete Authentication Flow', () => {
  test('用户可以注册并收到验证邮件', async ({ page }) => {
    const registerPage = new RegisterPage(page)
    await registerPage.goto()

    // 填写注册表单
    await registerPage.fillName('Test User')
    await registerPage.fillEmail(`test${Date.now()}@example.com`)
    await registerPage.fillPassword('SecurePassword123!')
    await registerPage.fillConfirmPassword('SecurePassword123!')

    // 提交表单
    await registerPage.submit()

    // 验证成功消息
    await expect(page.locator('[data-testid="success-message"]')).toBeVisible()
    await expect(page.locator('[data-testid="success-message"]')).toContainText('验证邮件已发送')
  })

  test('用户可以使用注册的凭据登录', async ({ page }) => {
    const loginPage = new LoginPage(page)
    await loginPage.goto()

    // 使用测试凭据登录
    await loginPage.fillEmail('test@example.com')
    await loginPage.fillPassword('SecurePassword123!')
    await loginPage.submit()

    // 验证重定向到仪表板
    await expect(page).toHaveURL(/\/dashboard/)
    await expect(page.locator('h1')).toContainText('欢迎回来')
  })

  test('用户可以使用 Google OAuth 登录', async ({ page }) => {
    const loginPage = new LoginPage(page)
    await loginPage.goto()

    // 点击 Google 登录按钮
    await loginPage.clickGoogleLogin()

    // 在实际测试中，这里会重定向到 Google OAuth 页面
    // 对于测试，我们模拟成功的回调
    await page.goto('/api/auth/callback/google?code=test_code&state=test_state')

    // 验证登录成功
    await expect(page).toHaveURL(/\/dashboard/)
  })
})
```

## 运行测试

```bash
npx playwright test tests/e2e/auth/

Running 3 tests using 3 workers

  ✓  [chromium] › 用户可以注册并收到验证邮件 (3.2s)
  ✓  [chromium] › 用户可以使用注册的凭据登录 (2.1s)
  ✓  [chromium] › 用户可以使用 Google OAuth 登录 (4.5s)

  3 passed (9.8s)
```

✅ E2E 测试全部通过！
```

```
# 步骤 5：提交代码

git add .
git commit -m "feat: implement complete user authentication system

- Email registration and login with password
- OAuth social login (Google, GitHub)
- Password reset flow
- Secure session management with JWT
- Rate limiting for brute force protection
- Password strength validation

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

✅ 认证系统开发完成！
```

---

### 场景 2：Bug 修复 - 修复内存泄漏问题

#### 背景
生产环境报告 Node.js 应用在运行 24 小时后内存使用持续增长，最终导致 OOM 错误。

#### 使用流程

```
# 步骤 1：分析问题

用户: 我需要调查生产环境的内存泄漏问题。应用在 24 小时后内存使用持续增长，直到 OOM。

用户报告的症状：
- 内存使用从 200MB 增长到 2GB+
- CPU 使用正常
- 无明显请求增加
- 最终应用崩溃并重启

# 步骤 2：使用 /tdd 重现和修复

用户: /tdd 创建一个测试来重现潜在的内存泄漏问题，然后修复它

Agent (tdd-guide):
# TDD 会话：内存泄漏修复

## 步骤 1：分析可疑代码

经过代码审查，发现可疑代码：

```typescript
// src/services/dataProcessor.ts
class DataProcessor {
  private cache: Map<string, any> = new Map()

  processData(data: any[]) {
    data.forEach(item => {
      const key = item.id
      // 问题：缓存从不清理！
      this.cache.set(key, this.processItem(item))
    })
  }

  private processItem(item: any): any {
    return {
      ...item,
      processed: true,
      timestamp: Date.now()
    }
  }
}

// src/app.ts
const processor = new DataProcessor()

// 问题：全局实例永不释放
app.post('/process', (req, res) => {
  processor.processData(req.body.data)
  res.json({ success: true })
})
```

## 步骤 2：编写测试来检测内存泄漏

```typescript
// src/services/__tests__/dataProcessor.memory.test.ts
import { DataProcessor } from '../dataProcessor'

describe('DataProcessor Memory Leak Test', () => {
  it('应该正确管理缓存大小', () => {
    const processor = new DataProcessor()

    // 模拟大量数据处理
    for (let i = 0; i < 10000; i++) {
      processor.processData([{ id: `item-${i}`, value: i }])
    }

    // 验证：缓存应该有大小限制
    const cacheSize = (processor as any).cache.size
    expect(cacheSize).toBeLessThanOrEqual(1000) // 应该有上限
  })

  it('应该清理旧缓存项', () => {
    const processor = new DataProcessor()

    // 添加项目
    processor.processData([{ id: 'item-1', value: 1 }])

    // 等待超过缓存过期时间
    await new Promise(resolve => setTimeout(resolve, 1100))

    // 添加更多项目，应该触发清理
    processor.processData([{ id: 'item-2', value: 2 }])

    // 验证：旧项目应该被清理
    const hasOldItem = (processor as any).cache.has('item-1')
    expect(hasOldItem).toBe(false)
  })
})
```

## 步骤 3：运行测试 - 验证失败

```bash
npm test src/services/__tests__/dataProcessor.memory.test.ts

FAIL dataProcessor.memory.test.ts
  ✕ 应该正确管理缓存大小
    Error: Expected 10000 to be less than or equal to 1000
    Received: 10000

1 test failed
```

✅ 测试检测到内存泄漏！

## 步骤 4：实现修复

```typescript
// src/services/dataProcessor.ts
class DataProcessor {
  private cache: Map<string, { data: any, timestamp: number }> = new Map()
  private readonly MAX_CACHE_SIZE = 1000
  private readonly CACHE_TTL = 60000 // 60 秒
  private cleanupInterval?: NodeJS.Timeout

  constructor() {
    // 定期清理过期项
    this.cleanupInterval = setInterval(() => {
      this.cleanup()
    }, 30000) // 每 30 秒清理一次
  }

  processData(data: any[]) {
    data.forEach(item => {
      const key = item.id
      this.cache.set(key, {
        data: this.processItem(item),
        timestamp: Date.now()
      })

      // 如果缓存过大，立即清理
      if (this.cache.size > this)MAX_CACHE_SIZE) {
        this.cleanup()
      }
    })
  }

  private processItem(item: any): any {
    return {
      ...item,
      processed: true,
      timestamp: Date.now()
    }
  }

  private cleanup() {
    const now = Date.now()
    const keysToDelete: string[] = []

    // 删除过期项
    for (const [key, value] of this.cache.entries()) {
      if (now - value.timestamp > this.CACHE_TTL) {
        keysToDelete.push(key)
      }
    }

    keysToDelete.forEach(key => this.cache.delete(key))

    // LRU 策略：如果仍然太大，删除最旧的项
    if (this.cache.size > this.MAX_CACHE_SIZE) {
      const entries = Array.from(this.cache.entries())
        .sort((a, b) => a[1].timestamp - b[1].timestamp)

      const toDelete = entries.slice(0, this.cache.size - this.MAX_CACHE_SIZE)
      toDelete.forEach(([key]) => this.cache.delete(key))
    }
  }

  destroy() {
    if (this.cleanupInterval) {
      clearInterval(this.cleanupInterval)
    }
    this.cache.clear()
  }
}

// src/app.ts - 改为局部实例
app.post('/process', (req, res) => {
  const processor = new DataProcessor()
  try {
    processor.processData(req.body.data)
    res.json({ success: true })
  } finally {
    processor.destroy() // 确保清理
  }
})
```

## 步骤 5：运行测试 - 验证通过

```bash
npm test src/services/__tests__/dataProcessor.memory.test.ts

PASS dataProcessor.memory.test.ts
  ✓ 应该正确管理缓存大小
  ✓ 应该清理旧缓存项

2 tests passed (5.2s)
```

✅ 测试通过！内存泄漏已修复。
```

```
# 步骤 3：使用 /code-review 验证修复

用户: /code-review

Agent (code-reviewer):
## Code Review Results

[INFO] Good cleanup pattern implementation File: src/services/dataProcessor.ts:45
Issue: Proper use of setInterval with cleanup in destructor

[MEDIUM] Consider using WeakMap for automatic cleanup
File: src/services/dataProcessor.ts:7
Issue: WeakMap would automatically remove entries when objects are GC'd
Fix: Consider if appropriate for your use case

## Review Summary

| Severity | Count | Status |
|----------|-------|--------|
| CRITICAL | 0     | pass   |
| HIGH     | 0     | pass   |
| MEDIUM   | 1     | info   |

Verdict: APPROVED — No blocking issues.

✅ 修复代码审查通过！
```

```
# 步骤 4：提交修复

git commit -m "fix: resolve memory leak in DataProcessor

- Implement LRU cache with TTL
- Add periodic cleanup interval
- Ensure proper resource cleanup on destroy
- Use local processor instances instead of global

Fixes: #1234

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

✅ 内存泄漏修复完成！
```

---

### 场景 3：重构 - 提取可重用的 API 客户端

#### 背景
项目中多个组件都有重复的 API 调用代码，需要重构为一个统一的 API 客户端。

#### 使用流程

```
# 步骤 1：使用 /plan 创建重构计划

用户: /plan 重构项目中的 API 调用，提取为统一的 API 客户端

Agent (planner):
# 重构计划：统一 API 客户端

## 概述
将分散在多个组件中的 API 调用代码重构为一个集中管理的 API 客户端，提供类型安全、错误处理和请求拦截。

## 需求
- 统一的 API 基础 URL 配置
- 类型安全的请求/响应
- 统一错误处理
- 请求/响应拦截器（用于 auth、日志）
- 自动重试机制
- 取消请求支持

## 实施步骤

### 阶段 1：创建 API 客户端（2 文件）
1. **创建 API 客户端核心** (File: src/lib/api/client.ts)
   - 操作：实现基于 fetch 的客户端，支持类型、错误处理、拦截器
   - 原因：提供统一的 API 调用接口

2. **创建 API 类型定义** (File: src/lib/api/types.ts)
   - 操作：定义所有 API 端点的请求和响应类型
   - 原因：类型安全的 API 调用

### 阶段 2：创建 API 方法（3 文件）
3. **创建用户 API** (File: src/lib/api/users.ts)
   - 操作：封装用户相关 API 调用
   - 原因：提供类型化的用户 API 方法

4. **创建市场 API** (File: src/lib/api/markets.ts)
   - 操作：封装市场相关 API 调用
   - 原因：提供类型化的市场 API 方法

5. **创建交易 API** (File: src/lib/api/trades.ts)
   - 操作：封装交易相关 API 调用
   - 原因：提供类型化的交易 API 方法

### 阶段 3：更新组件使用（多个文件）
6. **重构组件以使用新客户端**
   - 操作：更新所有组件移除直接的 fetch 调用
   - 原因：使用统一的 API 客户端

### 阶段 4：添加测试
7. **编写 API 客户端测试**
   - 操作：单元测试和集成测试
   - 原因：确保重构后功能正常

## 测试策略
- 单元测试：请求构建、错误处理、拦截器
- 集成测试：实际 API 调用（使用 mock）
- E2E 测试：组件仍然正常工作

## 风险和缓解措施
- **风险**: 重构可能破坏现有功能
  - 缓解: 保持旧代码直到新代码充分测试
- **风险**: 类型定义不完整
  - 缓解: 逐步迁移，保持灵活性

## 成功标准
- [ ] 所有 API 调用使用新客户端
- [ ] 类型安全的请求/响应
- [ ] 统一错误处理
- [ ] 请求拦截器工作（auth 添加）
- [ ] 自动重试机制工作
- [ ] 所有现有测试通过
- [ ] 新测试覆盖率 80%+

**等待确认**：是否继续此计划？（yes/no/modify）

用户: yes
```

```
# 步骤 2：使用 /tdd 实现和测试

用户: /tdd 创建 API 客户端和测试

Agent (tdd-guide):
# TDD 会话：API 客户端

[实现过程...]

✅ API 客户端实现完成，测试通过！
```

```
# 步骤 3：使用 /code-review 审查重构

用户: /code-review

[审查过程...]

✅ 重构代码审查通过！
```

```
# 步骤 4：提交重构

git commit -m "refactor: extract unified API client

- Create centralized API client with type safety
- Implement unified error handling
- Add request/response interceptors
- Auto-retry mechanism for failed requests
- Refactor all components to use new client

Breaking changes: Components must import from new API client

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

✅ 重构完成！
```

---

### 场景 4：多语言项目 - 为 Go 后端添加新 API 端点

#### 背景
Go 后端项目需要添加新的 REST API 端点来处理数据分析请求。

#### 使用流程

```
# 步骤 1：使用 /plan 创建计划

用户: /plan 为 Go 后端添加数据分析 API 端点，包括数据验证、查询优化和错误处理

Agent (planner):
# 实施计划：Go 数据分析 API

## 概述
添加新的 REST API 端点用于数据分析，包括输入验证、数据库查询优化和全面的错误处理。

## 实施步骤

### 阶段 1：数据模型和验证（2 文件）
1. **创建请求结构体** (File: internal/api/data_analysis.go)
2. **添加验证逻辑** (File: internal/api/data_analysis.go)

### 阶段 2：数据库查询（1 文件）
3. **优化数据库查询** (File: internal/db/queries.go)

### 阶段 3：API 处理器（1 文件）
4. **实现 HTTP 处理器** (File: internal/api/handlers.go)

### 阶段 4：路由配置（1 文件）
5. **注册路由** (File: cmd/server/routes.go)

## 测试策略
- 单元测试：验证逻辑、错误处理
- 集成测试：完整 API 调用
- 基准测试：查询性能

[计划确认...]
```

```
# 步骤 2：使用 /go-test 实现

用户: /go-test 实现数据分析 API

Agent (tdd-guide):
# TDD 会话：Go API 实现

[按照 Go TDD 模式实现...]

## 生成的测试

```go
// internal/api/data_analysis_test.go
package api_test

import (
	"net/http"
	"net/http/httptest"
	"strings"
	"testing"

	"github.com/stretchr/testify/assert"
	"github.com/stretchr/testify/require"
)

func TestDataAnalysisHandler_ValidRequest(t *testing.T) {
	reqBody := `{
		"start_date": "2024-01-01",
		"end_date": "2024-01-31",
		"metrics": ["revenue", "users", "orders"]
	}`

	req := httptest.NewRequest("POST", "/api/analytics", strings.NewReader(reqBody))
	req.Header.Set("Content-Type", "application/json")

	w := httptest.NewRecorder()
	DataAnalysisHandler(w, req)

	assert.Equal(t, http.StatusOK, w.Code())

	var response map[string]interface{}
	err := json.Unmarshal(w.Body.Bytes(), &response)
	require.NoError(t, err)
	assert.NotNil(t, response["data"])
}

func TestDataAnalysisHandler_InvalidJSON(t *testing.T) {
	reqBody := `{invalid json}`

	req := httptest.NewRequest("POST", "/api/analytics", strings.NewReader(reqBody))
	w := httptest.NewRecorder()
	DataAnalysisHandler(w, req)

	assert.Equal(t, http.StatusBadRequest, w.Code())
}

func TestDataAnalysisHandler_MissingRequiredFields(t *testing.T) {
	reqBody := `{"start_date": "2024-01-01"}`

	req := httptest.NewRequest("POST", "/api/analytics", strings.NewReader(reqBody))
	w := httptest.NewRecorder()
	DataAnalysisHandler(w, req)

	assert.Equal(t, http.StatusBadRequest, w.Code())

	var response map[string]interface{}
	json.Unmarshal(w.Body.Bytes(), &response)
	assert.Contains(t, response["error"], "required")
}

func TestDataAnalysisHandler_InvalidDateRange(t *testing.T) {
	reqBody := `{
		"start_date": "2024-01-01",
		"end_date": "2023-01-01"
	}`

	req := httptest.NewRequest("POST", "/api/analytics", strings.NewReader(reqBody))
	w := httptest.NewRecorder()
	DataAnalysisHandler(w, req)

	assert.Equal(t, http.StatusBadRequest, w.Code())
}
```

## 实现代码

```go
// internal/api/data_analysis.go
package api

import (
	"encoding/json"
	"errors"
	"net/http"
"
	"github.com/go-playground/validator/v10"
)

type AnalysisRequest struct {
	StartDate string   `json:"start_date" validate:"required"`
	EndDate   string   `json:"end_date" validate:"required"`
	Metrics   []string `json:"metrics" validate:"required,min=1"`
}

type AnalysisResponse struct {
	Data   interface{} `json:"data"`
	Meta   struct {
		TotalRecords int `json:"total_records"`
		QueryTime   int `json:"query_time_ms"`
	} `json:"meta"`
}

func DataAnalysisHandler(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPost {
		http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
		return
	}

	var req AnalysisRequest
	if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
		http.Error(w, "Invalid JSON", http.StatusBadRequest)
		return
	}

	validate := validator.New()
	if err := validate.Struct(&req); err != nil {
		http.Error(w, err.Error(), http.StatusBadRequest)
		return
	}

	// 验证日期范围
	startTime, err := time.Parse("2006-01-02", req.StartDate)
	if err != nil {
		http.Error(w, "Invalid start date format", http.StatusBadRequest)
		return
	}

	endTime, err := time.Parse("2006-01-02", req.EndDate)
	if err != nil {
		http.Error(w, "Invalid end date format", http.StatusBadRequest)
		return
	}

	if endTime.Before(startTime) {
		http.Error(w, "End date must be after start date", http.StatusBadRequest)
		return
	}

	// 调用数据库查询
	data, err := db.QueryAnalytics(startTime, endTime, req.Metrics)
	if err != nil {
		http.Error(w, "Internal server error", http.StatusInternalServerError)
		return
	}

	response := AnalysisResponse{
		Data: data,
		Meta: struct {
			TotalRecords int `json:"total_records"`
			QueryTime   int `json:"query_time_ms"`
		}{
			TotalRecords: len(data),
			QueryTime:   100, // 实际应该测量
		},
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(response)
}
```

✅ Go API 实现，测试通过！
```

```
# 步骤 3：使用 /go-review 审查代码

用户: /go-review

Agent (go-reviewer):
# Go Code Review

[审查检查...]

## Review Summary

| Severity | Count | Status |
|----------|-------|--------|
| CRITICAL | 0     | pass   |
| HIGH     | 0     | pass   |
| MEDIUM   | 1     | info   |

[建议] 考虑添加速率限制中间件防止 API 滥用
[建议] 添加请求 ID 用于日志追踪

✅ Go 代码审查通过！
```

```
# 步骤 4：提交代码

git commit -m "feat: add data analysis API endpoint

- Implement POST /api/analytics endpoint
- Add comprehensive input validation
- Optimize database queries
- Add structured error handling
- Include test coverage

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

✅ Go API 端点开发完成！
```

---

### 场景对比：不同开发任务的最佳工具选择

| 开发任务 | 推荐命令/代理 | 原因 |
|---------|-------------|------|
| **新功能开发** | `/plan` → `/tdd` → `/code-review` | 确保正确实现和代码质量 |
| **Bug 修复** | `/tdd` → `/code-review` | 先写测试重现 bug，确保修复正确 |
| **代码重构** | `/plan` → `/tdd` → `/code-review` | 计划重构步骤，保持测试通过 |
| **API 开发** | `/plan` → `/tdd` → `/api-design` skill | 设计良好的 API 并测试 |
| **前端组件** | `/tdd` → `frontend-patterns` skill | TDD + React 最佳实践 |
| **数据库变更** | `/plan` → `/tdd` → `postgres-patterns` skill | 安全的迁移和数据管理 |
| **性能优化** | `/plan` → `/tdd` → 测试性能 | 优化前先建立基准和测试 |
| **安全审查** | `/code-review` 或 `security-reviewer` agent | 主动发现安全漏洞 |
| **E2E 测试** | `/e2e` | 验证关键用户流程 |
| **构建错误** | `/build-fix` 或语言特定构建命令 | 快速修复编译错误 |
| **学习模式** | `/learn` 或 `/skill-create` | 保存可重用知识 |

---

### 团队协作场景

#### 场景：团队代码审查流程

```markdown
# 团队代码审查最佳实践

## 使用 ECC 进行代码审查的流程

### 1. 开发者完成实现

开发者使用 ECC 工作流程：
```
开发者:
# 使用 TDD 实现功能
/tdd 实现用户资料更新功能

[实现过程...]

# 本地代码审查
/code-review

✅ 本地审查通过，准备提交
```

### 2. 提交代码和创建 PR

```bash
git add .
git commit -m "feat: add user profile update"
git push origin feature/user-profile-update
```

在 GitHub/GitLab 创建 Pull Request。

### 3. CI 自动运行 ECC 命令

在 `.github/workflows/pr-check.yml` 中配置：

```yaml
name: PR Checks

on:
  pull_request:
    branches: [main]

jobs:
  ecc-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Run Code Review
        run: |
          # 使用 Claude Code 运行代码审查
          claude code-review

      - name: Run Tests
        run: npm test

      - name: Check Coverage
        run: npm run test:coverage
```

### 4. 审查者使用 ECC 进行审查

审查者可以使用 ECC 命令进行深入分析：

```markdown
审阅者:
# 审查 PR 中的代码

查看关键文件：
1. 提交信息和变更摘要
2. 核心实现代码
3. 相关的测试文件

# 使用代码审查命令
/code-review

[接收详细的审查报告]

# 可选：运行特定语言的审查
/python-review  # 如果是 Python 代码
/go-review      # 如果是 Go 代码
/rust-review    # 如果是 Rust 代码
```

### 5. 审查反馈模板

ECC 生成的审查报告可以直接用于 PR 评论：

```markdown
## Code Review Summary

### Security Issues (CRITICAL)
- [ ] 硬编码凭证已移除
- [ ] SQL 注入已修复

### Code Quality (HIGH)
- [ ] 函数大小已优化（<50 行）
- [ ] 错误处理已完善

### Best Practices (MEDIUM)
- [ ] 添加了必要的 JSDoc
- [ ] 提取了重复代码

## Overall Assessment
Status: ⚠️ REQUIRES CHANGES (1 CRITICAL issue)

建议修复 CRITICAL 问题后再合并。
```

### 6. 修复和重新审查

开发者根据反馈修复问题：

```markdown
开发者:
# 使用 build-fix 修复问题
/build-fix

[自动修复构建错误...]

# 重新运行代码审查
/code-review

✅ 所有问题已解决
```

### 7. 批准和合并

所有问题解决后，审阅者批准 PR 并合并。

---

### CI/CD 集成示例

#### GitHub Actions 工作流

```yaml
# .github/workflows/quality-gate.yml
name: Quality Gate

on:
  pull_request:
  pull_request_target:
    types: [closed, merged]

jobs:
  test:
    runs-on: ubuntu-lares
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test -- --coverage

      - name: Check coverage threshold
        run: |
          COVERAGE=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
          if (( $(echo "$COVERAGE < 0.80" | bc -l) )); then
            echo "Coverage $COVERAGE is below 80% threshold"
            exit 1
          fi

  e2e-tests:
    runs-on: ubuntu-lates
    steps:
      - uses: actions/checkout@v3

      - name: Install Playwright
        run: npx playwright install --with-deps

      - name: Run E2E tests
        run: npx playwright test

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: playwright-report
          path: playwright-report/

  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Run security review
        run: |
          # 使用 security-reviewer agent
          claude security-review

      - name: Run dependency audit
        run: npm audit --audit-level high

  code-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Run code review
        run: |
          # 使用 code-reviewer agent
          claude code-review
```

---

### 高级场景

#### 场景：多语言项目开发

**背景**：一个使用 TypeScript 前端、Go 后端和 Python 数据处理服务的项目。

**策略**：为每个语言使用相应的 ECC 工具。

```markdown
# 多语言项目开发策略

## 前端 (TypeScript)

### 开发流程
1. 使用 `/plan` 规划前端功能
2. 使用 `/tdd` 和 `frontend-patterns` skill 实现组件
3. 使用 `/e2e` 测试用户流程
4. 使用 `/code-review` 审查代码

### 示例：React 组件开发

用户: /tdd 创建一个市场列表组件，使用 TypeScript

[tdd-guide 模式工作流...]

✅ 组件实现完成

---

## 后端 (Go)

### 开发流程
1. 使用 `/plan` 规划 API 端点
2. 使用 `/go-test` 实现处理程序
3. 使用 `/go-review` 审查代码
4. 使用 `/go-build` 确保构建成功

### 示例：REST API 端点

用户: /go-test 实现用户认证 API 端点

[tdd-guide 使用 Go 模式...]

✅ API 端点实现完成

---

## 数据处理 (Python)

### 开发流程
1. 使用 `/plan` 规划数据处理逻辑
2. 使用 `/tdd` 和 `python-patterns` skill 实现
3. 使用 `/python-review` 审查代码

### 示例：数据分析管道

用户: /tdd 实现市场数据清洗和分析功能

[tdd-guide 使用 Python 模式...]

✅ 数据处理实现完成

---

## 集成测试

### 使用 E2E 测试验证跨系统流程

用户: /e2e 测试完整的数据流：
前端 → Go API → Python 处理 → 数据库

[e2e-runner 生成跨系统测试...]

✅ 集成测试通过
```

---

#### 场景：技术债务清理

**背景**：需要系统性地清理项目中的技术债务。

**策略**：分阶段清理，每个阶段使用适当的 ECC 工具。

```markdown
# 技术债务清理计划

## 阶段 1：代码质量审查

### 使用 code-reviewer 识别问题

用户: /code-review 审查整个代码库

Agent (code-reviewer):
## Code Review Results

发现以下技术债务：

### 安全问题 (CRITICAL)
- 3 处硬编码凭证
- 2 处 SQL 注入风险

### 代码质量 (HIGH)
- 15 个函数超过 50 行
- 8 个文件超过 800 行
- 25 处缺少错误处理
- 12 处 console.log 语句

### 最佳实践 (MEDIUM)
- 30 处缺少 JSDoc
- 20 处重复代码

## 清理计划

按优先级分组：
1. **CRITICAL** - 立即修复
2. **HIGH** - 本周修复
3. **MEDIUM** - 下个迭代修复

---

## 阶段 2：修复 CRITICAL 问题

### 使用 security-reviewer

用户: 修复所有硬编码凭证和 SQL 注入风险

[使用 security-reviewer 逐个修复...]

✅ CRITICAL 问题已修复

---

## 阶段 3：重构大函数

### 使用 refactor-cleaner

用户: /refactor-clean 清理大函数和文件

Agent (refactor-cleaner):
## Refactoring Plan

发现 15 个大函数：
- `src/utils/dataProcessor.ts:processData()` (120 行)
- `src/components/DashboardChart.tsx:renderChart()` (85 行)
- ...

## 重构策略

### 函数拆分
1. 识别函数职责
2. 提取辅助函数
3. 使用组合模式

### 示例：重构 processData

**原始代码（120 行）**：
```typescript
function processData(data: any[]) {
  // 120 行的复杂逻辑
}
```

**重构后**：
```typescript
function processData(data: any[]) {
  const validated = validateData(data)
  const transformed = transformData(validated)
  const aggregated = aggregateData(transformed)
  return formatOutput(aggregated)
}

function validateData(data: any[]) { /* ... */ }
function transformData(data: any[]) { /* ... */ }
function aggregateData(data: any[]) { /* ... */ }
function formatOutput(data: any[]) { /* ... */ }
```

[重构过程...]

✅ 大函数已重构

---

## 阶段 4：清理死代码

用户: 使用 knip 工具识别未使用的代码

```bash
npm run lint:unused
```

移除识别出的未使用代码：

```bash
# 使用 refactor-cleaner
/refactor-clean 移除未使用的导入和函数
```

---

## 阶段 5：添加文档

使用自动文档生成 + 手动补充：

```bash
# 生成 API 文档
npm run docs:api

# 使用 doc-updater
/update-docs 更新代码图
```

---

## 最终验证

```bash
# 运行所有测试
npm test

# 检查覆盖率
npm run test:coverage

# 代码审查
/code-review

✅ 所有清理完成，技术债务显著减少
```

---

## 工作流程指南

### 典型功能开发流程

```
1. /plan
   ├─ 创建详细的实施计划
   ├─ 识别依赖和风险
   └─ 等待用户确认

2. /tdd
   ├─ 编写失败的测试（RED）
   ├─ 实现最小代码（GREEN）
   ├─ 重构代码（REFACTOR）
   └─ 验证 80%+ 覆盖率

3. /build-fix
   ├─ 运行构建
   ├─ 修复构建错误
   └─ 验证构建通过

4. /e2e
   ├─ 生成端到端测试
   ├─ 运行测试
   └─ 验证关键用户流程

5. /code-review
   ├─ 审查代码质量
   ├─ 识别安全问题
   └─ 修复 CRITICAL 和 HIGH 问题

6. git commit
   └─ 提交更改
```

### Bug 修复流程

```
1. 分析 Bug
   └─ 理解根本原因

2. /tdd
   ├─ 编写重现 bug 的测试
   ├─ 修复问题
   └─ 验证测试通过

3. /code-review
   └─ 审查修复

4. git commit
   └─ 提交修复
```

### 重构流程

```
1. /plan
   └─ 创建重构计划

2. /tdd
   ├─ 确保现有测试覆盖
   ├─ 进行重构
   └─ 验证所有测试通过

3. /code-review
   └─ 审查重构

4. git commit
   └─ 提交重构
```

---

## 快速决策指南

### 根据任务类型选择合适的工具

#### "我需要实现一个新功能..."

**决策树**：

```
需要实现新功能
├─ 涉及多个文件/组件吗？
│  └─ 是 → /plan （创建详细计划）
│       ↓
│     /tdd （TDD 实施）
│       ↓
│     /e2e （关键流程测试）
│
└─ 否 → /tdd （直接开始实现）
```

**示例**：
```
用户：我需要实现用户订阅功能

Claude:
涉及多个文件（API 端点、数据库、前端组件）？
- 是，让我先创建计划。

[调用 /plan]
创建实现计划...

计划确认后：
[调用 /tdd]
使用 TDD 实现...

[调用 /e2e]
测试关键流程...

[调用 /code-review]
审查代码质量
```

---

#### "我发现了一个 bug..."

**决策树**：

```
发现 bug
├─ 知道根本原因吗？
│  ├─ 是 → /tdd （写测试重现 bug）
│  │         ↓
│  │     修复实现
│  │         ↓
│  │     /code-review （验证修复）
│  │
│  └─ 否 → /build-fix （如果是构建错误）
│          或
│          /code-review （审阅代码找问题）
```

**示例**：
```
用户：登录功能不工作，总是返回 401

Claude:
让我先审阅代码找出问题...

[调用 /code-review]
发现可能的问题：密码比较逻辑错误

现在写测试来重现：
[调用 /tdd]
创建失败的登录测试...

实现修复...

验证修复...
[调用 /code-review]
所有问题已解决 ✅
```

---

#### "我需要重构代码..."

**决策树**：

```
需要重构
├─ 影响范围大吗（>5 个文件）？
│  └─ 是 → /plan （创建重构计划）
│
├─ 需要保持现有行为吗？
│  └─ 是 → /tdd （确保测试覆盖）
│
└─ 提取可重用模式？
    └─ /learn （保存模式为技能）
```

**示例**：
```
用户：多个组件都有重复的 API 调用代码，需要重构

Claude:
重构影响多个组件，让我先创建计划...

[调用 /plan]
重构计划：提取统一 API 客户端

开始重构：
[调用 /tdd]
确保现有测试通过...

实现重构...

验证：
[调用 /e2e]
测试用户流程不受影响...

[调用 /code-review]
审查重构质量

发现的模式有用，保存为技能：
[调用 /learn]
保存 API 客户端模式
```

---

#### "构建失败了..."

**决策树**：

```
构建失败
├─ 知道具体错误吗？
│  └─ 是 → /build-fix （自动修复）
│
└─ 否 → /build-fix （检测并修复）
```

**示例**：
```
用户：npm run build 失败了

Claude:
让我自动修复构建错误...

[调用 /build-fix]
检测构建系统：npm
运行构建：npm run build

发现 15 个 TypeScript 错误：

1. ✅ 修复：src/api/client.ts:42 - 缺少类型导入
   重新运行构建...
   12 个错误

2. ✅ 修复：src/components/Chart.tsx:15 - 未定义变量
   重新运行构建...
   10 个错误

[继续...]

15 个错误全部修复 ✅

构建成功！
```

---

#### "我需要测试我的应用..."

**决策树**：

```
需要测试应用
├─ 测试特定功能？
│  └─ 是 → /tdd （编写单元/集成测试）
│
├─ 测试关键用户流程？
│  └─ 是 → /e2e （编写端到端测试）
│
└─ 检查测试覆盖率？
    └─ /test-coverage （生成覆盖率报告）
```

**示例**：
```
用户：我需要测试用户可以完成完整的购买流程

Claude:
这是一个关键用户流程，让我创建 E2E 测试...

[调用 /e2e]
生成用户购买流程测试：

1. 浏览产品
2. 添加到购物车
3. 进入结账
4. 输入付款信息
5. 确认订单
6. 查看订单确认页面

运行测试...

测试结果：
✅ 产品浏览
✅ 购物车添加
✅ 结账流程
✅ 付款处理
✅ 订单确认

所有测试通过！
```

---

### 语言特定决策

#### Python 项目

```
Python 开发任务
├─ 需要审查代码吗？
│  └─ /python-review
│
├─ 需要编写测试吗？
│  └─ /tdd （python-testing skill 会激活）
│
└─ 需要最佳实践指导吗？
    └─ python-patterns skill 自动激活
```

#### Go 项目

```
Go 开发任务
├─ 构建失败吗？
│  └─ /go-build
│
├─ 需要审查代码吗？
│  └─ /go-review
│
├─ 需要编写测试吗？
│  └─ /go-test
│
└─ 需要最佳实践指导吗？
    └─ golang-patterns skill 自动激活
```

#### Rust 项目

```
Rust 开发任务
├─ 构建失败（borrow checker）？
│  └─ /rust-build
│
├─ 需要审查代码吗？
│  └─ /rust-review
│
├─ 需要编写测试吗？
│  └─ /rust-test
│
└─ 需要最佳实践指导吗？
    └─ rust-patterns skill 自动激活
```

#### Kotlin 项目

```
Kotlin/Android 开发任务
├─ 构建失败吗？
│  └─ /kotlin-build （或 /gradle-build）
│
├─ 需要审查代码吗？
│  └─ /kotlin-review
│
├─ 需要编写测试吗？
│  └─ /kotlin-test
│
└─ 需要最佳实践指导吗？
    └─ kotlin-patterns skill 自动激活
```

---

### 复杂场景决策

#### 多人协作项目

```
团队协作
├─ 准备提交代码？
│  ├─ 本地测试：/tdd + /e2e
│  └─ 代码审查：/code-review
│
├─ 审查别人的代码？
│  └─ /code-review （或语言特定 review）
│
└─ 发现可重用模式？
    └─ /learn （或 /skill-create）
```

#### 维护遗留代码

```
维护遗留代码
├─ 需要理解代码库吗？
│  └─ /plan （分析并创建计划）
│
├─ 需要修复 bug？
│  └─ /tdd （添加测试保护）
│
├─ 需要清理技术债务？
│  ├─ /code-review （识别问题）
│  └─ /refactor-clean （清理死代码）
│
└─ 需要添加测试？
    └─ /tdd （补充测试覆盖率）
```

---

## 最佳实践

### 命令使用

**DO（做）**：
- ✅ 在编码前使用 `/plan` 进行复杂功能
- ✅ 对新功能使用 `/tdd` 强制测试驱动开发
- ✅ 在提交前使用 `/code-review` 审查代码
- ✅ 对关键用户流程使用 `/e2e`
- ✅ 对学习到的模式使用 `/learn`
- ✅ 构建失败时使用 `/build-fix`

**DON'T（不做）**：
- ❌ 跳过测试编写
- ❌ 不审审查就提交代码
- ❌ 忽略 CRITICAL 或 HIGH 问题
- ❌ 在生产上运行涉及真实资金的 E2E 测试

### 代理使用

**DO（做）**：
- ✅ 让代理做它们擅长的事
- ✅ 信任代理的专长
- ✅ 为复杂任务使用正确的代理

**DON'T（不做）**：
- ❌ 试图做代理擅长的所有事
- ❌ 忽略代理的建议

### 测试覆盖率

- **80% 最小值**用于所有代码
- **100% 必需**用于：
  - 金融计算
  - 身份验证逻辑
  - 安全关键代码
  - 核心业务逻辑

### 代码质量

- **函数大小**：< 50 行
- **文件大小**：< 800 行
- **嵌套深度**：< 4 层
- **错误处理**：所有地方都必须有
- **不可变性**：优先不可变操作

### 安全

- ❌ **绝不**硬编码凭证
- ✅ **总是**验证用户输入
- ✅ **总是**使用参数化查询
- ✅ **总是**清理用户输出

---

## 快速参考

### 常用命令

| 命令 | 用途 |
|-------|------|
| `/plan` | 创建实施计划 |
| `/tdd` | 测试驱动开发 |
| `/e2e` | 端到端测试 |
| `/code-review` | 代码审查 |
| `/build-fix` | 修复构建错误 |
| `/learn` | 提取模式 |

### 语言特定命令

| 语言 | 构建 | 审查 | 测试 |
|-----|------|------|------|
| **Kotlin** | `/kotlin-build` | `/kotlin-review` | `/kotlin-test` |
| **C++** | `/cpp-build` | `/cpp-review` | `/cpp-test` |
| **Go** | `/go-build` | `/go-review` | `/go-test` |
| **Rust** | `/rust-build` | `/rust-review` | `/rust-test` |
| **Python** | - | `/python-review` | - |

### 常用代理

| 代理 | 用途 | 模型 |
|-----|------|------|
| `planner` | 实施计划 | Opus 4.6 |
| `tdd-guide` | TDD 指导 | Sonnet 4.6 |
| `code-reviewer` | 代码审查 | Sonnet 4.6 |
| `e2e-runner` | E2E 测试 | Sonnet 4.6 |
| `security-reviewer` | 安全审查 | - |

### 常用技能

| 技能 | 用途 |
|-----|------|
| `python-patterns` | Python 开发模式 |
| `frontend-patterns` | 前端开发模式 |
| `golang-patterns` | Go 模式 |
| `rust-patterns` | Rust 模式 |
| `kotlin-patterns` | Kotlin 模式 |
| `backend-patterns` | 后端模式 |
| `api-design` | API 设计模式 |

---

## 贡献

如果你想为 ECC 做出贡献，请遵循以下格式：

- **Agents**：带有 frontmatter 的 Markdown（name、description、tools、model）
- **Skills**：清晰的章节（何时使用、如何工作、示例）
- **Commands**：带有描述 frontmatter 的 Markdown
- **Hooks**：带有 matcher 和 hooks 数组的 JSON

文件命名：小写带连字符（例如，`python-reviewer.md`、`tdd-workflow.md`）

更多详情请参阅 CONTRIBUTING.md。

---

## 许可证

本项目根据 MIT 许可证授权。有关详情，请参阅 LICENSE 文件。

---

## 联系和支持

- **GitHub**：https://github.com/anthropics/everything-claude-code
- **Issues**：https://github.com/anthropics/everything-claude-code/issues
- **Discussions**：https://github.com/anthropics/everything-claude-code/discussions

---

**记住**：ECC 旨在帮助你编写更好的代码。遵循这些模式和实践，你将能够构建健壮、可维护和安全的软件。
