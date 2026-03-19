# 专业化代理

本目录包含针对特定领域的专业化代理。这些代理提供深度、专业的分析和指导。

## 可用代理

### architect

**职责**: 系统架构设计和决策

**使用场景**:
- 新项目架构设计
- 系统重构规划
- 技术栈选型
- 架构模式选择
- 分布式系统设计

**何时调用**:
```bash
# 手动触发架构设计
Agent(architect, "设计一个高可用的电商系统架构")

# 或在规划复杂功能时自动调用
```

**示例输出**:
```markdown
# 架构设计文档

## 系统概览

### 业务需求
设计一个高可用的电商系统，支持：
- 每秒 10,000+ 并发请求
- 99.9% 可用性
- 水平扩展能力
- 多区域部署

### 架构方案

```
[用户] → [CDN] → [负载均衡器]
                ↓
        [API 网关集群]
                ↓
    ┌───────────┴───────────┐
    ↓                       ↓
[微服务集群]        [事件总线集群]
    ↓                       ↓
[数据库集群]    ←  [缓存集群]
    ↓
[对象存储]
```

## 技术栈选型

### 后端服务
- **框架**: Spring Boot / Kotlin
- **架构**: 微服务架构
- **容器**: Docker + Kubernetes

### 数据存储
- **关系型数据库**: PostgreSQL (主从复制)
- **缓存**: Redis Cluster
- **搜索**: Elasticsearch
- **消息队列**: Apache Kafka

### 基础设施
- **CDN**: Cloudflare / AWS CloudFront
- **负载均衡**: HAProxy / AWS ALB
- **监控**: Prometheus + Grafana
- **日志**: ELK Stack

### 部署
- **容器编排**: Kubernetes
- **CI/CD**: GitHub Actions / ArgoCD
- **多区域**: AWS Multi-Region

## 架构模式

### 1. 微服务架构
将系统拆分为独立的微服务：
- 用户服务
- 商品服务
- 订单服务
- 支付服务
- 通知服务

### 2. CQRS 模式
分离读写操作：
- 写操作：使用强一致性模型
- 读操作：使用最终一致性模型
- 同步：通过事件总线

### 3. 事件驱动架构
使用事件驱动服务间通信：
- 订单创建 → 发布事件
- 库存服务 → 监听事件并扣减库存
- 支付服务 → 监听事件并处理支付
- 通知服务 → 监听事件并发送通知

## 可用性设计

### 冗余设计
- 所有服务至少 2 个实例
- 数据库主从复制
- Redis Cluster 集群模式
- 多 AZ 部署

### 故障转移
- 健康检查和自动重启
- 服务降级机制
- 熔断器模式
- 限流和降级

### 数据一致性
- 分布式事务（Saga 模式）
- 补偿事务
- 幂等性保证

## 性能优化

### 缓存策略
- 多级缓存（本地缓存 + Redis）
- 缓存预热
- 缓存穿透防护
- 缓存雪崩防护

### 数据库优化
- 读写分离
- 分库分表
- 索引优化
- 查询优化

### 异步处理
- 非核心流程异步化
- 使用消息队列解耦
- 批处理优化

## 安全设计

### 认证授权
- OAuth 2.0 / JWT
- RBAC 权限模型
- 多因素认证（MFA）

### 数据安全
- 敏感数据加密
- 数据脱敏
- 审计日志

### 网络安全
- HTTPS/TLS
- DDoS 防护
- WAF（Web 应用防火墙）

## 扩展性设计

### 水平扩展
- 无状态服务设计
- 会话共享（Redis）
- 负载均衡策略

### 垂直扩展
- 数据库分片
- 缓存分区
- 消息队列分区

## 成本估算

### 基础设施成本（月）
- 云服务器: $3,000
- 数据库: $1,500
- CDN: $500
- 负载均衡: $300
- 监控和日志: $200

**总计**: ~$5,500/月

## 风险和缓解措施

### 技术风险
- **风险**: 微服务间通信延迟
  - **缓解**: 异步通信，缓存优化

- **风险**: 分布式事务复杂性
  - **缓解**: 使用 Saga 模式

### 运维风险
- **风险**: 多区域部署复杂性
  - **缓解**: 使用托管服务（AWS）

### 业务风险
- **风险**: 数据一致性保证
  - **缓解**: 最终一致性模型
```

---

### security-reviewer

**职责**: 深度安全分析和漏洞扫描

**使用场景**:
- 安全漏洞扫描
- 渗透测试支持
- 安全审计
- 合规性检查
- 安全最佳实践验证

**何时调用**:
```bash
# 手动触发安全审查
Agent(security-reviewer, "进行全面的安全审计")

# 或在处理敏感功能时自动调用
```

**检查项目**:
- OWASP Top 10
- SQL 注入
- XSS 攻击
- CSRF 攻击
- 认证和授权
- 敏感数据泄露
- 依赖项漏洞
- 配置安全

---

### database-reviewer

**职责**: 数据库设计和查询优化

**使用场景**:
- 数据库设计审查
- SQL 查询优化
- 索引优化
- 数据库性能分析
- 迁移脚本审查

**何时调用**:
```bash
# 手动触发数据库审查
Agent(database-reviewer, "优化订单查询性能")

# 或在处理数据库相关代码时自动调用
```

**分析内容**:
- 查询性能分析
- 索引使用情况
- N+1 查询检测
- 慢查询识别
- 数据库模式设计
- 数据迁移安全性

---

### refactor-cleaner

**职责**: 代码重构和死代码清理

**使用场景**:
- 代码重构
- 死代码识别和清理
- 代码重复消除
- 代码组织优化
- 技术债务清理

**何时调用**:
```bash
# 手动触发重构清理
Agent(refactor-cleaner, "清理项目中的死代码")

# 或在代码审查时自动调用
```

**清理任务**:
- 识别未使用的代码
- 识别重复代码
- 检测过时的依赖
- 建议重构机会
- 组织代码结构

---

### docs-lookup

**职责**: 文档查询和代码示例获取

**使用场景**:
- 查找库/框架文档
- 获取代码示例
- API 参考
- 最佳实践文档

**何时调用**:
```bash
# 查找文档
Agent(docs-lookup, "查找 Spring Boot JWT 集成方法")

# 或在使用新库时自动调用
```

**文档来源**:
- 官方文档
- 上下文 7（MCP）
- Stack Overflow
- GitHub 仓库
- 社区资源

---

### doc-updater

**职责**: 文档更新和代码映射生成

**使用场景**:
- 更新项目文档
- 生成代码映射（CODEMAPS）
- 更新 README
- 生成 API 文档

**何时调用**:
```bash
# 更新文档
Agent(doc-updater, "更新项目文档和代码映射")

# 或在代码变更后自动调用
```

**更新内容**:
- README.md
- CODEMAPS
- API 文档
- 架构文档
- 变更日志

---

### harness-optimizer

**职责**: ECC Harness 配置优化

**使用场景**:
- 优化 Harness 性能
- 调整配置设置
- 优化代理调用策略
- 提成本优化

**何时调用**:
```bash
# 优化 Harness
Agent(harness-optimizer, "优化 ECC Harness 配置")
```

**优化内容**:
- 代理调用策略
- 模型选择优化
- 上下文管理
- 成本优化
- 性能调优

---

### loop-operator

**职责**: 自主代理循环的操作和监控

**使用场景**:
- 管理长期运行的代理循环
- 监控循环进度
- 处理循环卡死
- 中断和恢复循环

**何时调用**:
```bash
# 管理代理循环
Agent(loop-operator, "监控并管理代理循环")
```

**操作内容**:
- 循环状态监控
- 进度跟踪
- 异常处理
- 优雅停止
- 超时管理

---

### chief-of-staff

**职责**: 个人通信管理和消息分类

**使用场景**:
- 邮件管理
- Slack 消息处理
- LINE 消息管理
- 消息分类和优先级
- 草稿回复生成

**何时调用**:
```bash
# 管理通信
Agent(chief-of-staff, "处理今天的邮件和 Slack 消息")
```

**功能**:
- 消息分类（4个层级）
  - skip（跳过）
  - info_only（仅信息）
  - meeting_info（会议信息）
  - action_required（需要行动）

- 自动起草回复
- 消息优先级排序
- 后续跟进提醒

---

## 使用指南

### 手动调用

```bash
# 架构设计
Agent(architect, "设计一个分布式任务队列系统")

# 安全审查
Agent(security-reviewer, "审计支付模块的安全性")

# 数据库优化
Agent(database-reviewer, "优化用户查询性能")



# 代码重构
Agent(refactor-cleaner, "清理未使用的代码和依赖")

# 文档查询
Agent(docs-lookup, "如何使用 Redis 实现分布式锁？")

# 更新文档
Agent(doc-updater, "更新项目文档")

# 优化 Harness
Agent(harness-optimizer, "优化代理调用成本")

# 管理通信
Agent(chief-of-staff, "处理今天的消息")
```

### 自动调用

专业化代理会在特定场景下自动调用：

| 代理 | 自动调用场景 |
|------|-------------|
| architect | 复杂功能规划、技术选型 |
| security-reviewer | 处理敏感数据、认证模块 |
| database-reviewer | 复杂数据库操作、性能问题 |
| refactor-cleaner | 代码审查、技术债务清理 |
| docs-lookup | 使用新库/框架 |
| doc-updater | 代码变更后 |
| harness-optimizer | 性能问题、成本优化 |
| loop-operator | 长期运行的代理任务 |
| chief-of-staff | 消息管理任务 |

### 组合使用

专业化代理可以组合使用：

```bash
# 先设计架构，再审查安全
Agent(architect, "设计用户认证系统")
    ↓
Agent(security-reviewer, "审查认证系统的安全性")

# 先优化数据库，再更新文档
Agent(database-reviewer, "优化查询性能")
    ↓
Agent(doc-updater, "更新数据库文档")

# 先清理代码，再审查安全
Agent(refactor-cleaner, "清理死代码")
    ↓
Agent(security-reviewer, "安全审查")
```

## 与其他代理的协作

### 与核心代理协作

```
planner (规划实现)
    ↓
architect (架构设计)
    ↓
tdd-guide (TDD 实现)
    ↓
security-reviewer (安全审查)
    ↓
database-reviewer (数据库审查)
    ↓
code-reviewer (代码审查)
    ↓
doc-updater (更新文档)
```

### 与语言审查代理协作

```
java-reviewer (Java 代码审查)
    ↓
security-reviewer (深度安全分析)
    ↓
database-reviewer (数据库查询优化)
```

## 学习资源

- [architect 示例](./examples/architect-design.md)
- [security-reviewer 示例](./examples/security-audit.md)
- [database-reviewer 示例](./examples/database-optimization.md)
- [refactor-cleaner 示例](./examples/refactor-cleanup.md)

## 下一步

- 查看 [architect 设计示例](./examples/architect-design.md)
- 或返回 [代理总览](../README.md)
