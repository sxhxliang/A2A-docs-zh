# 远程智能体至客户端更新机制

<!-- TOC -->

- [远程智能体至客户端更新机制](#远程智能体至客户端更新机制)
  - [连接状态](#连接状态)
  - [断连状态](#断连状态)
  - [设置任务通知](#设置任务通知)
  - [智能体安全](#智能体安全)
  - [通知接收方安全](#通知接收方安全)
      - [非对称密钥](#非对称密钥)
      - [对称密钥](#对称密钥)
      - [OAuth认证](#oauth认证)
      - [持有者令牌](#持有者令牌)
  - [其他考量](#其他考量)
      - [重放攻击防护](#重放攻击防护)
      - [密钥轮换](#密钥轮换)

<!-- /TOC -->

部分任务可能耗时远超数秒，甚至需要分钟、小时或天数（如"将样品寄往佛罗里达州客户处并在送达时通知我"）。A2A智能体需支持长周期通信，包括连接与断连场景。

客户端可通过智能体名片检查是否支持流式传输和推送通知能力：
<pre>
{
  "name": "智能体名称",
  "description": "智能体描述"
  ...

  "capabilities": {
    <b>"streaming": true,</b>
    <b>"pushNotifications": false,</b>
    "stateTransitionHistory": false
  }
  ...
}
</pre>

智能体可通过以下方式获取任务执行更新：
1. **持久连接**：客户端通过HTTP+服务器推送事件(SSE)建立持久连接，智能体通过该连接发送任务更新
2. **推送通知**：智能体将完整任务对象发送至客户端指定的通知URL（类似Webhook机制）

## 连接状态
连接状态下，智能体通过任务消息相互更新。客户端与远程智能体可在同一连接上并发处理多任务。

客户端使用[任务/发送](/documentation.md#发送任务)更新当前任务，远程智能体通过[任务更新](/documentation.md#流式支持)（流式模式）或[任务对象](/documentation.md#获取任务)（非流式模式）响应。非流式模式下可采用合理间隔轮询。

断连后可通过[任务/重订阅](/documentation.md#重新订阅任务)恢复连接获取实时更新。

## 断连状态
A2A支持通过[推送通知配置](/documentation.md#推送通知)实现断连状态下的更新通知。企业环境中，智能体必须验证通知服务身份，进行服务认证，并提供与执行任务绑定的标识符。

通知服务应视为独立于客户端智能体的服务，通常不由客户端直接实现。该服务负责智能体认证授权，并将验证后的通知代理至适当端点（如消息队列、邮件系统等）。

简单场景下（如VPC内本地服务网格），客户端可自行开放端口作为通知服务。但企业级实现建议采用集中式服务，通过可信凭证认证远程智能体，并处理在线/离线场景（类似移动推送通知服务）。

## 设置任务通知
客户端需通过"tasks/pushNotification/set"RPC或在"tasks/send"/"tasks/sendSubscribe"的`pushNotification`参数中设置推送配置：

<pre>
// 推送通知配置结构
interface PushNotificationConfig {
  url: string;       // 通知接收URL
  token?: string;    // 任务/会话专属令牌
  authentication?: { // 认证配置
    schemes: string[]; 
    credentials?: string;
  }
}

// 带推送配置的任务请求示例
{
  "jsonrpc": "2.0",
  "method": "tasks/send",
  "params": {
    "pushNotification": {
      "url": "https://example.com/callback",
      "authentication": {
        "schemes": ["bearer"]
      }
    }
    // 其他参数...
  }
}
</pre>

## 智能体安全
智能体不应盲目信任客户端提供的通知URL，推荐实践包括：
1. 通过GET质询请求验证URL有效性（防止恶意客户端诱导DDOS攻击）
2. 要求通知服务使用预共享密钥对验证令牌进行签名

## 通知接收方安全
通知接收方应验证通知真实性，可采用：
#### 非对称密钥
- 采用ECDSA/RSA生成密钥对
- 通过JWKS协议动态提供公钥
- 使用JWT标准化签名字段

#### 对称密钥
- 双方共享密钥进行签名验证
- 同样可采用JWT生成签名令牌

#### OAuth认证
- 智能体从OAuth服务器获取令牌
- 通知服务向OAuth服务器验证令牌

#### 持有者令牌
- 由任一方生成静态令牌
- 需注意令牌泄露风险

## 其他考量
#### 重放攻击防护
- 在JWT中使用iat字段标记事件时间
- 拒绝超过5分钟的旧事件
- 将时间戳纳入签名计算

#### 密钥轮换
- 通过JWKS实现零停机轮换
- 通知接收方应能同时验证新旧密钥签名

（注：代码示例和接口定义保持原格式未翻译，因涉及技术术语和规范一致性）