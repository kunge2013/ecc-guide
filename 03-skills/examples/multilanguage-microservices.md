# 使用场景：构建多语言微服务架构

这个示例展示如何结合使用 ECC 的多个技能和代理来构建一个包含多个技术栈的完整微服务架构。

## 场景描述

**项目**: 电商系统微服务架构

**技术栈**:
- 认证服务: Go (高性能 API 网关)
- 用户服务: Java Spring Boot (业务逻辑复杂）
- 订单服务: Python FastAPI (快速开发)
- 支付服务: Rust (安全性和性能要求高)
- 产品服务: Kotlin + Ktor (Android 团队维护)
- 搜索服务: Java Spring Boot (Elasticsearch 集成)
- 通知服务: Go (高并发场景)
- 前端: Next.js TypeScript

## 整体架构设计

```
                        [用户]
                           |
                      [Next.js 前端]
                           |
                +----------+----------+
                |                     |
           [认证服务]          [API 网关]
           (Go)               (Go/Chi)
                |                     |
                +----------+----------+
                           |
                    [服务发现/注册中心]
                           |
         +---------+---------+---------+---------+
         |         |         |         |         |
   [用户服务] [订单服务] [产品服务] [支付服务] [搜索服务]
  (Java)    (Python)   (Kotlin)   (Rust)    (Java)
         |         |         |         |         |
         +---------+---------+---------+---------+
                           |
                    [事件总线 / Kafka]
                           |
         +---------+---------+---------+---------+
         |         |         |         |         |
   [通知服务] [审计服务] [分析服务] [报表服务] [备份服务]
    (Go)      (Go)      (Python)   (Java)    (Go)
         |         |         |         |         |
         +---------+---------+---------+---------+
                           |
                   [数据库集群]
```

---

## 服务 1: Go 认证服务

**使用的技能**:
- `golang-patterns` - Go 编码模式
- `golang-testing` - Go 测试模式
- `backend-patterns` - 后端架构模式
- `security-review` - 安全审查

**关键功能**:
- JWT Token 生成和验证
- OAuth 2.0 社交登录
- 用户会话管理
- 权限验证
- 速率限制

**ECC 辅助开发流程**:

```
用户: 创建 Go 认证服务

ECC 检测到:
- 项目类型: Go
- 激活技能: golang-patterns, golang-testing, backend-patterns

ECC 使用技能建议:
1. 项目结构设计
2. 接口定义（服务抽象）
3. 错误处理模式
4. 并发安全模式
5. 测试模板生成

代码示例:
// pkg/auth/service.go
package auth

import (
    "context"
    "time"
    "github.com/golang-jwt/jwt/v5"
)

type Service interface {
    GenerateToken(ctx context.Context, userID string) (string, error)
    ValidateToken(ctx context.Context, token string) (*Claims, error)
    RefreshToken(ctx context.Context, token string) (string, error)
}

type JWTService struct {
    secretKey     []byte
    tokenDuration time.Duration
}

func (s *JWTService) GenerateToken(ctx context.Context, userID string) (string, error) {
    // 生成 JWT token
    now := time.Now()
    claims := jwt.MapClaims{
        "user_id": userID,
        "iat":      now.Unix(),
        "exp":      now.Add(s.tokenDuration).Unix(),
    }

    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString(s.secretKey)
}
```

**golang-patterns 技能提供的建议**:
- ✅ 使用接口定义契约
- ✅ 使用 context.Context 传播取消
- ✅ 使用结构体包装依赖
- ✅ 使用错误变量
- ✅ 使用 defer 清理资源

---

## 服务 2: Java 用户服务

**使用的技能**:
- `java-coding-standards` - Java 编码规范
- `springboot-patterns` - Spring Boot 模式
- `springboot-tdd` - Spring Boot TDD
- `backend-patterns` - 后端架构模式

**关键功能**:
- 用户 CRUD 操作
- 用户配置和偏好
- 用户历史记录
- 数据库访问
- 缓存管理

**ECC 辅助开发流程**:

```
用户: 创建 Spring Boot 用户服务

ECC 检测到:
- 项目类型: Java (pom.xml)
- 框架: Spring Boot
- 激活技能: java-coding-standards, springboot-patterns, springboot-tdd

ECC 使用技能建议:
1. 分层架构设计
2. 实体类定义
3. Repository 层
4. Service 层
5. Controller 层
6. 测试类生成

代码示例:
// User.java
@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_email", columnList = "email", unique = true),
    @Index(name = "idx_username", columnList = "username", unique = true)
})
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true, length = 100)
    private String email;

    @Column(nullable = false, unique = true, length = 50)
    private String username;

    @Column(nullable = false)
    private String hashedPassword;

    @Column(length = 100)
    private String fullName;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private UserStatus status;

    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column(name = "updated_at", nullable = false)
    private LocalDateTime updatedAt;
}

// UserService.java
@Service
@RequiredArgsConstructor
@Slf4j
public class UserService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    @Cacheable(value = "users", key = "#id")
    public UserDTO getUserById(Long id) {
        log.debug("Fetching user with id: {}", id);
        User user = userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found: " + id));
        return mapToDTO(user);
    }
}
```

**springboot-tdd 技能提供的建议**:
- ✅ 使用 JUnit 5 和 AssertJ
- ✅ 使用 @Mock 和 @InjectMocks
- ✅ 测试所有 Service 方法
- ✅ 测试 Repository 方法
- ✅ 验证 mock 调用
- ✅ 使用 Given-When-Then 模式

---

## 服务 3: Python 订单服务

**使用的技能**:
- `python-patterns` - Python 编码模式
- `python-testing` - Python 测试策略
- `backend-patterns` - 后端架构模式

**关键功能**:
- 订单创建和更新
- 订单状态管理
- 购物车同步
- 订单统计
- 事件发布

**ECC 辅助开发流程**:

```
用户: 创建 FastAPI 订单服务

ECC 检测到:
- 项目类型: Python
- 框架: FastAPI
- 激活技能: python-patterns, python-testing, backend-patterns

ECC 使用技能建议:
1. 项目结构设计
2. Pydantic schemas 定义
3. SQLAlchemy models
4. CRUD operations
5. 测试用例生成

代码示例:
# app/models/order.py
from typing import Optional
from datetime import datetime
from sqlalchemy import Column, Integer, String, Float, DateTime, Enum
from sqlalchemy.ext.declarative import declarative_base
import enum


class OrderStatus(enum.Enum):
    PENDING = "pending"
    PROCESSING = "processing"
    COMPLETED = "completed"
    CANCELLED = "cancelled"


Base = declarative_base()


class Order(Base):
    """订单模型"""
    __tablename__ = "orders"

    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, nullable=False, index=True)
    total_amount = Column(Float, nullable=False)
    status = Column(String(20), nullable=False, default="pending")
    created_at = Column(DateTime, nullable=False, default=datetime.utcnow)
    updated_at = Column(DateTime, nullable=False, onupdate=datetime.utcnow)

    __table_args__ = (
        Index('idx_user_status', 'user_id', 'status'),
    )


# app/schemas/order.py
from pydantic import BaseModel, Field, validator


class OrderCreate(BaseModel):
    """订单创建 Schema"""
    user_id: int = Field(..., description="用户 ID")
    items: list["OrderItemCreate"] = Field(..., min_items=1, description="订单项")

    @validator("total_amount", pre=True)
    def calculate_total(cls, v, values):
        """计算订单总金额"""
        items = values.get("items", [])
        total = sum(item.price * item.quantity for item in items)
        return total


class OrderItemCreate(BaseModel):
    """订单项创建 Schema"""
    product_id: int
    quantity: int = Field(..., gt=0, description="数量")
    price: float = Field(..., gt=0, description="单价")


# app/services/order_service.py
from typing import List
from fastapi import HTTPException
from sqlalchemy.orm import Session

from app.models.order import Order, OrderStatus
from app.schemas.order import OrderCreate, OrderDTO
from app.crud.order import create_order, get_order, get_user_orders
from app.events.publisher import publish_order_event


class OrderService:
    """订单服务"""

    def __init__(self, db: Session):
        self.db = db

    def create_order(self, order_data: OrderCreate) -> OrderDTO:
        """创建订单"""
        # 创建订单
        order = create_order(self.db, order_data)

        # 发布订单创建事件
        publish_order_event("order.created", {
            "order_id": order.id,
            "user_id": order.user_id,
            "total_amount": order.total_amount,
        })

        return self._map_to_dto(order)

    def get_order(self, order_id: int) -> OrderDTO:
        """获取订单"""
        order = get_order(self.db, order_id)
        if not order:
            raise HTTPException(status_code=404, detail="Order not found")
        return self._map_to_dto(order)

    def get_user_orders(self, user_id: int, skip: int = 0, limit: int = 100) -> List[OrderDTO]:
        """获取用户订单列表"""
        orders = get_user_orders(self.db, user_id, skip=skip, limit=limit)
        return [self._map_to_dto(order) for order in orders]
```

**python-patterns 技能提供的建议**:
- ✅ 使用 Pydantic 进行数据验证
- ✅ 使用类型注解
- ✅ 遵循 PEP 8 编码规范
- ✅ 使用异步编程（async/await）
- ✅ 使用 SQLAlchemy ORM

---

## 服务 4: Rust 支付服务

**使用的技能**:
- `rust-patterns` - Rust 编码模式
- `rust-testing` - Rust 测试模式
- `backend-patterns` - 后端架构模式
- `security-review` - 安全审查

**关键功能**:
- 支付处理（信用卡、PayPal、Alipay）
- 交易记录
- 退款处理
- 风控规则
- 支付验证

**ECC 辅助开发流程**:

```
用户: 创建 Rust 支付服务

ECC 检测到:
- 项目类型: Rust (Cargo.toml)
- 激活技能: rust-patterns, rust-testing, backend-patterns

ECC 使用技能建议:
1. 错误处理设计
2. Trait 定义
3. 结构体设计
4. 并发处理
5. 测试用例生成

代码示例:
// src/models/payment.rs
use serde::{Deserialize, Serialize};
use chrono::{DateTime, Utc};
use uuid::Uuid;


#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Payment {
    pub id: Uuid,
    pub user_id: u64,
    pub order_id: u64,
    pub amount: f64,
    pub currency: String,
    pub payment_method: PaymentMethod,
    pub status: PaymentStatus,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}


#[derive(Debug, Clone, Serialize, Deserialize)]
ver enum PaymentMethod {
    CreditCard,
    PayPal,
    Alipay,
    WeChat,
}


#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum PaymentStatus {
    Pending,
    Processing,
    Completed,
    Failed,
    Refunded,
}


// src/services/payment_service.rs
use anyhow::{Context, Result};
use crate::models::payment::{Payment, PaymentMethod, PaymentStatus};
use crate::repository::payment::PaymentRepository;
use crate::gateway::credit_card::{CreditCardGateway, CreditCardRequest};
use crate::gateway::paypal::{PayPalGateway, PayPaleRequest};


#[async_trait::async_trait]
pub trait PaymentGateway: Send + Sync {
    async fn charge(&self, request: PaymentRequest) -> Result<PaymentResponse>;
    async fn refund(&self, payment_id: Uuid) -> Result<PaymentResponse>;
}


pub struct PaymentService<R: PaymentRepository> {
    repository: R,
    credit_card_gateway: CreditCardGateway,
    paypal_gateway: PayPalGateway,
}


impl<R: PaymentRepository> PaymentService<R> {
    pub fn new(
        repository: R,
        credit_card_gateway: CreditCardGateway,
        paypal_gateway: PayPalGateway,
    ) -> Self {
        Self {
            repository,
            credit_card_gateway,
            paypal_gateway,
        }
    }

    pub async fn process_payment(
        &self,
        user_id: u64,
        order_id: u64,
        amount: f64,
        currency: String,
        method: PaymentMethod,
        card_details: Option<CreditCardDetails>,
        paypal_token: Option<String>,
    ) -> Result<Payment> {
        // 创建支付记录
        let payment = Payment {
            id: Uuid::new_v4(),
            user_id,
            order_id,
            amount,
            currency,
            payment_method: method.clone(),
            status: PaymentStatus::Pending,
            created_at: Utc::now(),
            updated_at: Utc::now(),
        };

        // 保存到数据库
        self.repository.create(&payment).await?;

        // 根据支付方法选择网关
        let response = match method {
            PaymentMethod::CreditCard => {
                let details = card_details
                    .context("Credit card details required")?;

                let request = PaymentRequest {
                    amount: payment.amount,
                    currency: payment.currency.clone(),
                    card_number: details.card_number,
                    card_expiry: details.card_expiry,
                    card_cvv: details.card_cvv,
                };

                self.credit_card_gateway.charge(request).await?
            }
            PaymentMethod::PayPal => {
                let token = paypal_token
                    .context("PayPal token required")?;

                let request = PaymentRequest {
                    amount: payment.amount,
                    currency: payment.currency.clone(),
                    token: token,
                };

                self.paypal_gateway.charge(request).await?
            }
            _ => {
                anyhow::bail!("Unsupported payment method")
            }
        };

        // 更新支付状态
        let payment = Payment {
            status: if response.success {
                PaymentStatus::Completed
            } else {
                PaymentStatus::Failed
            },
            ..payment
        };

        self.repository.update(&payment).await?;

        Ok(payment)
    }
}
```

**rust-patterns 技能提供的建议**:
- ✅ 使用 async_trait 定义异步 trait
- ✅ 使用 anyhow 统一错误处理
- ✅ 使用 ? 操作符简化错误传播
- ✅ 使用 Result<T, E> 表示可能失败的操作
- ✅ 使用所有权系统管理内存

---

## 服务 5: Kotlin 产品服务

**使用的技能**:
- `kotlin-patterns` - Kotlin 编码模式
- `kotlin-testing` - Kotlin 测试模式
- `kotlin-ktor-patterns` - Ktor 模式
- `backend-patterns` - 后端架构模式

**关键功能**:
- 产品 CRUD 操作
- 产品分类管理
- 产品搜索（集成搜索引擎）
- 产品推荐
- 库存管理

**ECC 辅助开发流程**:

```
用户: 创建 Ktor 产品服务

ECC 检测到:
- 项目类型: Kotlin (build.gradle.kts)
- 框架: Ktor
- 激活技能: kotlin-patterns, kotlin-ktor-patterns, kotlin-testing

ECC 使用技能建议:
1. 项目结构设计
2. Data class 定义
3. Repository 层
4. Service 层
5. Routing 定义
6. 测试用例生成

代码示例:
// Product.kt
import kotlinx.serialization.Serializable
import java.time.LocalDateTime


@Serializable
data class Product(
    val id: Long? = null,
    val name: String,
    val description: String,
    val price: Double,
    val category: String,
    val stock: Int,
    val imageUrl: String? = null,
    val tags: List<String> = emptyList(),
    val createdAt: LocalDateTime = LocalDateTime.now(),
    val updatedAt: LocalDateTime = LocalDateTime.now(),
) {
    init {
        require(name.isNotBlank()) { "Product name cannot be blank" }
        require(price > 0) { "Product price must be positive" }
        require(stock >= 0) { "Product stock cannot be negative" }
    }
}

// ProductService.kt
import org.koin.core.component.inject
import org.slf4j.LoggerFactory


class ProductService(
    private val productRepository: ProductRepository,
    private val searchService: SearchService,
) {
    private val logger = LoggerFactory.getLogger(ProductService::class.java)

    suspend fun getAllProducts(
        page: Int = 1,
        size: Int = 20,
        sort: String? = null,
    ): PagedResult<Product> {
        logger.debug("Fetching all products: page={}, size={}, sort={}", page, size, sort)

        val offset = (page - 1) * size
        val products = productRepository.findAll(offset, size, sort)
        val total = productRepository.count()

        return PagedResult(
            data = products,
            page = page,
            size = size,
            total = total,
            totalPages = (total + size - 1) / size,
        )
    }

    suspend fun getProductById(id: Long): Product {
        logger.debug("Fetching product by id: {}", id)

        return productRepository.findById(id)
            ?: throw ProductNotFoundException(id)
    }

    suspend fun searchProducts(
        query: String,
        category: String? = null,
        minPrice: Double? = null,
        maxPrice: Double? = null,
    ): List<Product> {
        logger.debug(
            "Searching products: query={}, category={}, price range=[{}, {}]",
            query, category, minPrice, maxPrice
        )

        return searchService.search(query, category, minPrice, maxPrice)
    }

    suspend fun createProduct(product: Product): Product {
        logger.info("Creating product: {}", product.name)

        val createdProduct = productRepository.create(product)

        // 索引到搜索引擎
        try {
            searchService.index(createdProduct)
        } catch (e: Exception) {
            logger.error("Failed to index product: {}", e.message)
            // 不抛出异常，允许产品创建成功
        }

        return createdProduct
    }
}
```

**kotlin-ktor-patterns 技能提供的建议**:
- ✅ 使用 data class 定义数据模型
- ✅ 使用 suspend functions 定义异步函数
- ✅ 使用 Koin 进行依赖注入
- ✅ 使用 Ktor routing 定义 API 端点
- ✅ 使用协程处理并发

---

## 跨服务通信

### 使用 Kafka 事件驱动

```
用户: 设计服务间的事件通信

ECC 使用 backend-patterns 技能:
1. 事件架构设计
2. 消息格式定义
3. 消费者实现
4. 生产者实现
5. 错误处理
```

**事件格式**:

```json
{
  "event_id": "550e8400-e29b-41d4-a716-446655440000",
  "event_type": "order.created",
  "event_version": "1.0",
  "source": "order-service",
  "timestamp": "2024-03-19T10:00:00Z",
  "data": {
    "order_id": 123,
    "user_id": 456,
    "total_amount": 99.99,
    "currency": "USD",
    "items": [
      {
        "product_id": 789,
        "quantity": 2,
        "price": 49.99
      }
    ]
  },
  "correlation_id": "req-550e8400-e29b-41d4-a716-446655440000",
  "causation_id": "req-550e8400-e29b-41d4-a716-446655440000"
}
```

**消费者实现**:

```go
// 订单事件消费者（Go）
package consumer

import (
    "encoding/json"
    "log"

    "github.com/segmentio/kafka-go"
)

type OrderEventConsumer struct {
    consumer *kafka.Consumer
    handler  EventHandler
}

func NewOrderEventConsumer(brokers []string, groupID string, handler EventHandler) (*OrderEventConsumer, error) {
    config := kafka.NewConfig()
    config.Group.Rebalance.Strategy = kafka.StickyAssignStrategy
    config.Consumer.Offsets.Initial = sarama.OffsetNewest

    consumer, err := kafka.NewConsumer(brokers, groupID, config)
    if err != nil {
        return nil, err
    }

    // 订阅订单主题
    if err := consumer.SubscribeTopics([]string{"orders"}, nil); err != nil {
        return nil, err
    }

    return &OrderEventConsumer{
        consumer: consumer,
        handler:  handler,
    }, nil
}

func (c *OrderEventConsumer) Consume() {
    for {
        msg, err := c.consumer.ReadMessage(-1)
        if err != nil {
            log.Printf("Consumer error: %v\n", err)
            continue
        }

        // 解析事件
        var event OrderEvent
        if err := json.Unmarshal(msg.Value, &event); err != nil {
            log log.Printf("Failed to unmarshal event: %v\n", err)
            continue
        }

        // 处理事件
        if err := c.handler.Handle(&event); err != nil {
            log.Printf("Failed to handle event: %v\n", err)
        } else {
            // 提交偏移量
            c.consumer.CommitOffsets(map[string][]kafka.TopicPartition{
                *msg.TopicPartition: {{Offset: *msg.Offset}},
            }, "")
        }
    }
}
```

---

## 统一配置和部署

### Kubernetes 部署配置

```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ecommerce
---
# config-map.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: common-config
  namespace: ecommerce
data:
  LOG_LEVEL: "info"
  ENVIRONMENT: "production"
  KAFKA_BOOTSTRAP_SERVERS: "kafka:9092"
  REDIS_URL: "redis://redis:6379"
```

### Docker Compose 开发环境

```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  # API Gateway (Go)
  api-gateway:
    build: ./services/api-gateway
    ports:
      - "8080:8080"
    environment:
      - KAFKA_BROKERS=kafka:9092
      - REDIS_URL=redis://redis:6379
      - CONSUL_HOST=consul
    depends_on:
      - kafka
      - redis
      - consul

  # Auth Service (Go)
  auth-service:
    build: ./services/auth-service
    ports:
      - "8081:8081"
    environment:
      - DATABASE_URL=postgresql://user:pass@auth-db:5432/auth
      - REDIS_URL=redis://redis:6379
      - KAFKA_BROKERS=kafka:9092
    depends_on:
      - auth-db
      - redis
      - kafka

  # User Service (Java)
  user-service:
    build: ./services/user-service
    ports:
      - "8082:8082"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://user:pass@user-db:5432/users
      - KAFKA_BOOTSTRAP_SERVERS=kafka:9092
    depends_on:
      - user-db
      - kafka

  # Order Service (Python)
  order-service:
    build: ./services/order-service
    ports:
      - "8083:8083"
    environment:
      - DATABASE_URL=postgresql://user:pass@order-db:5432/orders
      - KAFKA_BOOTSTRAP_SERVERS=kafka:9092
      - SEARCH_SERVICE_URL=http://search-service:8084
    depends_on:
      - order-db
      - kafka
      - search-service

  # Product Service (Kotlin)
  product-service:
    build: ./services/product-service
    ports:
      - "8085:8085"
    environment:
      - DATABASE_URL=jdbc:postgresql://user:pass@product-db:5432/products
      - SEARCH_SERVICE_URL=http://search-service:8084
      - KAFKA_BOOTSTRAP_SERVERS=kafka:9092
    depends_on:
      - product-db
      - kafka
      - search-service

  # Payment Service (Rust)
  payment-service:
    build: ./services/payment-service
    ports:
      - "8086:8086"
    environment:
      - DATABASE_URL=postgresql://user:pass@payment-db:5432/payments
      - KAFKA_BOOTSTRAP_SERVERS=kafka:9092
      - STRIPE_API_KEY=${STRIPE_API_KEY}
      - PAYPAL_API_KEY=${PAYPAL_API_KEY}
    depends_on:
      - payment-db
      - kafka

  # Search Service (Java)
  search-service:
    build: ./services/search-service
    ports:
      - "8084:8084"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://user:pass@search-db:5432/search
      - ELASTICSEARCH_HOST=elasticsearch:9200
    depends_on:
      - search-db
      - elasticsearch

  # Notification Service (Go)
  notification-service:
    build: ./services/notification-service
    ports:
      - "8087:8087"
    environment:
      - KAFKA_BROKERS=kafka:9092
      - SMTP_HOST=${SMTP_HOST}
      - SMTP_PORT=${SMTP_PORT}
    depends_on:
      - kafka

  # Infrastructure
  kafka:
    image: confluentinc/cp-kafka:7.4.0
    ports:
      - "9092:9092"
    environment:
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
    depends_on:
      - zookeeper

  zookeeper:
    image: confluentinc/cp-zookeeper:7.4.0
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  consul:
    image: consul:latest
    ports:
      - "8500:8500"

  elasticsearch:
    image: elasticsearch:8.5.0
    ports:
      - "9200:9200"
    environment:
      discovery.type: single-node
      ES_JAVA_OPTS: "-Xms512m -Xmx512m"

  auth-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: auth
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    ports:
      - "5432:5432"

  user-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: users
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    ports:
      - "5433:5433"

  order-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: orders
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    ports:
      - "5434:5434"

  product-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: products
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    ports:
      - "5435:5435"

  search-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: search
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    ports:
      - "5436:5436"

  payment-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: payments
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    ports:
      - "5437:5437"
```

---

## 测试策略

### 集成测试

```
用户: 编写跨服务的集成测试

ECC 使用 e2e-runner 代理:
1. 生成端到端测试
2. 使用 Docker Compose 启动所有服务
3. 验证完整用户流程
4. 清理测试环境
```

**测试用例**:

```typescript
// e2e/ecommerce-flow.spec.ts
import { test, expect } from '@playwright/test';

test.describe('电商系统完整流程', () => {
  test('应该完成完整的购买流程', async ({ page }) => {
    // 1. 注册用户
    await page.goto('/register');
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'SecurePass123!');
    await page.fill('[name="confirmPassword"]', 'SecurePass123!');
    await page.click('button[type="submit"]');
    await expect(page.locator('.success-message')).toContainText('注册成功');

    // 2. 登录
    await page.goto('/login');
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'SecurePass123!');
    await page.click('button[type="submit"]');
    await expect(page).toHaveURL('/dashboard');

    // 3. 浏览商品
    await page.goto('/products');
    await expect(page.locator('.product-card')).toHaveCount(10);

    // 4. 添加商品到购物车
    await page.click('.product-card:first-child button:has-text("加入购物车")');
    await expect(page.locator('.cart-count')).toContainText('1');

    // 5. 结账
    await page.click('a:has-text("购物车")');
    await page.click('button:has-text("结账")');

    // 6. 填写收货地址
    await page.fill('[name="fullName"]', 'Test User');
    await page.fill('[name="address"]', '123 Main St');
    await page.fill('[name="city"]', 'New York');
    await page.fill('[name="state"]', 'NY');
    await page.fill('[name="zipCode"]', '10001');

    // 7. 支付（测试模式）
    await page.selectOption('[name="paymentMethod"]', 'test');
    await page.click('button:has-text("支付")');

    // 8. 验证订单创建成功
    await expect(page.locator('.order-success')).toBeVisible();
    await expect(page.locator('.order-number')).toBeVisible();
  });
});
```

---

## 监控和日志

### 统一监控

```
用户: 添加监控和日志

ECC 使用 skills:
- backend-patterns: 可观测性模式
- golang-patterns: Go 监控模式
- springboot-patterns: Spring Boot 监控模式
```

**Prometheus 指标**:

```go
// metrics/metrics.go
package metrics

import (
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
)

var (
    // HTTP 请求指标
    httpRequestsTotal = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total number of HTTP requests",
        },
        []string{"method", "endpoint", "status"},
    )

    httpRequestDuration = promauto.NewHistogramVec(
        prometheus.HistogramOpts{
            Name: "http_request_duration_seconds",
            Help: "HTTP request duration in seconds",
            Buckets: []float64{.005, .01, .025, .05, .1, .25, .5, 1, 2.5, 5, 10},
        },
        []string{"method", "endpoint"},
    )

    // 业务指标
    orderCreatedTotal = promauto.NewCounter(
        prometheus.CounterOpts{
            Name: "order_created_total",
            Help: "Total number of orders created",
        },
    )

    paymentProcessedTotal = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Name: "payment_processed_total",
            Help: "Total number of payments processed",
        },
        []string{"status", "method"},
    )

    paymentAmount = promauto.NewHistogramVec(
        prometheus.HistogramOpts{
            Name: "payment_amount_dollars",
            Help: "Payment amount in dollars",
            Buckets: []float64{10, 25, 50, 100, 250, 500, 1000, 2500, 5000, 10000},
        },
        []string{"currency", "method"},
    )
)

func init() {
    prometheus.MustRegister(httpRequestsTotal)
    prometheus.MustRegister(httpRequestDuration)
    prometheus.MustRegister(orderCreatedTotal)
    prometheus.MustRegister(paymentProcessedTotal)
    prometheus.MustRegister(paymentAmount)
}
```

---

## 关键优势

| 传统微服务开发 | 使用 ECC |
|----------------|---------|
| 每个服务独立设计 | ECC 提供统一的架构指导 |
| 技术栈学习曲线陡 | 技能自动激活，减少学习成本 |
| 跨服务通信复杂 | backend-patterns 提供统一模式 |
| 测试覆盖率低 | e2e-runner 自动生成测试 |
| 代码审查分散 | code-reviewer 统一审查标准 |
| 部署配置繁琐 | docker-patterns 自动生成配置 |

---

## 最佳实践

1. **选择合适的技术栈**
   - Go: 高性能、API 网关、高并发
   - Java: 企业级应用、复杂业务逻辑
   - Python: 快速开发、原型设计
   - Rust: 性能关键、安全要求高
   - Kotlin: JVM 生态、现代化开发

2. **统一通信协议**
   - 使用 Kafka 进行异步通信
   - 使用 REST/gRPC 进行同步通信
   - 使用统一的事件格式

3. **标准化错误处理**
   - 每个服务使用一致的错误格式
   - 提供有意义的错误消息
   - 记录完整的错误上下文

4. **实现可观测性**
   - 统一的日志格式（JSON）
   - 标准化的指标（Prometheus）
   - 分布式追踪（OpenTelemetry）

5. **自动化测试**
   - 单元测试：每个服务独立测试
   - 集成测试：服务间交互测试
   - E2E 测试：完整用户流程测试

---

## 扩展场景

### 添加新的支付方式

```
用户: 添加 Apple Pay 和 Google Pay

ECC 使用 skills:
- rust-patterns: Rust 模式扩展
- security-review: 安全审查
- backend-patterns: 支付模式
```

### 添加 AI 推荐

```
用户: 添加基于 AI 的商品推荐

ECC 使用 skills:
- python-patterns: Python ML 集成
- claude-api: Claude API 集成
- backend-patterns: 推荐系统模式
```

### 添加数据分析

```
用户: 添加实时数据分析

ECC 使用 skills:
- python-patterns: Python 数据处理
- backend-patterns: 数据管道模式
```
