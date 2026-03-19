# 示例：Go Web 服务代码审查

这个示例展示 go-reviewer 如何审查 Go Web 服务代码。

## 审查的代码

### 1. main.go

```go
package main

import (
    "fmt"
    "net/http"
    "sync"
)

var db *Database

type Database struct {
    users map[string]User
    mu    sync.Mutex
}

type User struct {
    ID       int
    Username string
    Email    string
}

func main() {
    db = &Database{
        users: make(map[string]User),
    }

    http.HandleFunc("/users", handleUsers)
    http.HandleFunc("/users/", handleUserDetail)

    fmt.Println("Server starting on :8080")
    http.ListenAndServe(":8080", nil)
}

func handleUsers(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")

    db.mu.Lock()
    defer db.mu.Unlock()

    users := make([]User, 0, len(db.users))
    for _, user := range db.users {
        users = append(users, user)
    }

    json.NewEncoder(w).Encode(users)
}

func handleUserDetail(w http.ResponseWriter, r *http.Request) {
    username := r.URL.Path[len("/users/"):]
    if username == "" {
        http.Error(w, "Username required", http.StatusBadRequest)
        return
    }

    db.mu.Lock()
    defer db.mu.Unlock()

    user, exists := db.users[username]
    if !exists {
        http.Error(w, "User not found", http.StatusNotFound)
        return
    }

    json.NewEncoder(w).Encode(user)
}
```

### 2. worker.go

```go
package main

import (
    "time"
)

func StartWorker() {
    for {
        processTask()
        time.Sleep(1 * time.Second)
    }
}

func processTask() {
    // 处理任务
    fmt.Println("Processing task...")
}
```

### 3. cache.go

```go
package main

var cache map[string]interface{}

func init() {
    cache = make(map[string]interface{})
}

func SetCache(key string, value interface{}) {
    cache[key] = value
}

func GetCache(key string) (interface{}, bool) {
    value, exists := cache[key]
    return value, exists
}
```

## 审查报告

# Go 代码审查报告

## 概览
- 审查的文件：3 个
- 发现的 CRITICAL 问题：3 个
- 发现的 HIGH 问题：4 个
- 发现的 MEDIUM 问题：3 个
- 发现的 LOW 问题：2 个

---

## 🔴 CRITICAL 问题

### 1. 全局状态和依赖注入
**文件**: `main.go:11`
**等级**: CRITICAL
**问题**: 使用全局变量导致测试困难

```go
var db *Database  // 全局变量

func main() {
    db = &Database{...}
    http.HandleFunc("/users", handleUsers)
}
```

**问题**:
- 难以测试（无法 mock 数据库）
- 并发访问问题
- 隐式依赖关系

**建议**: 使用依赖注入：

```go
// server.go
type Server struct {
    db    *Database
    mux   *http.ServeMux
}

func NewServer(db *Database) *Server {
    mux := http.NewServeMux()
    s := &Server{db: db, mux: mux}

    mux.HandleFunc("/users", s.handleUsers)
    mux.HandleFunc("/users/", s.handleUserDetail)

    return s
}

func (s *Server) Start(addr string) error {
    return http.ListenAndServe(addr, s.mux)
}

func (s *Server) handleUsers(w http.ResponseWriter, r *http.Request) {
    s.db.mu.Lock()
    defer s.db.mu.Unlock()

    users := make([]User, 0, len(s.db.users))
    for _, user := range s.db.users {
        users = append(users, user)
    }

    json.NewEncoder(w).Encode(users)
}

// main.go
func main() {
    db := NewDatabase()
    server := NewServer(db)

    if err := server.Start(":8080"); err != nil {
        log.Fatalf("Server failed: %v", err)
    }
}

// server_test.go
func TestServer(t *testing.T) {
    // 可以轻松注入 mock 数据库
    mockDB := &MockDatabase{}
    server := NewServer(mockDB)

    // 测试 server
    req := httptest.NewRequest("GET", "/users", nil)
    w := httptest.NewRecorder()
    server.handleUsers(w, req)

    if w.Code != http.StatusOK {
        t.Errorf("Expected status 200, got %d", w.Code)
    }
}
```

---

### 2. 非线程安全的 map
**文件**: `cache.go:5`
**等级**: CRITICAL
**问题**: 全局 map 没有并发保护

```go
var cache map[string]interface{}  // 非线程安全

func SetCache(key string, value interface{}) {
    cache[key] = value  // 并发写入会 panic
}

func GetCache(key string) (interface{}, bool) {
    value, exists := cache[key]  // 并发读写会 panic
    return value, exists
}
```

**问题**: 多个 goroutine 同时访问会引发 panic

**建议**: 使用 sync.Map 或加锁：

```go
// 选项 1: 使用 sync.Map
import "sync"

var cache sync.Map  // 线程安全

func SetCache(key string, value interface{}) {
    cache.Store(key, value)
}

func GetCache(key string) (interface{}, bool) {
    return cache.Load(key)
}

// 选项 2: 使用 mutex
type Cache struct {
    mu    sync.RWMutex
    items map[string]interface{}
}

func NewCache() *Cache {
    return &Cache{
        items: make(map[string]interface{}),
    }
}

func (c *Cache) Set(key string, value interface{}) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.items[key] = value
}

func (c *Cache) Get(key string) (interface{}, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    value, exists := c.items[key]
    return value, exists
}
```

---

### 3. 潜在的 goroutine 泄漏
**文件**: `worker.go:5`
**等级**: CRITICAL
**问题**: goroutine 没有退出机制

```go
func StartWorker() {
    for {  // 无限循环，无法退出
        processTask()
        time.Sleep(1 * time.Second)
    }
}
```

**问题**:
- 无法优雅关闭
- 资源泄漏
- 测试困难

**建议**: 使用 context 控制生命周期：

```go
// worker.go
import (
    "context"
    "time"
)

func StartWorker(ctx context.Context) {
    ticker := time.NewTicker(1 * time.Second)
    defer ticker.Stop()

    for {
        select {
        case <-ctx.Done():
            // 优雅关闭
            log.Println("Worker shutting down...")
            return
        case <-ticker.C:
            processTask()
        }
    }
}

// main.go
func main() {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    // 启动 worker
    go StartWorker(ctx)

    // 设置信号处理
    sigChan := make(chan os.Signal, 1)
    signal.Notify(sigChan, os.Interrupt, syscall.SIGTERM)

    // 等待信号
    <-sigChan

    // 取消 context，所有 worker 会优雅关闭
    cancel()

    log.Println("Server shutting down...")
    time.Sleep(2 * time.Second)  // 等待清理完成
}

// worker_test.go
func TestStartWorker(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    processed := make(chan bool, 1)
    go func() {
        StartWorker(ctx)
        processed <- true
    }()

    // 等待一些处理
    time.Sleep(3 * time.Second)

    // 取消 worker
    cancel()

    // 验证 worker 已关闭
    select {
    case <-processed:
        // 正常关闭
    case <-time.After(5 * time.Second):
        t.Error("Worker did not shutdown in time")
    }
}
```

---

## 🟠 HIGH 问题

### 1. 错误处理缺失
**文件**: `main.go:28`
**等级**: HIGH
**问题**: 错误返回值被忽略

```go
json.NewEncoder(w).Encode(users)  // 忽略错误
```

**建议**: 处理错误：

```go
func (s *Server) handleUsers(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")

    s.db.mu.Lock()
    defer s.db.mu.Unlock()

    users := make([]User, 0, len(s.db.users))
    for _, user := range s.db.users {
        users = append(users, user)
    }

    if err := json.NewEncoder(w).Encode(users); err != nil {
        log.Printf("Failed to encode users: %v", err)
        http.Error(w, "Internal server error", http.StatusInternalServerError)
    }
}
```

---

### 2. 不安全的字符串操作
**文件**: `main.go:52`
**等级**: HIGH
**问题**: 手动提取 URL 路径参数不安全

```go
username := r.URL.Path[len("/users/"):]  // 可能越界
```

**问题**: 如果路径不是以 `/users/` 开头，会 panic

**建议**: 使用 path 包或 strings.TrimPrefix：

```go
import (
    "path"
    "strings"
)

// 选项 1: 使用 path 包
username := strings.TrimPrefix(r.URL.Path, "/users/")

// 选项 2: 使用 httprouter
import "github.com/julienschmidt/httprouter"

func (s *Server) handleUserDetail(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
    username := ps.ByName("username")
    // ...
}

// router setup
router := httprouter.New()
router.GET("/users/:username", s.handleUserDetail)

// 选项 3: 使用 httpRouter
import "github.com/gorilla/mux"

func (s *Server) handleUserDetail(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    username := vars["username"]
    // ...
}

// router setup
router := mux.NewRouter()
router.HandleFunc("/users/{username}", s.handleUserDetail)
```

---

### 3. 缺少 HTTP 方法验证
**文件**: `main.go:37`
**等级**: HIGH
**问题**: 没有验证 HTTP 方法

```go
func handleUsers(w http.ResponseWriter, r *http.Request) {
    // 接受所有 HTTP 方法
    // ...
}
```

**建议**: 验证 HTTP 方法：

```go
func (s *Server) handleUsers(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodGet {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }

    w.Header().Set("Content-Type", "application/json")
    // ...
}

// 或使用方法路由
func (s *Server) handleUsers(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case http.MethodGet:
        s.getUsers(w, r)
    case http.MethodPost:
        s.createUser(w, r)
    default:
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
    }
}
```

---

### 4. 缺少输入验证
**文件**: `main.go:45`
**等级**: HIGH
**问题**: 没有验证用户名

```go
username := strings.TrimPrefix(r.URL.Path, "/users/")
if username == "" {
    http.Error(w, "Username required", http.StatusBadRequest)
    return
}

// 没有进一步验证
```

**建议**: 添加输入验证：

```go
func (s *Server) handleUserDetail(w http.ResponseWriter, r *http.Request) {
    username := strings.TrimPrefix(r.URL.Path, "/users/")

    if err := validateUsername(username); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    // ...
}

func validateUsername(username string) error {
    if username == "" {
        return errors.New("username required")
    }

    if len(username) < 3 {
        return errors.New("username too short")
    }

    if len(username) > 50 {
        return errors.New("username too long")
    }

    // 验证只包含字母数字和下划线
    for _, r := range username {
        if !isValidUsernameRune(r) {
            return errors.New("username contains invalid characters")
        }
    }

    return nil
}

func isValidUsernameRune(r rune) bool {
    return (r >= 'a' && r <= 'z') ||
           (r >= 'A' && r <= 'Z') ||
           (r >= '0' && r <= '9') ||
           r == '_'
}
```

---

## 🟡 MEDIUM 问题

### 1. 缺少请求超时
**文件**: `main.go`
**等级**: MEDIUM
**问题**: 请求没有超时控制

**建议**: 添加超时：

```go
func (s *Server) handleUsers(w http.ResponseWriter, r *http.Request) {
    ctx, cancel := context.WithTimeout(r.Context(), 5*time.Second)
    defer cancel()

    done := make(chan struct{})
    var users []User

    go func() {
        s.db.mu.Lock()
        defer s.db.mu.Unlock()

        users = make([]User, 0, len(s.db.users))
        for _, user := range s.db.users {
            users = append(users, user)
        }
        close(done)
    }()

    select {
    case <-done:
        // 正常完成
    case <-ctx.Done():
        // 超时
        http.Error(w, "Request timeout", http.StatusRequestTimeout)
        return
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(users)
}
```

---

### 2. 缺少日志
**文件**: 所有文件
**等级**: MEDIUM
**问题**: 没有日志记录

**建议**: 添加日志：

```go
import (
    "log/slog"
    "os"
)

func init() {
    // 配置日志
    handler := slog.NewTextHandler(os.Stdout, &slog.HandlerOptions{
        Level: slog.LevelInfo,
    })
    slog.SetDefault(slog.New(handler))
}

func (s *Server) handleUserDetail(w http.ResponseWriter, r *http.Request) {
    username := strings.TrimPrefix(r.URL.Path, "/users/")

    slog.Info("Handling user detail request",
        "username", username,
        "remote_addr", r.RemoteAddr,
    )

    if err := validateUsername(username); err != nil {
        slog.Warn("Invalid username",
            "username", username,
            "error", err,
        )
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    // ...
}
```

---

### 3. 缺少中间件
**文件**: `main.go`
**等级**: MEDIUM
**问题**: 没有使用中间件

**建议**: 添加常用中间件：

```go
// middleware.go
import (
    "net/http"
    "time"
)

func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()

        // 创建响应包装器以捕获状态码
        wrapped := &responseWriter{ResponseWriter: w}

        next.ServeHTTP(wrapped, r)

        duration := time.Since(start)
        slog.Info("Request completed",
            "method", r.Method,
            "path", r.URL.Path,
            "status", wrapped.status,
            "duration", duration,
        )
    })
}

func recoveryMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        defer func() {
            if err := recover(); err != nil {
                slog.Error("Panic recovered",
                    "error", err,
                    "path", r.URL.Path,
                )
                http.Error(w, "Internal server error", http.StatusInternalServerError)
            }
        }()
        next.ServeHTTP(w, r)
    })
}

func corsMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Access-Control-Allow-Origin", "*")
        w.Header().Set("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS")
        w.Header().Set("Access-Control-Allow-Headers", "Content-Type, Authorization")

        if r.Method == "OPTIONS" {
            w.WriteHeader(http.StatusOK)
            return
        }

        next.ServeHTTP(w, r)
    })
}

type responseWriter struct {
    http.ResponseWriter
    status int
}

func (rw *responseWriter) WriteHeader(status int) {
    rw.status = status
    rw.ResponseWriter.WriteHeader(status)
}

// 使用中间件
func main() {
    db := NewDatabase()
    server := NewServer(db)

    // 应用中间件
    handler := recoveryMiddleware(
        loggingMiddleware(
            corsMiddleware(
                server.mux,
            ),
        ),
    )

    if err := http.ListenAndServe(":8080", handler); err != nil {
        log.Fatalf("Server failed: %v", err)
    }
}
```

---

## 总体评分: 6.8/10

## 优先修复建议

### 立即修复（CRITICAL）
1. ✅ 移除全局变量，使用依赖注入
2. ✅ 修复非线程安全的 map
3. ✅ 添加 goroutine 退出机制

### 高优先级（HIGH）
1. ✅ 处理所有错误
2. ✅ 使用安全的路由参数提取
3. ✅ 验证 HTTP 方法
4. ✅ 添加输入验证

### 中优先级（MEDIUM）
1. 添加请求超时
2. 添加日志记录
3. 使用中间件

## Go 最佳实践检查清单

- [ ] 使用依赖注入而非全局变量
- [ ] 所有错误都被检查和处理
- [ ] 使用 context 控制 goroutine 生命周期
- [ ] 并发安全（使用 sync 包或 channel）
- [ ] 使用强类型路由（gorilla/mux, chi, httprouter）
- [ ] 添加输入验证
- [ ] 使用中间件（日志、恢复、CORS）
- [ ] 添加测试（单元测试、集成测试）
- [ ] 使用 go vet 和静态分析工具
- [ ] 添加文档注释

## 推荐的 Go 工具

| 工具 | 用途 |
|------|------|
| go vet | 静态分析 |
| golangci-lint | 代码检查 |
| go test | 单元测试 |
| go cover | 测试覆盖率 |
| pprof | 性能分析 |
| race detector | 竞态检测 |

### 运行检查

```bash
# 静态分析
go vet ./...

# 使用 golangci-lint
golangci-lint run

# 运行测试
go test ./...

# 测试覆盖率
go test -cover ./...

# 竞态检测
go test -race ./...

# 基准测试
go test -bench=. ./...
```
