# 技术债清理

这个场景展示了如何使用 Claude Code 系统化地清理和减少技术债。

## 🎯 场景概述

### 适用情况

- 代码质量指标下降
- 技术债积累过多
- 新功能开发变慢
- 维护成本增加

### 核心挑战

- 平衡新功能和技术债
- 优先级排序
- 影响范围控制
- 验证清理效果

## 📋 使用场景

### 场景 1: 代码重复清理

**问题：** 代码重复率高（>10%）

**清理策略：**
- 使用 SonarQube 识别重复代码
- 提取公共函数/类
- 重构相似逻辑
- 验证测试通过

**Agent 使用：**
- Code Quality Analyzer Agent 识别重复
- Refactor Agent 执行重构
- Test Runner Agent 验证功能

---

### 场景 2: 复杂度降低

**问题：** 函数复杂度过高（圈复杂度 > 15）

**清理策略：**
- 使用复杂度分析工具定位
- 拆分大函数
- 提取子方法
- 应用设计模式

**Agent 使用：**
- Code Reviewer Agent 识别高复杂度
- Planner Agent 设计重构方案
- TDD Guide Agent 确保重构正确

---

### 场景 3: 过时依赖更新

**问题：** 依赖版本过旧，存在安全漏洞

**清理策略：**
- 使用 Snyk/npm audit 扫描
- 分批更新依赖
- 验证兼容性
- 运行完整测试套件

**Agent 使用：**
- Security Scanner Agent 识别漏洞
- Build Error Resolver Agent 修复版本冲突
- Test Runner Agent 验证更新

## 🚀 清理流程

### 步骤 1: 评估技术债

```bash
# 使用 Code Quality Analyzer
Agent(code-quality-analyzer, """
分析项目技术债：

1. 代码质量指标
   - 圈复杂度
   - 代码重复率
   - 测试覆盖率
   - 安全漏洞

2. 依赖健康
   - 过时依赖
   - 安全漏洞
   - 许可证问题

3. 代码异味
   - 魔法代码
   - 死代码
   - 未使用代码

请生成技术债报告和清理优先级。
""")
```

### 步骤 2: 制定清理计划

```bash
# 使用 Planner Agent
Agent(planner, """
根据技术债报告制定清理计划：

优先级规则：
1. 安全问题（CRITICAL）
2. 高复杂度（HIGH）
3. 代码重复（MEDIUM）
4. 过时依赖（LOW）

请为每个项目生成清理任务，包括：
- 任务描述
- 优先级
- 预计工作量
- 风险评估
- 依赖关系
""")
```

### 步骤 3: 执行清理

```bash
# 使用 Refactor Agent
Agent(refactor-cleaner, """
执行技术债清理：

清理任务：
1. 提取重复代码
2. 降低函数复杂度
3. 移除死代码
4. 更新依赖

要求：
- 保持测试通过
- 不破坏现有功能
- 提高代码可读性
- 改善性能（如果可能）
""")
```

## 📊 技术债度量

### 质量指标

| 指标 | 当前 | 目标 | 差距 |
|------|------|------|------|
| 圈复杂度 | 25 | 10 | -15 |
| 代码重复率 | 12% | 3% | -9% |
| 测试覆盖率 | 65% | 80% | +15% |
| 安全漏洞 | 5 | 0 | -5 |
| 代码异味 | 45 | 15 | -30 |

### 清理进度

```javascript
// 追踪清理进度
const DebtCleanupTracker = {
  metrics: {
    complexity: { current: 25, target: 10 },
    duplication: { current: 12, target: 3 },
    coverage: { current: 65, target: 80 },
    vulnerabilities: { current: 5, target: 0 }
  },

  updateMetric(metric, value) {
    this.metrics[metric].current = value;
    this.calculateProgress();
  },

  calculateProgress() {
    const totalMetrics = Object.keys(this.metrics).length;
    const completedMetrics = Object.values(this.metrics).filter(
      m => Math.abs(m.current - m.target) < m.target * 0.1
    ).length;

    return Math.round((completedMetrics / totalMetrics) * 100);
  }
};
```

## 💡 最佳实践

### 1. 渐进式清理

- 每次清理一小部分
- 频繁提交和测试
- 避免大规模重构

### 2. 风险控制

- 先在非关键代码清理
- 保留回滚能力
- 充分测试验证

### 3. 持续改进

- 每个 Sprint 分配技术债时间
- 集成到开发流程
- 定期评估技术债

### 4. 团队协作

- 代码审查确保清理质量
- 知识共享避免重复
- 文档化清理决策

## 🚨 常见陷阱

### 陷阱 1: 过度重构

**问题：** 一次性重构太大范围

**解决：** 小步前进，频繁验证

### 陷阱 2: 忽视新功能

**问题：** 只关注技术债，忽略业务需求

**解决：** 平衡新功能和技术债工作

### 陷阱 3: 缺乏测试

**问题：** 清理代码时破坏现有测试

**解决：** 先确保测试通过，再清理

### 陷阱 4: 没有记录

**问题：** 清理决策没有文档化

**解决：** 记录所有清理决策和理由

## 🔗 相关资源

- [Refactor Cleaner Agent](../../02-agents/06-specialized/) - 清理工具
- [Code Quality Analysis](../../02-agents/06-specialized/examples/03-code-quality-analysis.md) - 质量分析
- [TDD 场景](../../01-commands/02-tdd) - TDD 开发流程

---

**提示：** 技术债清理是持续的过程。定期评估和清理可以防止技术债失控。
