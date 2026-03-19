# 团队代码审查

这个场景展示了如何在团队环境中建立和执行有效的代码审查流程。

## 🎯 场景概述

### 适用情况

- 多人协作的团队
- 需要建立代码审查规范
- 大型项目或多模块项目
- 新成员入队需要培训

### 核心挑战

- 建立统一的审查标准
- 平衡审查质量和开发速度
- 处理审查意见分歧
- 培养团队审查文化

## 📋 使用场景

### 场景 1: 建立 Code Review 规范

**目标：** 为团队制定代码审查标准

**内容包括：**
- 审查流程定义
- 检查清单制定
- 工具配置（GitHub/GitLab）
- 培训材料准备

**Agent 使用：**
- Code Reviewer Agent 提供最佳实践
- Planner Agent 设计审查流程
- Doc Updater Agent 创建文档

---

### 场景 2: 多人协作审查

**目标：** 优化多人审查流程

**挑战：**
- 协调多个审查者时间
- 合并多组反馈
- 避免重复工作
- 处理意见冲突

**策略：**
- 分配审查责任
- 使用审查模板
- 定期同步会议
- 建立仲裁机制

---

### 场景 3: 审查流程优化

**目标：** 改进现有审查流程

**优化方向：**
- 自动化预检查
- 缩短反馈循环
- 提高审查效率
- 增强审查质量

**Agent 使用：**
- 分析审查指标
- 识别瓶颈
- 提出改进方案
- 实施自动化工具

## 🚀 团队审查最佳实践

### 1. 建立规范

```markdown
# 团队 Code Review 规范

## 审查范围
- 所有代码变更必须审查
- 配置文件变更需要技术负责人审查
- 安全相关代码需要安全专家审查

## 审查者
- 至少一人审查
- 代码所有者不能审查
- 关键变更需要两人审查

## 响应时间
- 首次审查：24 小时内
- 反馈修复：48 小时内
- 紧急问题：立即响应

## 审查标准
- 代码符合编码规范
- 没有安全漏洞
- 测试覆盖充分
- 文档更新完整
```

### 2. 使用工具

**静态分析工具：**
- ESLint / Pylint / go vet
- SonarQube 代码质量分析
- Snyk 安全扫描

**审查工具：**
- GitHub PR Review
- GitLab Merge Request
- Phabricator Differential

### 3. 培训团队

```bash
# 使用 Agent 创建培训材料
Agent(doc-updater, """
创建 Code Review 培训材料，包括：

1. 基本概念
   - 什么是 Code Review
   - 为什么需要 Code Review
   - Code Review 的价值

2. 审查流程
   - 如何发起审查
   - 如何进行审查
   - 如何响应反馈

3. 最佳实践
   - 建设性反馈
   - 具体明确
   - 关注重要问题
   - 尊重审查者

4. 工具使用
   - GitHub PR Review
   - 静态分析工具
   - 自动化检查

5. 常见场景
   - 小改动快速审查
   - 大改动分批审查
   - 紧急修复快速通道
""")
```

### 4. 监控指标

```javascript
// 审查指标追踪
const ReviewMetrics = {
  // 审查速度
  avgReviewTime: 4.2, // 小时
  avgFixTime: 6.8,   // 小时

  // 审查质量
  bugsFound: 15,
  securityIssues: 3,
  codeQualityIssues: 42,

  // 团队参与
  reviewersCount: 8,
  activeReviewers: 7,
  coverage: 87.5, // % 代码经过审查

  // 审查效率
  passRate: 68.2, // % 一次通过
  commentDensity: 1.5, // 平均每个 PR 评论数

  // 趋势
  trend: 'improving' // improving, stable, declining
};
```

## 💡 建立审查文化

### 1. 建设性沟通

**审查者应该：**
- 具体明确地指出问题
- 提供改进建议
- 解释原因
- 尊重作者的付出

**被审查者应该：**
- 积极响应反馈
- 解释设计决策
- 主动修复问题
- 感谢审查者

### 2. 频防冲突

**冲突解决机制：**
1. 优先级排序（CRITICAL > HIGH > MEDIUM）
2. 技术负责人仲裁
3. 团队讨论达成共识
4. 遵循团队规范

### 3. 持续改进

**定期回顾：**
- 每周审查流程会议
- 每月审查质量分析
- 每季度规范更新
- 年度团队培训

## 🔗 相关资源

- [Code Review 指南](../../01-commands/03-code-review/) - 详细 Code Review 指南
- [Code Reviewer Agent](../../02-agents/03-code-reviewer/) - 自动化审查工具
- [Code Review 示例](../../01-commands/03-code-review/examples/) - 实战案例

---

**提示：** 团队 Code Review 不仅关乎代码质量，还关乎团队协作和学习。建立良好的审查文化可以显著提高团队整体能力。
