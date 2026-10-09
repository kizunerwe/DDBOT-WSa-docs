# Lagrange 对接

[Lagrange](https://lagrangedev.github.io/Lagrange.Doc/) 是跨平台的 OneBot 实现，纯命令行/服务化运行，**适合 Linux 服务器**无桌面场景。

---

## 安装 Lagrange

参考 [Lagrange 官方文档](https://lagrangedev.github.io/Lagrange.Doc/v1/Lagrange.OneBot/) 下载 `Lagrange.OneBot` 并完成首次登录（扫码或 Keystore）。

---

## 配置反向 WS 连接 DDBOT

编辑 Lagrange 的 `appsettings.json`，把下面的对象加入 `Implementations` 数组（`Implementations` 支持配置多个连接；完整的 `appsettings.json` 还包含 `Account`、`Message`、`SignServerUrl` 等段，请以 Lagrange 生成的默认配置为准）：

```json
{
  "Implementations": [
    {
      "Type": "ReverseWebSocket",
      "Host": "127.0.0.1",
      "Port": 15630,
      "Suffix": "/ws",
      "ReconnectInterval": 5000,
      "HeartBeatInterval": 5000,
      "AccessToken": ""
    }
  ]
}
```

| 字段 | 值 | 说明 |
|------|----|------|
| `Type` | `ReverseWebSocket` | 反向 WS（Lagrange 主动连 DDBOT） |
| `Host` | `127.0.0.1` | DDBOT 所在机器 |
| `Port` | `15630` | DDBOT 监听端口 |
| `Suffix` | `/ws` | 路径，必须带 |
| `AccessToken` | 通常留空 | `master` / `next` 的 ws-server 不校验 token；`[next-dev]` 配置了 `websocket.token` 时填一致的值 |

!!! tip "DDBOT 侧配置"
    DDBOT 保持默认：

    ```yaml
    websocket:
      mode: ws-server
      ws-server: 0.0.0.0:15630   # master / next 默认监听全部网卡
    ```

    `master` / `next` 默认监听全部网卡，Lagrange 用 `ws://127.0.0.1:15630/ws` 即可连入；这两个分支的 ws-server **不校验 `token`**。

    `[next-dev]` 默认收紧为 `127.0.0.1:15630`：未配置 `token` 时只允许监听回环地址；若 Lagrange 在**其它机器或容器**里、需把 `ws-server` 改成它可达的地址，则**必须配置 `token`**，否则拒绝启动。

---

## 验证连接

1. 启动 DDBOT，确认 `WebSocket server started`
2. 启动 Lagrange，DDBOT 日志出现 `有新的ws连接了!!`（`[next-dev]` 为 `WebSocket client connected (ws-server mode)`）
3. 私聊 BOT 发 `/ping`，收到 `pong` 即正常

---

## Lagrange PMHQ

Lagrange PMHQ 是 Lagrange 的一个分支，DDBOT-WSa 同样兼容，配置方式与本页一致。

---

## 常见问题

### 连接被拒绝 / 401

- `master` / `next` 的 ws-server 不校验 token，token 不一致不会导致连不上；`[next-dev]` 才校验，此时检查 `AccessToken` 与 DDBOT 的 `websocket.token` 是否一致
- 检查 `Suffix` 是否为 `/ws`（缺省会连错路径）

### 消息发送失败

- Lagrange 对部分消息类型支持与 LLOneBot/NapCat 略有差异，遇到不支持的类型 DDBOT 会容错跳过
- 媒体相关见 [媒体与 FFmpeg](media.md)

---

## 参考

- Lagrange 官方文档：https://lagrangedev.github.io/Lagrange.Doc/
- Lagrange GitHub：https://github.com/LagrangeDev/Lagrange.Core
