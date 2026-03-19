# 使用场景：使用 Rust 构建高性能数据处理管道

这个示例展示如何结合使用 ECC 的 Rust 技能和代理来构建一个安全、高性能的数据处理管道。

## 场景描述

**任务**: 构建一个实时数据处理管道，处理来自多个数据源的流式数据

**功能要求**:
- 从 Kafka 消费消息
- 并行处理数据
- 使用 Rayon 进行并行计算
- 将结果写入 Elasticsearch
- 错误处理和重试机制
- 优雅关闭
- 性能监控
- 内存管理
- 零成本抽象

**技术栈**: Rust 1.70+ + Kafka + Elasticsearch + Tokio + Rayon

## ECC 辅助开发流程

### 步骤 1: 项目初始化

```
用户: 创建一个新的 Rust 数据处理管道项目

ECC 检测到:
- 项目类型: Rust (Cargo.toml)
- 激活技能: rust-patterns, rust-testing
```

根据 `rust-patterns` 技能的建议：

```toml
# Cargo.toml
[package]
name = "data-pipeline"
version = "0.1.0"
edition = "2021"

[dependencies]
# 异步运行时
tokio = { version = "1.35", features = ["full"] }
tokio-util = "0.7"

# Kafka 客户端
rdkafka = { version = "0.36", features = ["cmake-build", "ssl-vendored"] }

# Elasticsearch 客户端
elasticsearch = "8.5"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"

# 错误处理
anyhow = "1.0"
thiserror = "1.0"

# 日志
tracing = "serde:0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter", "json"] }
tracing-opentelemetry = { version = "0.22", features = ["reqwest-client"] }

# 性能监控
metrics = "0.21"

# 并行处理
rayon = "1.8"

# 配置
config = { version = "0.14", features = ["toml"] }

# 序列化
bincode = "1.3"

# 时间处理
chrono = { version = "0.4", features = ["serde"] }

# UUID
uuid = { version = "1.6", features = ["v4", "serde"] }

[dev-dependencies]
tokio-test = "0.4"
criterion = "0.5"
mockall = "0.11"

[[bin]]
name = "pipeline"
path = "src/main.rs"

[[bin]]
name = "producer"
path = "src/producer.rs"
```

**rust-patterns 技能提供的建议**:
- ✅ 使用 workspace 管理多个 binary
- ✅ 使用 feature flags 控制依赖
- ✅ 使用 edition = "2021" 获取最新特性
- ✅ 使用 `[dev-dependencies]` 分离开发和生产依赖

---

### 步骤 2: 数据模型定义

根据 `rust-patterns` 技能（所有权、借用、序列化）：

```rust
// src/models/mod.rs
pub mod event;
pub mod processed;
pub mod error;
pub mod metrics;

use serde::{Deserialize, Serialize};
use chrono::{DateTime, Utc};
use uuid::Uuid;

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Event {
    pub id: Uuid,
    pub source: String,
    pub event_type: String,
    pub timestamp: DateTime<Utc>,
    pub data: serde_json::Value,
    pub metadata: EventMetadata,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct EventMetadata {
    pub version: u32,
    pub source_ip: Option<String>,
    pub user_id: Option<String>,
    pub tags: Vec<String>,
}

impl Event {
    pub fn new(source: String, event_type: String, data: serde_json::Value) -> Self {
        Self {
            id: Uuid::new_v4(),
            source,
            event_type,
            timestamp: Utc::now(),
            data,
            metadata: EventMetadata {
                version: 1,
                source_ip: None,
                user_id: None,
                tags: Vec::new(),
            },
        }
    }
}
```

```rust
// src/models/processed.rs
use super::event::Event;
use anyhow::Result;
use rayon::prelude::*;

#[derive(Debug, Clone, serde::Serialize)]
pub struct ProcessedEvent {
    pub id: String,
    pub event_type: String,
    pub processed_at: DateTime<Utc>,
    pub processing_duration_ms: u64,
    pub result: ProcessedResult,
    pub errors: Vec<String>,
}

#[derive(Debug, Clone, serde::Serialize)]
pub enum ProcessedResult {
    Success { record_id: String },
    PartialSuccess { record_id: String, warnings: Vec<String> },
    Failure { error: String },
}

// 并行处理多个事件
pub fn process_events_parallel(events: Vec<Event>) -> Vec<ProcessedEvent> {
    events
        .into_par_iter()
        .map(process_single_event)
        .collect()
}

// 处理单个事件
fn process_single_event(event: Event) -> ProcessedEvent {
    let start = std::time::Instant::now();

    match process_event_logic(&event) {
        Ok(result) => {
            let duration = start.elapsed().as_millis() as u64;
            ProcessedEvent {
                id: event.id.to_string(),
                event_type: event.event_type.clone(),
                processed_at: Utc::now(),
                processing_duration_ms: duration,
                result,
                errors: Vec::new(),
            }
        }
        Err(e) => {
            let duration = start.elapsed().as_millis() as u64;
            ProcessedEvent {
                id: event.id.to_string(),
                event_type: event.event_type.clone(),
                processed_at: Utc::now(),
                processing_duration_ms: duration,
                result: ProcessedResult::Failure {
                    error: e.to_string(),
                },
                errors: vec![e.to_string()],
            }
        }
    }
}

fn process_event_logic(event: &Event) -> anyhow::Result<ProcessedResult> {
    // 实现处理逻辑
    match event.event_type.as_str() {
        "user_signup" => process_user_signup(event),
        "order_created" => process_order_created(event),
        "payment_completed" => process_payment_completed(event),
        _ => Ok(ProcessedResult::Success {
            record_id: event.id.to_string(),
        }),
    }
}

fn process_user_signup(event: &Event) -> anyhow::Result<ProcessedResult> {
    // 验证和转换用户数据
    let user_data = validate_user_data(&event.data)?;

    // 生成记录 ID
    let record_id = format!("user_{}", event.id);

    // 转换为 Elasticsearch 文档
    let _doc = serde_json::to_value(&user_data)?;

    Ok(ProcessedResult::Success { record_id })
}

fn validate_user_data(data: &serde_json::Value) -> anyhow::Result<UserData> {
    let user_data: UserSignupData = serde_json::from_value(data.clone())?;

    if user_data.email.is_empty() {
        anyhow::bail!("Email is required");
    }

    if user_data.username.len() < 3 {
        anyhow::bail!("Username too short");
    }

    Ok(UserData {
        email: user_data.email,
        username: user_data.username,
        created_at: Utc::now(),
    })
}
```

**rust-patterns 技能提供的建议**:
- ✅ 使用 derive 自动实现 trait
- ✅ 使用 Result<T, E> 进行错误处理
- ✅ 使用 into_par_iter() 实现并行处理
- ✅ 避免不必要的克隆（使用引用）
- ✅ 使用 ? 操作符简化错误传播

---

### 步骤 3: Kafka 消费者

根据 `rust-patterns` 技能（异步编程、错误处理）：

```rust
// src/kafka/consumer.rs
use anyhow::{Context, Result};
use rdkafka::config::ClientConfig;
use rdkafka::consumer::{BaseConsumer, StreamConsumer};
use rdkafka::message::{BorrowedMessage, Message};
use rdkafka::Message as KafkaMessage;
use tokio::sync::mpsc;
use tracing::{error, info, instrument, span};

use super::models::Event;

pub struct KafkaConsumer {
    consumer: StreamConsumer<BaseConsumer>,
}

impl KafkaConsumer {
    pub fn new(config: &KafkaConfig) -> Result<Self> {
        let mut client_config = ClientConfig::new();

        // 设置 Kafka 配置
        client_config.set("group.id", &config.group_id)?;
        client_config.set("bootstrap.servers", &config.brokers)?;
        client_config.set("enable.auto.commit", "false")?;
        client_config.set("auto.offset.reset", "earliest")?;

        // 配置 SSL（如果需要）
        if let Some(ssl) = &config.ssl {
            client_config.set("security.protocol", "ssl")?;
            client_config.set("ssl.ca.location", &ssl.ca_location)?;
        }

        // 创建消费者
        let consumer: StreamConsumer<BaseConsumer> = client_config.create()?;

        // 订阅主题
        let topics: Vec<&str> = config.topics.iter().map(|s| s.as_str()).collect();
        consumer.subscribe(&topics)?;

        Ok(Self { consumer })
    }

    pub async fn run(
        &mut self,
        event_sender: mpsc::Sender<Event>,
        shutdown: tokio::sync::broadcast::Receiver<()>,
    ) -> Result<()> {
        info!("Starting Kafka consumer");

        loop {
            tokio::select! {
                // 检查关闭信号
                _ = shutdown.recv() => {
                    info!("Received shutdown signal");
                    break;
                }

                // 消费消息
                message_result = self.consumer.recv(), if !shutdown.is_empty() => {
                    match message_result {
                        Ok(message) => {
                            if let Some(event) = self.process_message(message) {
                                if let Err(e) = event_sender.send(event).await {
                                    error!("Failed to send event to channel: {}", e);
                                    break;
                                }
                            }
                        }
                        Err(e) => {
                            error!("Kafka consumer error: {}", e);
                            // 可以在这里添加重试逻辑
                            continue;
                        }
                    }
                }
            }
        }

        Ok(())
    }

    fn process_message(&self, message: BorrowedMessage) -> Option<Event> {
        let _span = span!(tracing::Level::INFO, "process_kafka_message");

        // 解析消息
        let payload = message.payload_view::<[u8]>()?;

        // 反序列化为 JSON
        let json_str = std::str::from_utf8(payload).ok()?;
        let json_value: serde_json::Value = serde_json::from_str(json_str).ok()?;

        // 转换为 Event
        let event: Event = serde_json::from_value(json_value).ok()?;

        Some(event)
    }
}

#[derive(Debug, Clone)]
pub struct KafkaConfig {
    pub brokers: String,
    pub group_id: String,
    pub topics: Vec<String>,
    pub ssl: Option<SslConfig>,
}

#[derive(Debug, Clone)]
pub struct SslConfig {
    pub ca_location: String,
}
```

**rust-patterns 技能提供的建议**:
- ✅ 使用 tokio::select! 处理多个异步事件
- ✅ 使用 instrument! 添加 tracing span
- ✅ 使用 Option 表示可能失败的操作
- ✅ 使用 Result 表示必须成功的操作
- ✅ 使用 Context 添加错误上下文

---

### 步骤 4: Elasticsearch 客户端

根据 `rust-patterns` 技能（async/await、资源管理）：

```rust
// src/elasticsearch/client.rs
use anyhow::Result;
use elasticsearch::{
    Elasticsearch,
    http::{transport::Transport, Url},
    SearchPartsBuilder,
};
use serde_json::json;
use tokio::sync::mpsc;
use tracing::{debug, error, info};

use super::models::processed::ProcessedEvent;

pub struct ElasticsearchClient {
    client: Elasticsearch,
}

impl ElasticsearchClient {
    pub async fn new(url: &str) -> Result<Self> {
        let transport = Transport::single(Url::parse(url)?)?;
        let client = Elasticsearch::new(transport);

        Ok(Self { client })
    }

    pub async fn index_event(&self, event: &ProcessedEvent) -> Result<()> {
        let response = self
            .client
            .index(elasticsearch::Index::Parts::Index(json!["data-pipeline")))
            .refresh(elasticsearch::params::Refresh::False)
            .body(json!(event))
            .send()
            .await?;

        if response.status_code().as_u16() != 201 {
            error!("Failed to index event: {:?}", response);
            anyhow::bail!("Failed to index event");
        }

        debug!("Successfully indexed event: {}", event.id);
        Ok(())
    }

    pub async fn bulk_index_events(&self, events: &[ProcessedEvent]) -> Result<()> {
        let mut body = Vec::new();

        for event in events {
            // 添加 index 操作
            body.push(json!({
                "index": {
                    "_index": "data-pipeline"
                }
            }));

            // 添加文档
            body.push(json!(event));
        }

        let response = self
            .client
            .bulk(elasticsearch::BulkPartsBuilder::new(body))
            .refresh(elasticsearch::params::Refresh::False)
            .send()
            .await?;

        if response.status_code().as_u16() != 200 {
            error!("Failed to bulk index events: {:?}", response);
            anyhow::bail!("Failed to bulk index events");
        }

        debug!("Successfully indexed {} events", events.len());
        Ok(())
    }

    pub async fn run(
        &self,
        mut event_receiver: mpsc::Receiver<ProcessedEvent>,
        batch_size: usize,
        batch_timeout_ms: u64,
    ) -> Result<()> {
        info!("Starting Elasticsearch writer");

        let mut batch = Vec::new();
        let mut last_batch_time = std::time::Instant::now();

        loop {
            tokio::select! {
                // 接收事件
                event_opt = event_receiver.recv() => {
                    match event_opt {
                        Some(event) => {
                            batch.push(event);

                            // 检查是否达到批处理大小
                            if batch.len() >= batch_size {
                                if let Err(e) = self.bulk_index_events(&batch).await {
                                    error!("Failed to index batch: {}", e);
                                    // 可以将失败的事件发送到死信队列
                                }
                                batch.clear();
                                last_batch_time = std::time::Instant::now();
                            }
                        }
                        None => {
                            info!("Event channel closed");
                            break;
                        }
                    }
                }

                // 检查批处理超时
                _ = tokio::time::sleep(tokio::time::Duration::from_millis(batch_timeout_ms)) => {
                    if !batch.is_empty() && last_batch_time.elapsed() > tokio::time::Duration::from_millis(batch_timeout_ms) {
                        if let Err(e) = self.bulk_index_events(&batch).await {
                            error!("Failed to index timed-out batch: {}", e);
                        }
                        batch.clear();
                        last_batch_time = std::time::Instant::now();
                    }
                }
            }
        }

        // 处理剩余的事件
        if !batch.is_empty() {
            info!("Indexing remaining {} events", batch.len());
            self.bulk_index_events(&batch).await?;
        }

        Ok(())
    }
}
```

---

### 步骤 5: 主应用程序

根据 `rust-patterns` 技能（结构体、trait、生命周期）：

```rust
// src/main.rs
mod kafka;
mod models;
mod elasticsearch;
mod config;

use anyhow::Result;
use tokio::sync::{broadcast, mpsc};
use tracing::{info, error};
use tracing_subscriber::{EnvFilter, fmt};

use config::load_config;

#[tokio::main]
async fn main() -> Result<()> {
    // 初始化日志
    init_tracing();

    // 加载配置
    let config = load_config("config.toml")?;
    info!("Loaded configuration");

    // 创建关闭通道
    let (shutdown_tx, _) = broadcast::channel(1);

    // 创建 Kafka 消费者
    let mut kafka_consumer = kafka::KafkaConsumer::new(&config.kafka)?;

    // 创建事件通道
    let (event_sender, event_receiver) = mpsc::channel(1000);
    let (processed_sender, processed_receiver) = mpsc::channel(1000);

    // 创建 Elasticsearch 客户端
    let es_client = elasticsearch::ElasticsearchClient::new(&config.elasticsearch.url).await?;

    // 克隆关闭发送者
    let shutdown_tx_kafka = shutdown_tx.clone();
    let shutdown_tx_processor = shutdown_tx.clone();
    let shutdown_tx_writer = shutdown_tx.clone();

    // 启动 Kafka 消费者任务
    let kafka_task = tokio::spawn(async move {
        if let Err(e) = kafka_consumer.run(event_sender, shutdown_tx_kafka.subscribe()).await {
            error!("Kafka consumer error: {}", e);
        }
    });

    // 启动数据处理任务
    let processor_task = tokio::spawn(async move {
        let mut receiver = event_receiver;
        let mut shutdown = shutdown_tx_processor.subscribe();

        loop {
            tokio::select! {
                _ = shutdown.recv() => {
                    info!("Processor received shutdown");
                    break;
                }

                event_opt = receiver.recv() => {
                    match event_opt {
                        Some(event) => {
                            // 并行处理事件
                            let processed = models::processed::process_single_event(event);

                            if let Err(e) = processed_sender.send(processed).await {
                                error!("Failed to send processed event: {}", e);
                                break;
                            }
                        }
                        None => {
                            info!("Event channel closed");
                            break;
                        }
                    }
                }
            }
        }
    });

    // 启动 Elasticsearch 写入任务
    let writer_task = tokio::spawn(async move {
        let mut shutdown = shutdown_tx_writer.subscribe();

        loop {
            tokio::select! {
                _ = shutdown.recv() => {
                    info!("Writer received shutdown");
                    break;
                }

                // 等待一段时间后退出
                _ = tokio::time::sleep(tokio::time::Duration::from_secs(1)) => {
                    // 检查是否应该退出
                    if shutdown.is_empty() && processed_receiver.is_empty() {
                        break;
                    }
                }
            }
        }

        if let Err(e) = es_client.run(processed_receiver, 100, 5000).await {
            error!("Elasticsearch writer error: {}", e);
        }
    });

    // 等待 Ctrl+C
    tokio::signal::ctrl_c()
        .await
        .expect("Failed to install Ctrl+C handler");

    info!("Received Ctrl+C, shutting down...");

    // 发送关闭信号
    let _ = shutdown_tx.send(());

    // 等待任务完成
    tokio::try_join!(kafka_task, processor_task, writer_task)?;

    info!("Shutdown complete");
    Ok(())
}

fn init_tracing() {
    let env_filter = EnvFilter::from_default_env();

    tracing_subscriber::fmt()
        .json()
        .with_env_filter(env_filter)
        .with_target(false)
        .with_current_span(false)
        .init();
}
```

---

### 步骤 6: 使用 rust-testing 技能编写测试

```
用户: 为数据处理逻辑编写测试

ECC 自动激活: rust-testing 技能
```

根据 `rust-testing` 技能（单元测试、属性测试、Mock）：

```rust
// src/models/processed_test.rs
use super::processed::*;
use super::event::*;

#[cfg(test)]
mod tests {
    use super::*;
    use chrono::Utc;
    use serde_json::json;

    #[test]
    fn test_process_user_signup_success() {
        let event = Event::new(
            "api".to_string(),
            "user_signup".to_string(),
            json!({
                "email": "test@example.com",
                "username": "testuser"
            }),
        );

        let processed = process_single_event(event);

        assert_eq!(processed.errors.len(), 0);
        assert!(!processed.id.is_empty());
        assert_eq!(processed.event_type, "user_signup");
    }

    #[test]
    fn test_process_user_signup_invalid_email() {
        let event = Event::new(
            "api".to_string(),
            "user_signup".to_string(),
            json!({
                "email": "",
                "username": "testuser"
            }),
        );

        let processed = process_single_event(event);

        assert!(!processed.errors.is_empty());
        assert_eq!(processed.errors[0], "Email is required");
    }

    #[test]
    fn test_process_events_parallel() {
        let events = vec![
            Event::new("api".to_string(), "user_signup".to_string(), json!({"email": "user1@example.com", "username": "user1"})),
            Event::new("api".to_string(), "user_signup".to_string(), json!({"email": "user2@example.com", "username": "user2"})),
            Event::new("api".to_string(), "user_signup".to_string(), json!({"email": "user3@example.com", "username": "user3"})),
        ];

        let results = process_events_parallel(events);

        assert_eq!(results.len(), 3);
        assert!(results.iter().all(|r| r.errors.is_empty()));
    }

    #[test]
    fn test_processing_performance() {
        let events: Vec<Event> = (0..100)
            .map(|i| {
                Event::new(
                    "api".to_string(),
                    "user_signup".to_string(),
                    json!({
                        "email": format!("user{}@example.com", i),
                        "username": format!("user{}", i)
                    }),
                )
            })
            .collect();

        let start = std::time::Instant::now();
        let results = process_events_parallel(events);
        let duration = start.elapsed();

        assert_eq!(results.len(), 100);
        assert!(duration.as_millis() < 1000, "Should process 100 events in < 1 second");
    }

    // 属性测试
    use quickcheck::{Arbitrary, Gen, QuickCheck, TestResult};

    impl Arbitrary for Event {
        fn arbitrary<G: Gen>(g: &mut G) -> Self {
            let event_type = if bool::arbitrary(g) {
                "user_signup".to_string()
            } else {
                "order_created".to_string()
            };

            Event::new(
                "test".to_string(),
                event_type,
                json!({
                    "email": "test@example.com",
                    "username": "test"
                }),
            )
        }
    }

    #[test]
    fn prop_process_event_preserves_id(event: Event) -> bool {
        let processed = process_single_event(event.clone());
        processed.id == event.id.to_string()
    }

    #[test]
    fn prop_process_events_parallel_length_preserved(events: Vec<Event>) -> bool {
        let results = process_events_parallel(events.clone());
        results.len() == events.len()
    }

    #[test]
    fn run_quickcheck() {
        QuickCheck::new()
            .tests(100)
            .gen(Gen::new(Seed::fixed(0x5dbece5d)))
            .quickcheck(prop_process_event_preserves_id as fn(Event) -> bool);
    }
}
```

---

## 性能优化

根据 `rust-patterns` 技能（零成本抽象、内存管理）：

```rust
// 优化 1: 使用对象池减少内存分配
use std::sync::Arc;

struct EventPool {
    events: Vec<Arc<Event>>,
}

impl EventPool {
    fn new(capacity: usize) -> Self {
        let events = (0..capacity)
            .map(|_| Arc::new(Event::default()))
            .collect();
        Self { events }
    }

    fn get_event(&mut self) -> Arc<Event> {
        self.events.pop().unwrap_or_else(|| {
            Arc::new(Event::default())
        })
    }

    fn return_event(&mut self, event: Arc<Event>) {
        self.events.push(event);
    }
}

// 优化 2: 使用零成本抽象
trait EventProcessor {
    fn process(&self, event: &Event) -> ProcessedEvent;
}

struct UserSignupProcessor;

impl EventProcessor for UserSignupProcessor {
    fn process(&self, event: &Event) -> ProcessedEvent {
        // 处理逻辑
        unimplemented!()
    }
}

struct OrderCreatedProcessor;

impl EventProcessor for OrderCreatedProcessor {
    fn process(&self, event: &Event) -> ProcessedEvent {
        // 处理逻辑
        unimplemented!()
    }
}

// 运行时多态但零成本抽象
fn get_processor(event_type: &str) -> &'static dyn EventProcessor {
    match event_type {
        "user_signup" => &UserSignupProcessor,
        "order_created" => &OrderCreatedProcessor,
        _ => panic!("Unknown event type"),
    }
}

// 优化 3: 使用 SIMD 加速
use std::simd::*;

fn validate_emails_batch(emails: &[String]) -> Vec<bool> {
    emails.iter()
        .map(|email| {
            // 可以使用 SIMD 加速字符串验证
            email.contains('@')
        })
        .collect()
}
```

---

## 运行和测试

```bash
# 构建项目
cargo build --release

# 运行主程序
cargo run --bin pipeline

# 运行生产者（测试用）
cargo run --bin producer

# 运行所有测试
cargo test

# 运行特定测试
cargo test test_process_user_signup_success

# 运行测试并显示输出
cargo test -- --nocapture

# 运行基准测试
cargo bench

# 生成文档
cargo doc --open

# 检查代码
cargo clippy

# 格式化代码
cargo fmt

# 检查格式
cargo fmt --check
```

---

## Docker 部署

```dockerfile
# Dockerfile
FROM rust:1.70 as builder

WORKDIR /app

# 复制依赖文件
COPY Cargo.toml Cargo.lock ./
COPY src ./src

# 构建
RUN cargo build --release

# 运行时镜像
FROM debian:bullseye-slim

RUN apt-get update && \
    apt-get install -y ca-certificates && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY --from=builder /app/target/release/pipeline /app/pipeline
COPY --from=builder /app/target/release/producer /app/producer

EXPOSE 9090

CMD ["./pipeline"]
```

---

## 关键优势

| 传统数据处理 | 使用 Rust |
|-------------|----------|
| 内存安全问题多 | Rust 的所有权系统保证内存安全 |
| 并发错误难以调试 | Rust 的类型系统防止数据竞争 |
| 性能瓶颈难以优化 | Rayon 和 SIMD 提供高性能并行处理 |
| 错误处理不一致 | Result<T, E> 统一错误处理 |
| 运行时错误多 | 编译时捕获大多数错误 |

---

## 扩展场景

### 添加监控和追踪

```
用户: 添加 OpenTelemetry 追踪

ECC 使用 skills:
- rust-patterns: 追踪集成模式
- backend-patterns: 可观测性最佳实践
```

### 添加死信队列

```
用户: 添加失败事件重试机制

ECC 使用 skills:
- rust-patterns: 错误处理模式
- backend-patterns: 可靠性模式
```

### 添加数据验证

```
用户: 添加严格的数据验证

ECC 使用 skills:
- rust-patterns: 类型系统利用
- backend-patterns: 数据验证模式
```
