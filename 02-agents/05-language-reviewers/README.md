# 语言特定代码审查代理

本目录包含针对不同编程语言的专用代码审查代理。这些代理在通用 code-reviewer 的基础上，提供语言特定的深度分析。

## 可用代理

### java-reviewer

专门审查 Java 代码，特别是 Spring Boot 应用。

**使用场景**:
- Java/Spring Boot 项目
- Java EE 应用
- Android 应用（Java 部分）

**审查重点**:
- Java 编码规范
- Spring Boot 最佳实践
- JPA/Hibernate 使用
- 并发和线程安全
- 依赖注入
- 异常处理

**触发条件**:
```bash
# Java 项目中自动调用
Agent(java-reviewer, "审查 Spring Boot 应用代码")
```

**示例输出**:
```markdown
# Java 代码审查报告

## Spring Boot 最佳实践

### ⚠️ 事务传播配置不正确
**文件**: `src/main/java/com/example/service/OrderService.java:45`
**等级**: HIGH

**问题**: 使用了错误的传播级别

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void processOrder(Long orderId) {
    // 这会创建新事务，可能破坏数据一致性
}
```

**建议**: 对于需要与调用者在同一事务的操作，使用默认传播级别：
```java
@Transactional
public void processOrder(Long orderId) {
    // 使用默认传播级别 (REQUIRED)
}
```

---

### ⚠️ 未使用构造器注入
**文件**: `src/main/java/com/example/controller/UserController.java:12`
**等级**: MEDIUM

**问题**: 使用字段注入而非构造器注入

```java
@Autowired
private UserService userService;
```

**建议**: 使用构造器注入提高可测试性：
```java
private final UserService userService;

public UserController(UserService userService) {
    this.userService = userService;
}
```

---

## JPA 使用问题

### 🔴 N+1 查询问题
**文件**: `src/main/java/com/example/repository/TaskRepository.java:32`
**等级**: HIGH

**问题**: 访问关联实体时触发额外的查询

```java
@Entity
public class Task {
    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;  // 没有配置加载策略
}

// 使用时
List<Task> tasks = taskRepository.findAll();
for (Task task : tasks) {
    System.out.println(task.getUser().getName());  // 触发 N+1 查询
}
```

**建议**: 使用 JOIN FETCH 或 @EntityGraph：
```java
@Query("SELECT t FROM Task t JOIN FETCH t.user")
List<Task> findAllWithUser();

// 或使用 @EntityGraph
@EntityGraph(attributePaths = {"user"})
"})
List<Task> findAll();
```

---

## 并发问题

### 🔴 非线程安全的共享状态
**文件**: `src/main/java/com/example/service/CacheService.java:15`
**等级**: CRITICAL

**问题**: 共享状态没有适当的同步

```java
public class CacheService {
    private Map<String, Object> cache = new HashMap<>();  // 非线程安全

    public void put(String key, Object value) {
        cache.put(key, value);
    }

    public Object get(String key) {
        return cache.get(key);
    }
}
```

**建议**: 使用线程安全的集合或添加同步：
```java
private final ConcurrentHashMap<String, Object> cache = new ConcurrentHashMap<>();

// 或使用 synchronized
private final Map<String, Object> cache = new HashMap<>();

public synchronized void put(String key, Object value) {
    cache.put(key, value);
}

public synchronized Object get(String key) {
    return cache.get(key);
}
```

---

## 总体评分: 8.2/10

代码质量良好，但需要修复 HIGH 等级问题。
```

---

### python-reviewer

专门审查 Python 代码，特别是 Django 和 Flask 应用。

**使用场景**:
- Python Web 应用（Django, Flask, FastAPI）
- 数据科学项目
- 自动化脚本

**审查重点**:
- PEP 8 编码规范
- Django/Flask 最佳实践
- 类型注解
- 异步编程
- 虚拟环境管理

**触发条件**:
```bash
# Python 项目中自动调用
Agent(python-reviewer, "审查 Django 应用代码")
```

**示例输出**:
```markdown
# Python 代码审查报告

## PEP 8 编码规范

### ⚠️ 违反命名约定
**文件**: `src/services/user_service.py.py:23`
**等级**: LOW

**问题**: 函数名使用驼峰命名

```python
def getUserById(user_id):  # 应该使用 snake_case
    pass
```

**建议**: 使用 PEP 8 推荐的 snake_case：
```python
def get_user_by_id(user_id):
    pass
```

---

### ⚠️ 缺少类型注解
**文件**: `src/services/calculation_service.py:15`
**等级**: MEDIUM

**问题**: 函数参数和返回值缺少类型注解

```python
def calculate_discount(price, discount_rate):
    return price * (1 - discount_rate)
```

**建议**: 添加类型注解提高代码可读性：
```python
from typing import Union

def calculate_discount(price: float, discount_rate: float) -> float:
    return price * (1 - discount_rate)
```

---

## Django 最佳实践

### 🔴 直接使用 .get() 可能引发异常
**文件**: `src/apps/users/views.py:45`
**等级**: HIGH

**问题**: 直接使用 `.get()` 可能引发 DoesNotExist 异常

```python
def user_detail(request, user_id):
    user = User.objects.get(id=user_id)  # 可能抛出异常
    return render(request, 'user_detail.html', {'user': user})
```

**建议**: 使用 `get_object_or_404`：
```python
from django.shortcuts import get_object_or_404

def user_detail(request, user_id):
    user = get_object_or_404(User, id=user_id)
    return render(request, 'user_detail.html', {'user': user})
```

---

### 🔴 N+1 查询问题
**文件**: `src/apps/tasks/views.py:32`
**等级**: HIGH

**问题**: 在循环中访问关联对象

```python
def task_list(request):
    tasks = Task.objects.all()
    for task in tasks:
        print(task.user.username)  # 触发 N+1 查询
    return render(request, 'task_list.html', {'tasks': tasks})
```

**建议**: 使用 select_related：
```python
def task_list(request):
    tasks = Task.objects.select_related('user').all()
    return render(request, 'task_list.html', {'tasks': tasks})
```

---

## 异步编程

### ⚠️ 同步阻塞调用在异步函数中
**文件**: `src/api/endpoints.py:28`
**等级**: HIGH

**问题**: 在异步函数中使用阻塞 I/O

```python
import asyncio
import time

async def fetch_data(url):
    time.sleep(1)  # 阻塞整个事件循环
    return {"data": "result"}
```

**建议**: 使用异步版本：
```python
import aiohttp

async def fetch_data(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.json()
```

---

## 总体评分: 7.8/10

遵循 P 8 规范，但需要优化数据库查询和异步编程。
```

---

### go-reviewer

专门审查 Go 代码。

**使用场景**:
- Go Web 应用
- 微服务
- 命令行工具
- 系统级程序

**审查重点**:
- Go 惯用语
- 错误处理
- 并发模式（goroutine, channel）
- 接口设计
- 性能优化

**触发条件**:
```bash
# Go 项目中自动调用
Agent(go-reviewer, "审查 Go 应用代码")
```

**示例输出**:
```markdown
# Go 代码审查报告

## Go 惯用语

### ⚠️ 错误的 error 检查
**文件**: `src/service/user_service.go:45`
**等级**: HIGH

**问题**: 忽略错误返回值

```go
user, _ := userRepo.FindByID(id)  // 忽略错误
```

**建议**: 始终检查错误：
```go
user, err := userRepo.FindByID(id)
if err != nil {
    return nil, fmt.Errorf("failed to find user: %w", err)
}
```

---

### ⚠️ 使用全局变量
**文件**: `src/config/config.go:12`
**等级**: MEDIUM

**问题**: 使用全局变量导致测试困难

```go
var db *sql.DB

func InitDB() {
    db, _ = sql.Open("mysql", dsn)
}
```

**建议**: 使用依赖注入：
```go
type Database struct {
    *sql.DB
}

func NewDatabase(dsn string) (*Database, error) {
    db, err := sql.Open("mysql", dsn)
    if err != nil {
        return nil, err
    }
    return &Database{DB: db}, nil
}
```

---

## 并发问题

### 🔴 潜在的 goroutine 泄漏
**文件**: `src/worker/pool.go:28`
**等级**: HIGH

**问题**: 创建 goroutine 但没有正确的退出机制

```go
func StartWorker() {
    go func() {
        for {
            processTask()
            // 没有退出机制
        }
    }()
}
```

**建议**: 使用 context 控制生命周期：
```go
func StartWorker(ctx context.Context) {
    go func() {
        ticker := time.NewTicker(1 * time.Second)
        defer ticker.Stop()

        for {
            select {
            case <-ctx.Done():
                return
            case <-ticker.C:
                processTask()
            }
        }
    }()
}
```

---

### 🔴 数据竞争
**文件**: `src/counter/counter.go:15`
**等级**: CRITICAL

**问题**: 多个 goroutine 同时访问共享变量

```go
type Counter struct {
    count int
}

func (c *Counter) Increment() {
    c.count++  // 非原子操作
}
```

**建议**: 使用 sync.Mutex 或 atomic：
```go
type Counter struct {
    mu    sync.Mutex
    count int
}

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}

// 或使用 atomic
import "sync/atomic"

func (c *Counter) Increment() {
    atomic.AddInt64(&c.count, 1)
}
```

---

## 总体评分: 8.5/10

代码质量良好，遵循 Go 惯用语，并发模式正确。
```

---

### rust-reviewer

专门审查 Rust 代码。

**使用场景**:
- 系统级应用
- WebAssembly
- 嵌入式开发
- 高性能应用

**审查重点**:
- 所有权和借用规则
- 错误处理（Result, Option）
- 并发安全
- Unsafe 使用
- 生命周期

**触发条件**:
```bash
# Rust 项目中自动调用
Agent(rust-reviewer, "审查 Rust 应用代码")
```

---

### kotlin-reviewer

专门审查 Kotlin 代码，特别是 Android 和 Spring Boot 应用。

**使用场景**:
- Android 应用
- Spring Boot 应用
- Kotlin Multiplatform
- 后端服务

**审查重点**:
- Kotlin 惯用语
- 协程安全
- 空值安全
- 扩展函数使用
- Android 特定最佳实践

**触发条件**:
```bash
# Kotlin 项目中自动调用
Agent(kotlin-reviewer, "审查 Kotlin 应用代码")
```

---

### cpp-reviewer

专门审查 C++ 代码。

**使用场景**:
- 系统级应用
- 游戏开发
- 嵌入式开发
- 高性能计算

**审查重点**:
- 内存安全
- 现代 C++ 特性（C++17/20）
- RAII 原则
- 智能指针使用
- 并发和线程安全

**触发条件**:
```bash
# C++ 项目中自动调用
Agent(cpp-reviewer, "审查 C++ 应用代码")
```

## 使用指南

### 自动触发

语言审查代理会根据项目类型自动触发：

```bash
# Java 项目 → 自动调用 java-reviewer
# Python 项目 → 自动调用 python-reviewer
# Go 项目 → 自动调用 go-reviewer
# Rust 项目 → 自动调用 rust-reviewer
# Kotlin 项目 → 自动调用 kotlin-reviewer
# C++ 项目 → 自动调用 cpp-reviewer
```

### 手动触发

你也可以手动触发特定语言的审查：

```bash
# 明确指定语言
Agent(java-reviewer, "深度审查 Spring Boot 应用")

# 指定文件范围
Agent(python-reviewer, "审查 Django models.py")

# 聚焦特定问题
Agent(go-reviewer, "检查并发问题")
"```

## 与通用 code-reviewer 的协作

语言审查代理与通用 code-reviewer 协作：

```
代码变更
    ↓
code-reviewer (通用检查)
    ↓
根据项目语言调用对应的 language-reviewer
    ↓
综合审查报告
```

## 学习资源

- [java-reviewer 示例](./examples/java-spring-boot-review.md)
- [python-reviewer 示例](./examples/python-django-review.md)
- [go-reviewer 示例](./examples/go-web-service-review.md)

## 下一步

- 选择你的语言并查看对应示例
- 或返回 [代理总览](../README.md)
