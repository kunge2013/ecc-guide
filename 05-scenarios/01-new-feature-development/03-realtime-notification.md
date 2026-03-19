# 使用场景 3: 实时通知系统

## 需求描述

实现现实时通知系统，包括：
- WebSocket 推送
- 多客户端同步
- 消息持久化
- 离线消息
- 通知类型过滤

## 技术要求

- 使用 WebSocket 协议
- 支持多客户端连接
- 消息持久化到 Redis
- 离线时缓存消息
- 支持按类型过滤通知

## 完整实现流程

### 步骤 1: 架构设计

```bash
Agent(planner, """
设计实时通知系统架构：

需求：
- WebSocket 推送
- 多客户端同步
- 消息持久化
- 离线消息
- 通知类型过滤

技术栈：Go + Gin + WebSocket + Redis

请设计：
1. WebSocket 服务器架构
2. 消息存储方案
3. 客户端管理
4. 消息广播机制
5. 离线消息处理
""")
```

### 步骤 2: 核心实现

```go
// websocket_server.go
package main

import (
	"encoding/json"
	"log"
	"net/http"
	"sync"
	"time"

	"github.com/gin-gonic/gin"
	"github.com/gorilla/websocket"
	"github.com/go-redis/redis/v8"
)

// Notification 结构
type Notification struct {
	ID        string    `json:"id"`
	Type      string    `json:"type"`
	UserID    string    `json:"user_id"`
	Title     string    `json:"title"`
	Message   string    `json:"message"`
	Data      any       `json:"data,omitempty"`
	Timestamp time.Time `json:"timestamp"`
	Read      bool      `json:"read"`
}

// Client 结构
type Client struct {
	ID     string
	UserID string
	Conn   *websocket.Conn
	Send   chan Notification
}

// Hub 管理所有客户端
type Hub struct {
	// 已连接的客户端
	clients map[string]*Client

	// 用户 ID 到客户端 ID 的映射
	userClients map[string][]string

	// 注册和注销
	register   chan *Client
	unregister chan *Client

	// 广播消息
	broadcast chan Notification

	// 私有消息
	private chan *MessageRequest

	mu sync.RWMutex
}

type MessageRequest struct {
	UserID string
	Msg    Notification
}

// 创建 Hub
func NewHub() *Hub {
	return &Hub{
		clients:     make(map[string]*Client),
		userClients: make(map[string][]string),
		register:    make(chan *Client),
		unregister:  make(chan *Client),
		broadcast:   make(chan Notification, 256),
		private:     make(chan *MessageRequest, 256),
	}
}

// 运行 Hub
func (h *Hub) Run() {
	for {
		select {
		case client := <-h.register:
			h.mu.Lock()
			h.clients[client.ID] = client
			h.userClients[client.UserID] = append(h.userClients[client.UserID], client.ID)

			// 发送离线消息
			go h.sendOfflineMessages(client)

			h.mu.Unlock()
			log.Printf("客户端已连接: %s (用户: %s)", client.ID, client.UserID)

		case client := <-h.unregister:
			h.mu.Lock()
			delete(h.clients, client.ID)

			// 从用户客户端列表中移除
			userClients := h.userClients[client.UserID]
			for i, id := range userClients {
				if id == client.ID {
					h.userClients[client.UserID] = append(userClients[:i], userClients[i+1:]...)
					break
				}
			}

			close(client.Send)
			h.mu.Unlock()
			log.Printf("客户端已断开: %s", client.ID)

		case notification := <-h.broadcast:
			h.mu.RLock()
			for _, client := range h.clients {
				// 根据用户过滤
				if notification.UserID == "" || client.UserID == notification.UserID {
					select {
					case client.Send <- notification:
					default:
						// 客户端缓冲区满，关闭连接
						close(client.Send)
						delete(h.clients, client.ID)
					}
				}
			}
			h.mu.RUnlock()

		case req := <-h.private:
			h.mu.RLock()
			// 发送给特定用户的所有客户端
			if clients, exists := h.userClients[req.UserID]; exists {
				for _, clientID := range clients {
					if client, exists := h.clients[clientID]; exists {
						select {
						case client.Send <- req.Msg:
						default:
							// 缓冲区满，存储到离线消息
							go storeOfflineMessage(req.UserID, req.Msg)
						}
					}
				}
			}
			h.mu.RUnlock()
		}
	}
}

// 存储离线消息
func storeOfflineMessage(userID string, notification Notification) {
	// 实现存储到 Redis
}

// 发送离线消息
func (h *Hub) sendOfflineMessages(client *Client) {
	// 从 Redis 获取离线消息
	// 发送给客户端
	// 标记为已读
}

// 处理 WebSocket 连接
func (h *Hub) HandleWebSocket(c *gin.Context) {
	// 升级 HTTP 连接到 WebSocket
	upgrader := websocket.Upgrader{
		CheckOrigin: func(r *http.Request) bool {
			return true // 生产环境应该验证来源
		},
	}

	conn, err := upgrader.Upgrade(c.Writer, c.Request, nil)
	if err != nil {
		log.Printf("WebSocket 升级失败: %v", err)
		return
	}

	// 获取用户 ID
	userID := c.Query("user_id")
	if userID == "" {
		conn.Close()
		return
	}

	// 创建客户端
	clientID := generateClientID()
	client := &Client{
		ID:     clientID,
		UserID: userID,
		Conn:   conn,
		Send:   make(chan Notification, 256),
	}

	// 注册客户端
	h.register <- client

	// 启动写入协程
	go client.writePump()

	// 读取消息
	client.readPump(h)
}

// 写入消息到 WebSocket
func (c *Client) writePump() {
	defer c.Conn.Close()

	for notification := range c.Send {
		err := c.Conn.WriteJSON(notification)
		if err != nil {
			log.Printf("写入消息失败: %v", err)
			break
		}
	}
}

// 从 WebSocket 读取消息
func (c *Client) readPump(h *Hub) {
	defer func() {
		h.unregister <- c
	}()

	c.Conn.SetReadLimit(512)

	for {
		_, message, err := c.Conn.ReadMessage()
		if err != nil {
			if websocket.IsUnexpectedCloseError(err, websocket.CloseGoingAway, websocket.CloseAbnormalClosure) {
				log.Printf("读取消息失败: %v", err)
			}
			break
		}

		// 处理客户端消息（如心跳）
		var msg map[string]any
		if err := json.Unmarshal(message, &msg); err == nil {
			if msg["type"] == "ping" {
				// 发送 pong 响应
				c.Send <- Notification{
					Type:    "pong",
					UserID:  c.UserID,
				}
			}
		}
	}
}

// 发送通知给特定用户
func (h *Hub) SendNotification(userID string, notification Notification) {
	h.private <- &MessageRequest{
		UserID: userID,
		Msg:    notification,
	}
}

// 广播通知给所有用户
func (h *Hub) BroadcastNotification(notification Notification) {
	h.broadcast <- notification
}

func generateClientID() string {
	return "client-" + time.Now().Format("20060102150405.999999999") + "-" + randomString(8)
}

func randomString(n int) string {
	// 实现随机字符串生成
	return "random"
}

func main() {
	// 创建 Hub
	hub := NewHub()
	go hub.Run()

	// 创建 Gin 路由
	r := gin.Default()

	// WebSocket 端点
	r.GET("/ws", hub.HandleWebSocket)

	// HTTP API 端点
	r.POST("/api/notify", func(c *gin.Context) {
		var notification Notification
		if err := c.ShouldBindJSON(&notification); err != nil {
			c.JSON(400, gin.H{"error": err.Error()})
			return
		}

		// 设置时间戳
		notification.Timestamp = time.Now()

		// 发送通知
		hub.SendNotification(notification.UserID, notification)

		c.JSON(200, gin.H{"success": true})
	})

	r.POST("/api/broadcast", func(c *gin.Context) {
		var notification Notification
		if err := c.ShouldBindJSON(&notification); err != nil {
			c.JSON(400, gin.H{"error": err.Error()})
			return
		}

		notification.Timestamp = time.Now()

		// 广播通知
		hub.BroadcastNotification(notification)

		c.JSON(200, gin.H{"success": true})
	})

	// 启动服务器
	r.Run(":8080")
}
```

### 步骤 3: 客户端实现

```javascript
// notification-client.js
class NotificationClient {
  constructor(userId, wsUrl = 'ws://localhost:8080/ws') {
    this.userId = userId;
    this.wsUrl = `${wsUrl}?user_id=${userId}`;
    this.ws = null;
    this.reconnectAttempts = 0;
    this.maxReconnectAttempts = 5;
    this.reconnectDelay = 1000;
    this.listeners = new Map();
  }

  connect() {
    this.ws = new WebSocket(this.wsUrl);

    this.ws.onopen = () => {
      console.log('WebSocket 已连接');
      this.reconnectAttempts = 0;
      this.startHeartbeat();
    };

    this.ws.onmessage = (event) => {
      const notification = JSON.parse(event.data);
      this.notifyListeners(notification);
    };

    this.ws.onerror = (error) => {
      console.error('WebSocket 错误:', error);
    };

    this.ws.onclose = () => {
      console.log('WebSocket 已关闭');
      this.stopHeartbeat();
      this.reconnect();
    };
  }

  reconnect() {
    if (this.reconnectAttempts < this.maxReconnectAttempts) {
      this.reconnectAttempts++;
      const delay = this.reconnectDelay * Math.pow(2, this.reconnectAttempts - 1);

      console.log(`尝试重连 (${this.reconnectAttempts}/${this.maxReconnectAttempts})，延迟 ${delay}ms`);

      setTimeout(() => {
        this.connect();
      }, delay);
    } else {
      console.error('达到最大重连次数，放弃重连');
    }
  }

  startHeartbeat() {
    this.heartbeatInterval = setInterval(() => {
      if (this.ws.readyState === WebSocket.OPEN) {
        this.ws.send(JSON.stringify({ type: 'ping' }));
      }
    }, 30000); // 30 秒心跳
  }

  stopHeartbeat() {
    if (this.heartbeatInterval) {
      clearInterval(this.heartbeatInterval);
    }
  }

  on(type, callback) {
    if (!this.listeners.has(type)) {
      this.listeners.set(type, []);
    }
    this.listeners.get(type).push(callback);
  }

  off(type, callback) {
    if (this.listeners.has(type)) {
      const callbacks = this.listeners.get(type);
      const index = callbacks.indexOf(callback);
      if (index > -1) {
        callbacks.splice(index, 1);
      }
    }
  }

  notifyListeners(notification) {
    const callbacks = this.listeners.get(notification.type) || [];
    callbacks.forEach(callback => callback(notification));

    // 通知所有监听器
    const allCallbacks = this.listeners.get('*') || [];
    allCallbacks.forEach(callback => callback(notification));
  }

  disconnect() {
    this.stopHeartbeat();
    if (this.ws) {
      this.ws.close();
    }
  }
}

// 使用示例
const client = new NotificationClient('user123');
client.connect();

// 监听特定类型的通知
client.on('alert', (notification) => {
  console.log('收到警报:', notification.title);
  showNotification(notification);
});

// 监听所有通知
client.on('*', (notification) => {
  console.log('收到通知:', notification);
});

// 清理
client.disconnect();
```

## 关键学习点

### 1. WebSocket 连接管理

- 连接注册和注销
- 用户到客户端的映射
- 连接池管理

### 2. 消息广播

- 向所有客户端广播
- 向特定用户发送私有消息
- 按用户过滤消息

### 3. 离线消息处理

- 连接时发送离线消息
- 断开时存储消息
- 消息持久化

### 4. 心跳机制

- 定期发送心跳
- 检测连接状态
- 自动重连机制

---

**提示：** WebSocket 实时通信展示了如何处理长连接、消息广播和离线消息。
