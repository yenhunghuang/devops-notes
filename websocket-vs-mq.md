# WebSocket 與消息佇列技術筆記

## 一、WebSocket vs HTTP 核心比較

### 1. 連接特性
| 特性 | HTTP | WebSocket |
|------|------|-----------|
| 連接類型 | 短連接 | 長連接 |
| 通信方式 | 單向（客戶端發起） | 全雙工 |
| 連接開銷 | 每次請求都需建立 | 一次建立持續使用 |
| 資源消耗 | 較大 | 較小 |

### 2. 應用場景
#### HTTP 適用：
- RESTful API 調用
- 文件上傳下載
- 一次性數據請求
- 標準的 CRUD 操作

#### WebSocket 適用：
- 實時數據推送
- 在線聊天應用
- 遊戲實時通信
- 協同編輯
- 股票行情推送

## 二、常見面試問題與解答

### 1. WebSocket 如何處理斷線重連？
#### 解決方案：指數退避算法（Exponential Backoff）
```javascript
class WebSocketClient {
    constructor() {
        this.failedAttempts = 0;
        this.maxRetryInterval = 30000; // 30秒
    }

    connect() {
        try {
            // 連接邏輯
        } catch (error) {
            this.failedAttempts++;
            const retryInterval = Math.min(
                1000 * Math.pow(2, this.failedAttempts), 
                this.maxRetryInterval
            );
            setTimeout(() => this.connect(), retryInterval);
        }
    }
}
```

### 2. 如何確保 WebSocket 連接的可靠性？
#### 主要機制：
1. 心跳檢測
2. 消息確認
3. 狀態監控
4. 消息隊列

```javascript
class ReliableWebSocket {
    constructor() {
        this.heartbeatInterval = 30000;
        this.messageQueue = [];
        this.pendingMessages = new Map();
    }

    startHeartbeat() {
        setInterval(() => {
            if (this.ws.readyState === WebSocket.OPEN) {
                this.ws.send(JSON.stringify({
                    type: 'ping',
                    timestamp: Date.now()
                }));
            }
        }, this.heartbeatInterval);
    }
}
```

### 3. 什麼情況下選擇 HTTP 輪詢而非 WebSocket？
#### 適用場景：
1. 完整性要求高的數據傳輸
   - 市場完整列表
   - 客戶詳細信息
   
2. 安全性要求高的操作
   - 需要每次請求驗證
   - 需要請求加密
   - 需要審計日誌

```javascript
// 示例：安全性要求高的 HTTP 請求
async function secureFetch(url, data) {
    return await fetch(url, {
        headers: {
            'Authorization': `Bearer ${getToken()}`,
            'X-Transaction-ID': generateId()
        },
        body: encryptData(data)
    });
}
```

### 4. WebSocket 的擴展性問題如何解決？
#### 擴展策略：
1. 水平擴展
   - 負載均衡
   - Redis Pub/Sub 消息同步
   - 會話親和性

2. 垂直擴展
   - 連接池管理
   - 工作線程處理
   - 資源限制

```javascript
// 示例：集群管理
const cluster = require('cluster');
if (cluster.isMaster) {
    // 創建工作進程
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }
}
```

## 三、消息佇列（Message Queue）知識

### 1. RabbitMQ vs Kafka
| 特性 | RabbitMQ | Kafka |
|------|----------|-------|
| 適用場景 | 複雜路由、低延遲 | 大規模數據流、日誌 |
| 延遲 | ~100ms | ~200ms |
| 吞吐量 | 每秒數萬消息 | 每秒數十萬消息 |
| 特點 | 靈活的路由 | 高吞吐、可持久化 |

### 2. 使用場景
#### RabbitMQ：
- 精確的消息路由
- 複雜的消息分發
- 需要即時處理

#### Kafka：
- 日誌收集
- 流式處理
- 事件溯源

## 四、實際應用建議

### 1. 小規模應用
- 直接使用 WebSocket
- 注重實現簡單性
- 關注開發效率

### 2. 大規模應用
- 考慮消息佇列
- 注重系統可靠性
- 關注擴展性

### 3. 混合架構
```javascript
class DataService {
    constructor() {
        this.ws = new WebSocket('ws://...');  // 實時數據
        this.http = new HttpClient();         // 敏感操作
        this.mq = new MessageQueue();         // 異步任務
    }
}
```

## 五、監控與維護

### 1. 關鍵指標
- 連接數
- 消息延遲
- 錯誤率
- 資源使用

### 2. 日誌記錄
- 連接狀態變化
- 錯誤信息
- 性能指標
- 安全事件

### 3. 告警機制
- 連接異常
- 性能下降
- 資源耗盡
- 安全威脅