# 使用场景：使用 Go 构建高性能 API 网关

这个示例展示如何结合使用 ECC 的 Go 技能和代理来构建一个高性能、生产级的 Go API 网关。

## 场景描述

**任务**: 构建一个 API 网关，提供统一入口和跨域请求转发

**功能要求**:
- 服务发现和负载均衡
- 请求路由和重写
- 认证和授权中间件
- 速率限制
- 请求/响应日志
- 健康检查
- 指标收集（Prometheus）
- 优雅关闭
- 配置热更新

**技术栈**: Go 1.21 + Chi Router + Prometheus + Redis

## ECC 辅助开发流程

### 步骤 1: 项目初始化

```
用户: 创建一个新的 Go API 网关项目

ECC 检测到:
- 项目类型: Go (go.mod)
- 框架: Chi Router (检测到依赖或配置)
- 激活技能: golang-patterns, golang-testing, backend-patterns
```

根据 `golang-patterns` 和 `backend-patterns` 技能的建议：

```go
// 项目结构
api-gateway/
├── cmd/
│   └── gateway/
│       └── main.go                 # 应用入口
├── internal/
│   ├── config/
│   │   ├── config.go                # 配置结构
│   │   └── loader.go                # 配置加载
│   ├── handler/
│   │   ├── health.go                # 健康检查处理器
│   │   ├── metrics.go               # 指标处理器
│   │   └── proxy.go                 # 代理处理器
│   ├── middleware/
│   │   ├── auth.go                  # 认证中间件
│   │   ├── ratelimit.go             # 速率限制中间件
│   │   ├── logging.go               # 日志中间件
│   │   ├── recovery.go              # 恢复中间件
│   │   └── cors.go                  # CORS 中间件
│   ├── service/
r│   │   ├── discovery.go             # 服务发现
│   │   ├── loadbalancer.go           # 负载均衡
│   │   ├── proxy.go                 # 代理服务
│   │   └── metrics.go               # 指标服务
│   ├── model/
│   │   ├── service.go               # 服务模型
│   │   ├── request.go               # 请求模型
│   │   └── response.go              # 响应模型
│   └── repository/
│       └── service_repo.go          # 服务仓库
├── pkg/
│   ├── logger/
│   │   └── logger.go                # 日志工具
│   └── util/
│       └── util.go                  # 通用工具
├── api/
│   └── http/
│       └── routes.go                # 路由定义
├── config/
│   ├── config.yaml                   # 默认配置
│   └── config.example.yaml          # 配置示例
├── deployments/
│   ├── Dockerfile
│   └── k8s.yaml                    # Kubernetes 部署
├── scripts/
│   ├── build.sh
│   └── test.sh
├── go.mod
├── go.sum
├── Makefile
└── README.md
```

---

### 步骤 2: 核心模型和配置

根据 `golang-patterns` 技能（Go 习语、错误处理、并发安全）：

```go
// internal/model/service.go
package model

import "time"

// Service 表示一个后端服务
type Service struct {
    ID       string
    Name     string
    Host     string
    Port     int
    Protocol string
    Weight   int
    Healthy  bool
    LastSeen time.Time
    Metadata map[string]string
}

// Route 表示一个路由规则
type Route struct {
    ID          string
    Path        string
    Method      string
    ServiceID   string
    StripPrefix bool
    Middleware  []string
    Config      RouteConfig
}

type RouteConfig struct {
    Timeout         time.Duration
    RetryCount      int
    RetryDelay      time.Duration
    CircuitBreaker   CircuitBreakerConfig
    RateLimit       RateLimitConfig
}

// CircuitBreakerConfig 熔断器配置
type CircuitBreakerConfig struct {
    Enabled         bool
    FailureThreshold int
    SuccessThreshold int
    Timeout          time.Duration
}

// RateLimitConfig 速率限制配置
type RateLimitConfig struct {
    Enabled  bool
    Requests int
    Window  time.Duration
}
```

```go
// internal/config/config.go
package config

import (
    "time"
)

// Config 应用配置
type和其他 Config struct {
    Server   ServerConfig
    Gateway  GatewayConfig
    Services []ServiceConfig
    Routes   []RouteConfig
    Metrics  MetricsConfig
    Logging  LoggingConfig
}

type ServerConfig struct {
    Addr         string
    ReadTimeout  time.Duration
    WriteTimeout time.Duration
    IdleTimeout  time.Duration
}

type GatewayConfig struct {
    DiscoveryType      string // "consul", "static", "kubernetes"
    LoadBalancerType   string // "round_robin", "weighted", "least_connections"
    RateLimitEnabled   bool
    RateLimitRequests  int
    RateLimitWindow    time.Duration
}

type ServiceConfig struct {
    ID       string
    Name     string
    Host     string
    Port     int
    Weight   int
    Protocol string
}

type RouteConfig struct {
    ID          string
    Path        string
    Method      string
    ServiceID   string
    StripPrefix bool
    Timeout     time.Duration
}

type MetricsConfig struct {
    Enabled bool
    Port    int
    Path    string
}

type LoggingConfig struct {
    Level  string // "debug", "info", "warn", "error"
    Format string // "json", "text"
    Output string // "stdout", "file"
}
```

**golang-patterns 技能提供的建议**:
- ✅ 使用结构体标签进行 JSON/YAML 解码
- ✅ 使用 time.Duration 而非字符串表示时间
- ✅ 遵循 Go 命名约定（驼峰命名、导出函数大写开头）
- ✅ 使用接口定义契约
- ✅ 包级文档注释

---

### 步骤 3: 服务发现和负载均衡

根据 `golang-patterns` 技能（并发模式、接口设计）：

```go
// internal/service/discovery.go
package service

import (
    "context"
    "sync"
    "time"
)

// Discovery 服务发现接口
type Discovery interface {
    // Start 启动服务发现
    Start(ctx context.Context) error
    // Stop 停止服务发现
    Stop() error
    // GetServices 获取所有服务
    GetServices(ctx context.Context) ([]model.Service, error)
    // GetService 获取特定服务
    GetService(ctx context.Context, id string) (*model.Service, error)
    // Watch 监听服务变化
    Watch(ctx context.Context) (<-chan []model.Service, error)
}

// StaticDiscovery 静态服务发现实现
type StaticDiscovery struct {
    mu       sync.RWMutex
    services map[string]model.Service
    watcher  chan []model.Service
    done     chan struct{}
}

func NewStaticDiscovery(configs []config.ServiceConfig) *StaticDiscovery {
    d := &StaticDiscovery{
        services: make(map[string]model.Service),
        watcher:  make(chan []model.Service, 1),
        done:     make(chan struct{}),
    }

    d.loadServices(configs)
    return d
}

func (d *StaticDiscovery) loadServices(configs []config.ServiceConfig) {
    d.mu.Lock()
    defer d.mu.Unlock()

    for _, cfg := range configs {
        d.services[cfg.ID] = model.Service{
            ID:       cfg.ID,
            Name:     cfg.Name,
            Host:     cfg.Host,
            Port:     cfg.Port,
            Protocol: cfg.Protocol,
            Weight:   cfg.Weight,
            Healthy:  true,
            LastSeen: time.Now(),
        }
    }
}

func (d *StaticDiscovery) Start(ctx context.Context) error {
    close(d.watcher)
    return nil
}

func (d *StaticDiscovery) Stop() error {
    return nil
}

func (d *StaticDiscovery) GetServices(ctx context.Context) ([]model.Service, error) {
    d.mu.RLock()
    defer d.mu.RUnlock()

    services := make([]model.Service, 0, len(d.services))
    for _, service := range d.services {
        services = append(services, service)
    }
    return services, nil
}

func (d *StaticDiscovery) GetService(ctx context.Context, id string) (*model.Service, error) {
    d.mu.RLock()
    defer d.mu.RUnlock()

    if service, ok := d.services[id]; ok {
        return &service, nil
    }
    return nil, ErrServiceNotFound
}

func (d *StaticDiscovery) Watch(ctx context.Context) (<-chan []model.Service, error) {
    return d.watcher, nil
}

// LoadBalancer 负载均衡接口
type LoadBalancer interface {
    // Next 获取下一个服务实例
    Next(ctx context.Context, serviceID string) (*model.Service, error)
    // UpdateHealth 更新服务健康状态
    UpdateHealth(ctx context.Context, serviceID string, host string, healthy bool)
}

// RoundRobinLoadBalancer 轮询负载均衡
type RoundRobinLoadBalancer struct {
    mu    sync.Mutex
    index map[string]int
}

func NewRoundRobinLoadBalancer() *RoundRobinLoadBalancer {
    return &RoundRobinLoadBalancer{
        index: make(map[string]int),
    }
}

func (lb *RoundRobinLoadBalancer) Next(
    ctx context.Context,
    serviceID string,
) (*model.Service, error) {
    services, err := discovery.GetServices(ctx)
    if err != nil {
        return nil, err
    }

    // 过滤出指定服务的所有实例
    var instances []model.Service
    for _, service := range services {
        if service.ID == serviceID && service.Healthy {
            instances = append(instances, service)
        }
    }

    if len(instances) == 0 {
        return nil, ErrNoHealthyInstances
    }

    lb.mu.Lock()
    defer lb.mu.Unlock()

    idx := lb.index[serviceID]
    lb.index[serviceID] = (idx + 1) % len(instances)

    return &instances[idx], nil
}
```

**golang-patterns 技能提供的建议**:
- ✅ 使用 sync.RWMutex 保护读多写少的并发访问
- ✅ 使用 context.Context 传播取消信号
- ✅ 定义接口而不是具体类型
- ✅ 使用错误变量，而非字符串错误
- ✅ 恰当使用 defer 清理资源

---

### 步骤 4: 代理服务

根据 `golang-patterns` 技能（错误处理、资源管理）：

```go
// internal/service/proxy.go
package service

import (
    "bytes"
    "context"
    "io"
    "net/http"
    "net/url"
    "time"

    "github.com/pkg/errors"
)

// ProxyService 代理服务
type ProxyService struct {
    discovery Discovery
    balancer  LoadBalancer
    client    *http.Client
    metrics   *MetricsService
    logger    *logger.Logger
}

func NewProxyService(
    discovery Discovery,
    balancer LoadBalancer,
    client *http.Client,
    metrics *MetricsService,
    logger *logger.Logger,
) *ProxyService {
    return &ProxyService{
        discovery: discovery,
        balancer:  balancer,
        client:    client,
        metrics:   metrics,
        logger:    logger,
    }
}

func (p *ProxyService) ProxyRequest(
    ctx context.Context,
    req *http.Request,
    route model.Route,
) (*http.Response, error) {
    startTime := time.Now()

    // 获取服务实例
    service, err := p.balancer.Next(ctx, route.ServiceID)
    if err != nil {
        p.metrics.RecordFailure(route.ID, "no_healthy_instances")
        return nil, errors.Wrap(err, "failed to get service instance")
    }

    // 构建目标 URL
    targetURL, err := p.buildTargetURL(req, service, route)
    if err != nil {
        p.metrics.RecordFailure(route.ID, "url_build_failed")
        return nil, errors.Wrap(err, "failed to build target URL")
    }

    // 克隆请求并修改
    proxyReq, err := p.buildProxyRequest(ctx, req, targetURL)
    if err != nil {
        p.metrics.RecordFailure(route.ID, "request_build_failed")
        return nil, errors.Wrap(err, "failed to build proxy request")
    }

    // 执行代理请求
    resp, err := p.executeProxyRequest(proxyReq)
    if err != nil {
        p.metrics.RecordFailure(route(route.ID), "proxy_request_failed")
        p.logger.Error("Proxy request failed",
            "route", route.ID,
            "service", service.ID,
            "error", err,
        )
        return nil, errors.Wrap(err, "failed to execute proxy request")
    }

    // 记录成功指标
    duration := time.Since(startTime)
    p.metrics.RecordSuccess(route.ID, int(duration/time.Millisecond))

    p.logger.Info("Proxy request completed",
        "route", route.ID,
        "service", service.ID,
        "status", resp.StatusCode,
        "duration_ms", duration.Milliseconds(),
    )

    return resp, nil
}

func (p *ProxyService) buildTargetURL(
    req *http.Request,
    service *model.Service,
    route model.Route,
) (*url.URL, error) {
    scheme := service.Protocol
    if scheme == "" {
        scheme = req.URL.Scheme
    }

    targetURL := &url.URL{
        Scheme: scheme,
        Host:   fmt.Sprintf("%s:%d", service.Host, service.Port),
    }

    if route.StripPrefix {
        targetURL.Path = strings.TrimPrefix(req.URL.Path, route.Path)
    } else {
        targetURL.Path = req.URL.Path
    }

    targetURL.RawQuery = req.URL.RawQuery
    targetURL.Fragment = req.URL.Fragment

    return targetURL, nil
}

func (p *ProxyService) buildProxyRequest(
    ctx context.Context,
    original *http.Request,
    targetURL *url.URL,
) (*http.Request, error) {
    // 创建新请求
    proxyReq, err := http.NewRequestWithContext(
        ctx,
        original.Method,
        targetURL.String(),
        original.Body,
    )
    if err != nil {
        return nil, err
    }

    // 复制头
    for key, values := range original.Header {
        // 跳过某些头
        if shouldSkipHeader(key) {
            continue
        }
        proxyReq.Header[key] = values
    }

    // 更新 Host 头
    proxyReq.Host = targetURL.Host

    // 更新 X-Forwarded-For 头
    updateForwardedHeaders(original, proxyReq)

    return proxyReq, nil
}

func (p *ProxyService) executeProxyRequest(
    req *http.Request,
) (*http.Response, error) {
    resp, err := p.client.Do(req)
    if err != nil {
        return nil, err
    }

    return resp, nil
}

func shouldSkipHeader(key string) bool {
    skipHeaders := map[string]bool{
        "Host":               true,
        "Connection":         true,
        "Keep-Alive":         true,
        "Proxy-Authenticate": true,
        "Proxy-Authorization": true,
        "Te":                 true,
        "Trailers":           true,
        "Transfer-Encoding":   true,
        "Upgrade":            true,
    }
    return skipHeaders[http.CanonicalHeaderKey(key)]
}

func updateForwardedHeaders(original, proxy *http.Request) {
    // X-Forwarded-For
    if clientxff := original.Header.Get("X-Forwarded-For"); clientxff != "" {
        proxy.Header.Set("X-Forwarded-For", clientxff+", "+original.RemoteAddr)
    } else {
        proxy.Header.Set("X-Forwarded-For", original.RemoteAddr)
    }

    // X-Forwarded-Proto
    if original.TLS != nil {
        proxy.Header.Set("X-Forwarded-Proto", "https")
    } else {
        proxy.Header.Set("X-Forwarded-Proto", "http")
    }

    // X-Forwarded-Host
    proxy.Header.Set("X-Forwarded-Host", original.Host)
}
```

**golang-patterns 技能提供的建议**:
- ✅ 使用 errors.Wrap 添加上下文
- ✅ 使用 context 传播超时和取消
- ✅ 正确处理响应体（避免泄漏）
- ✅ 使用结构体包装多个依赖
- ✅ 添加详细的日志和指标

---

### 步骤 5: 中间件

根据 `golang-patterns` 技能（函数类型、闭包）：

```go
// internal/middleware/ratelimit.go
package middleware

import (
    "net/http"
    "sync"
    "time"

    "github.com/redis/go-redis/v9"
    "golang.org/x/time/rate"
)

// RateLimiter 速率限制中间件
type RateLimiter struct {
    limiters map[string]*rate.Limiter
    mu      sync.RWMutex
    redis   *redis.Client
}

func NewRateLimiter(redis *redis.Client) *RateLimiter {
    return &RateLimiter{
        limiters: make(map[string]*rate.Limiter),
        redis:     redis,
    }
}

// RateLimit 返回速率限制中间件
func (rl *RateLimiter) RateLimit(requests int, window time.Duration) func(http.Handler) http.Handler {
    limiter := rate.NewLimiter(rate.Every(window/time.Duration(requests)), requests)

    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            // 获取客户端 IP
            clientIP := getClientIP(r)

            // 检查 Redis 中的速率限制（跨实例）
            if rl.redis != nil {
                allowed, err := rl.checkRedisRateLimit(clientIP, requests, window)
                if err != nil {
                    http.Error(w, "Internal server error", http.StatusInternalServerError)
                    return
                }
                if !allowed {
                    http.Error(w, "Rate limit exceeded", http.StatusTooManyRequests)
                    return
                }
            } else {
                // 检查本地速率限制（单实例）
                if !limiter.Allow() {
                    http.Error(w, "Rate limit exceeded", http.StatusTooManyRequests)
                    return
                }
            }

            next.ServeHTTP(w, r)
        })
    }
}

func (rl *RateLimiter) checkRedisRateLimit(
    clientIP string,
    requests int,
    window time.Duration,
) (bool, error) {
    key := fmt.Sprintf("ratelimit:%s", clientIP)
    windowSec := int64(window.Seconds())

    // 使用 Redis Lua 脚本保证原子性
    luaScript := `
        local current = redis.call('GET', KEYS[1])
        if current == false then
            redis.call('SET', KEYS[1], 1)
            redis.call('EXPIRE', KEYS[1], ARGV[1])
            return 1
        else
            if tonumber(current) < tonumber(ARGV[2]) then
                redis.call('INCR', KEYS[1])
                return 1
            else
                return 0
            end
        end
    `

    result, err := rl.redis.Eval(
        context.Background(),
        luaScript,
        []string{key},
        windowSec,
        requests,
    ).Result()
    if err != nil {
        return false, err
    }

    return result.(int64) == 1, nil
}

func getClientIP(r *http.Request) string {
    // 检查 X-Forwarded-For 头
    if xff := r.Header.Get("X-Forwarded-For"); xff != "" {
        // 取第一个 IP（如果有多个）
        if idx := strings.Index(xff, ","); idx != -1 {
            return strings.TrimSpace(xff[:idx])
        }
        return strings.TrimSpace(xff)
    }

    // 检查 X-Real-IP 头
    if xri := r.Header.Get("X-Real-IP"); xri != "" {
        return strings.TrimSpace(xri)
    }

    // 使用 RemoteAddr
    return strings.Split(r.RemoteAddr, ":")[0]
}
```

```go
// internal/middleware/recovery.go
package middleware

import (
    "net/http"
    "runtime/debug"

    "github.com/pkg/errors"
)

// Recovery 恢复中间件
func Recovery(logger *logger.Logger) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            defer func() {
                if err := recover(); err != nil {
                    logger.Error("Panic recovered",
                        "path", r.URL.Path,
                        "method", r.Method,
                        "error", err,
                        "stack", string(debug.Stack()),
                    )

                    http.Error(w, "Internal server error", http.StatusInternalServerError)
                }
            }()

            next.ServeHTTP(w, r)
        })
    }
}
```

**golang-patterns 技能提供的建议**:
- ✅ 使用闭包创建中间件
- ✅ 使用 defer 处理资源清理
- ✅ 使用 recover 捕获 panic
- ✅ 使用 sync.RWMutex 保护共享状态
- ✅ 使用 Redis 实现分布式速率限制

---

### 步骤 6: 使用 golang-testing 技能编写测试

```
用户: 为负载均衡器编写测试

ECC 自动激活: golang-testing 技能
```

根据 `golang-testing` 技能（表驱动测试、测试辅助函数）：

```go
// internal/service/loadbalancer_test.go
package service

import (
    "context"
    "testing"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
)

func TestRoundRobinLoadBalancer_Next(t *testing.T) {
    tests := []struct {
        name         string
        services     []model.Service
        serviceID    string
        expectedHosts []string
    }{
        {
            name: "should distribute requests evenly",
            services: []model.Service{
                {ID: "service1", Host: "host1", Port: 8080, Healthy: true},
                {ID: "service1", Host: "host2", Port: 8080, Healthy: true},
                {ID: "service1", Host: "host3", Port: 8080, Healthy: true},
            },
            serviceID:    "service1",
            expectedHosts: []string{"host1", "host2", "host3", "host1", "host2"},
        },
        {
            name: "should skip unhealthy instances",
            services: []model.Service{
                {ID: "service1", Host: "host1", Port: 8080, Healthy: true},
                {ID: "service1", Host: "host2", Port: 8080, Healthy: false},
                {ID: "service1", Host: "host3", Port: 8080, Healthy: true},
            },
            serviceID:    "service1",
            expectedHosts: []string{"host1", "host3", "host1", "host3"},
        },
        {
            name: "should return error when no healthy instances",
            services: []model.Service{
                {ID: "service1", Host: "host1", Port: 8080, Healthy: false},
                {ID: "service1", Host: "host2", Port: 8080, Healthy: false},
            },
            serviceID:    "service1",
            expectedHosts: nil,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Setup
            discovery := &MockDiscovery{services: tt.services}
            lb := NewRoundRobinLoadBalancer()
            lb.discovery = discovery

            ctx := context.Background()

            // Test
            for i, expectedHost := range tt.expectedHosts {
                service, err := lb.Next(ctx, tt.serviceID)

                if expectedHost == "" {
                    require.Error(t, err)
                    assert.Equal(t, ErrNoHealthyInstances, errors.Cause(err))
                } else {
                    require.NoError(t, err)
                    assert.NotNil(t, service)
                    assert.Equal(t, expectedHost, service.Host,
                        "iteration %d: expected host %s, got %s", i, expectedHost, service.Host)
                }
            }
        })
    }
}

// MockDiscovery 用于测试
type MockDiscovery struct {
    services []model.Service
}

func (m *MockDiscovery) GetServices(ctx context.Context) ([]model.Service, error) {
    return m.services, nil
}

// Benchmark for load balancer
func BenchmarkRoundRobinLoadBalancer_Next(b *testing.B) {
    services := []model.Service{
        {ID: "service1", Host: "host1", Port: 8080, Healthy: true},
        {ID: "service1", Host: "host2", Port: 8080, Healthy: true},
        {ID: "service1", Host: "host3", Port: 8080, Healthy: true},
    }

    discovery := &MockDiscovery{services: services}
    lb := NewRoundRobinLoadBalancer()
    lb.discovery = discovery

    ctx := context.Background()

    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        _, err := lb.Next(ctx, "service1")
        if err != nil {
            b.Fatal(err)
        }
    }
}

// Example usage
func ExampleRoundRobinLoadBalancer_Next() {
    services := []model.Service{
        {ID: "service1", Host: "host1", Port: 8080, Healthy: true},
        {ID: "service1", Host: "host2", Port: 8080, Healthy: true},
    }

    discovery := &MockDiscovery{services: services}
    lb := NewRoundRobinLoadBalancer()
    lb.discovery = discovery

    ctx := context.Background()

    service1, _ := lb.Next(ctx, "service1")
    fmt.Println(service1.Host) // Output: host1

    service2, _ := lb.Next(ctx, "service1")
    fmt.Println(service2.Host) // Output: host2
}
```

**golang-testing 技能提供的建议**:
- ✅ 使用表驱动测试（subtests）
- ✅ 使用 testify 提供丰富的断言
- ✅ 使用 Mock 对象隔离测试
- ✅ 编写基准测试
- ✅ 提供示例用法

---

### 步骤 7: Docker 和 Kubernetes 部署

根据 `docker-patterns` 技能：

```dockerfile
# Dockerfile
FROM golang:1.21-alpine AS builder

WORKDIR /build

# 复制依赖文件
COPY go.mod go.sum ./
RUN go mod download

# 复制源代码
COPY . .

# 构建
RUN CGO_ENABLED=0 GOOS=linux go build

# 最终镜像
FROM alpine:latest

RUN apk --no-cache add ca-certificates

WORKDIR /app

COPY --from=builder /build/api-gateway .

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/health || exit 1

CMD ["/api-gateway"]
```

```yaml
# deployments/k8s.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-gateway
  template:
    metadata:
      labels:
        app: api-gateway
    spec:
      containers:
      - name: gateway
        image: api-gateway:latest
        ports:
        - containerPort: 8080
        env:
        - name: CONFIG_PATH
          value: "/config/config.yaml"
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: api-gateway-secrets
              key: redis-url
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "500m"
        volumeMounts:
        - name: config
          mountPath: /config
          readOnly: true
      volumes:
      - name: config
        configMap:
          name: api-gateway-config

---
apiVersion: v1
kind: Service
metadata:
  name: api-gateway
spec:
  selector:
    app: api-gateway
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: api-gateway-config
data:
  config.yaml: |
    server:
      addr: ":8080"
      read_timeout: 30s
      write_timeout: 30s
      idle_timeout: 60s
    gateway:
      discovery_type: "consul"
      load_balancer_type: "round_robin"
      rate_limit_enabled: true
      rate_limit_requests: 100
      rate_limit_window: "1m"
    metrics:
      enabled: true
      port: 9090
      path: "/metrics"
    logging:
      level: "info"
      format: "json"
      output: "stdout"
```

---

## 测试和运行

```bash
# 运行所有测试
go test ./...

# 运行特定包的测试
go test ./internal/service/...

# 运行特定测试函数
go test -run TestRoundRobinLoadBalancer_Next ./internal/service/

# 运行基准测试
go test -bench=. ./internal/service/

# 生成覆盖率报告
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out -o coverage.html

# 使用 race detector 检测竞态条件
go test -race ./...

# 使用 golangci-lint 进行代码检查
golangci-lint run

# 使用静态分析
go vet ./...

# 构建
make build

# 运行
make run

# Docker 构建
docker build -t api-gateway:latest .

# Docker 运行
docker run -p 8080:8080 api-gateway:latest

# Kubernetes 部署
kubectl apply -f deployments/k8s.yaml
```

---

## 性能优化

根据 `golang-patterns` 技能（性能优化建议）：

```go
// 优化 1: 使用对象池减少内存分配
var requestPool = sync.Pool{
    New: func() interface{} {
        return new(http.Request)
    },
}

// 优化 2: 使用 bytes.Buffer 减少 string 拼接
func buildURL(base, path, query string) string {
    var buf bytes.Buffer
    buf.Grow(len(base) + len(path) + len(query) + 2)
    buf.WriteString(base)
    buf.WriteString(path)
    if query != "" {
        buf.WriteByte('?')
        buf.WriteString(query)
    }
    return buf.String()
}

// 优化 3: 使用 sync.Map 替代 map+mutex（适合高并发读）
var cache sync.Map

// 优化 4: 使用 channel 实现 worker pool
type workerPool struct {
    tasks chan func()
    wg    sync.WaitGroup
}

func newWorkerPool(workers int) *workerPool {
    p := &workerPool{
        tasks: make(chan func(), 100),
    }

    p.wg.Add(workers)
    for i := 0; i < workers; i++ {
        go p.worker()
    }

    return p
}

func (p *workerPool) worker() {
    defer p.wg.Done()
    for task := range p.tasks {
        task()
    }
}

func (p *workerPool) submit(task func()) {
    p.tasks <- task
}

// 优化 5: 使用 context 控制超时
func (p *ProxyService) ProxyRequestWithTimeout(
    ctx context.Context,
    req *http.Request,
    route model.Route,
    timeout time.Duration,
) (*http.Response, error) {
    ctx, cancel := context.WithTimeout(ctx, timeout)
    defer cancel()

    return p.ProxyRequest(ctx, req, route)
}
```

---

## 关键优势

| 传统 Go 开发 | 使用 ECC |
|-------------|---------|
| 手动设计项目结构 | golang-patterns 自动生成结构 |
| 查阅文档编写并发代码 | golang-patterns 提供并发模式 |
| 手动编写测试模板 | golang-testing 生成测试模板 |
| 手动配置 Docker | docker-patterns 生成配置 |
| 提交后人工审查 | go-reviewer 自动审查 |
| 多次迭代优化 | 一次完成高质量代码 |

---

## 扩展场景

### 添加服务网格集成

```
用户: 集成 Istio 服务网格

ECC 使用 skills:
- golang-patterns: 服务网格集成模式
- backend-patterns: API 网关最佳实践
```

### 添加 gRPC 支持

```
用户: 添加 gRPC 代理支持

ECC 使用 skills:
- golang-patterns: gRPC 客户端模式
- backend-patterns: 协议转换模式
```

### 添加自定义插件

```
用户: 添加请求/响应转换插件

ECC 使用 skills:
- golang-patterns: 插件系统模式
- backend-patterns: 中间件链模式
```
