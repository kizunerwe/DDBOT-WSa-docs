# 连接 OneBot

DDBOT-WSa 不直接登录任何 IM 平台，而是通过 **OneBot 11 WebSocket 协议** 与一个实现端通信。本页讲清楚连接原理，具体实现端的配置见左侧子页面。

!!! info "OneBot 是通用协议，不限于 QQ"
    OneBot 是一套通用的 IM 机器人协议规范。任何实现了该规范的端（QQ、Telegram、KOOK 等，只要协议层兼容）理论上都能作为 DDBOT 的消息通道。下面列出的实现端是目前主流、经过验证的选择，但不代表全部。

---

## 正向 vs 反向 WebSocket

DDBOT 的 `application.yaml` 里 `websocket.mode` 控制连接方式，二选一：

### 正向 WebSocket（`ws-server`，默认）

DDBOT 在本地开一个 HTTP/WS 服务器，等实现端主动连过来。

```yaml
websocket:
  mode: ws-server
  ws-server: 0.0.0.0:15630   # 监听地址:端口
  token:                      # 可选，Access Token 校验
```

- 监听路径固定为 `/ws`，即完整地址 `ws://<ip>:15630/ws`
- 实现端那边配置「反向 WS」地址指向这里
- **新手推荐**：配置最简单

### 反向 WebSocket（`ws-reverse`）

DDBOT 主动去连实现端开的 WS 服务器。

```yaml
websocket:
  mode: ws-reverse
  ws-reverse: ws://localhost:3001   # 实现端的 WS 地址
  token:                              # 可选，与实现端一致
```

- 适合 DDBOT 在内网、实现端在公网，或反过来由实现端统管多连接的场景
- 默认地址 `ws://localhost:3001` 对应 LLOneBot 默认端口

!!! tip "怎么选"
    - 实现端和 DDBOT 在同一台机器：用默认的 `ws-server` 即可
    - 实现端已经在开 WS 服务器且不想改：用 `ws-reverse` 让 DDBOT 去连它
    - 两种方式功能完全等价，只是连接方向不同

---

## Access Token 校验

如果担心别人连到你的 BOT，可设置 `token`：

```yaml
websocket:
  mode: ws-server
  token: "your-secret-token"
```

- 正向模式：实现端连接时需在 HTTP Header 带 `Authorization: Bearer your-secret-token`
- 反向模式：DDBOT 连接实现端时会自动带上这个 Header

**token 必须两边一致**，否则连接会被拒绝。

---

## 连接成功的标志

DDBOT 日志出现：

```
有新的ws连接了!!
```

实现端那边也会显示已连接（具体文案因实现端而异）。

连接建立后，DDBOT 会延迟加载好友/群/群员信息（默认 3 秒，可配置 `reloadDelay`），加载完成后即可正常收发消息。

---

## 常见的 OneBot 实现端

下面列出几个常见的 OneBot 11 实现端，**本站不做特定推荐**，请根据自身环境（桌面/服务器、操作系统、是否已有云崽生态等）自行选择。各端配置教程见左侧子页面。

| 实现端 | 运行形态 | 文档 | 教程 |
|--------|---------|------|------|
| LLOneBot / LLBot | NTQQ 插件 | [LuckyLillia.com](https://LuckyLillia.com) | [对接 LLOneBot](llonebot.md) |
| NapCat | 跨平台 | [napneko.github.io](https://napneko.github.io/) | [对接 NapCat](napcat.md) |
| Lagrange | 跨平台，无桌面 | [Lagrange.Doc](https://lagrangedev.github.io/Lagrange.Doc/) | [对接 Lagrange](lagrange.md) |
| 云崽 + ws-plugin | 跨平台（基于云崽） | [ws-plugin](https://github.com/XasYer/ws-plugin) | [对接云崽](yunzai.md) |

此外还兼容 Lagrange PMHQ、go-cqhttp 等其它 OneBot 11 实现端，配置方法类似，可参照 Lagrange 一页。

!!! note "实现端项目变动频繁"
    各实现端可能更名、迁移仓库或调整配置格式（如 LLOneBot 已更名为 LLBot）。请以各项目官方文档为准，本页链接会尽量更新但不保证实时有效。

!!! note "实现端的选择是你的事"
    各实现端的稳定性、功能完整度、跟进 QQ 协议的速度各不相同，且会随时间变化。请以各项目官方文档为准，本站不对其可用性背书。

---

## 消息发送方式

DDBOT-WSa 从 v0.4.0 起**改用 OneBot array 消息格式**发送（早期版本用 CQ 码字符串）。

这意味着：

- 文本、图片、@、回复、戳一戳、视频、语音、文件等都能正常发送
- 媒体文件支持三种投递方式：**URL / Base64 / 本地路径**（由实现端处理，超时 120s）
- 大部分媒体由实现端直接处理，无需 DDBOT 侧额外配置

视频/语音/文件发送和 m3u8 视频推送需要 FFmpeg，见 [媒体与 FFmpeg](media.md)。

---

## 常见连接问题

### 连接不上

- 确认 DDBOT 已启动且日志显示 `WebSocket server started`
- 确认端口没被占用：`netstat -ano | findstr 15630`（Windows）/ `ss -tlnp | grep 15630`（Linux）
- 确认防火墙放行 15630（若实现端不在本机）
- 确认 `mode` 两边对应：DDBOT 用 `ws-server` 时，实现端必须用「反向 WS」连过来

### 连上但收不到消息

- 确认 BOT 已被拉进群，或已是好友
- 确认 `reloadDelay` 延迟加载已完成（看日志）
- 确认未被禁言（被禁言时 BOT 不会响应群命令）

### token 不匹配

日志报 `401` 或连接被立即断开：检查两边 `token` 是否完全一致，注意多余空格。

更多排障见 [常见问题](../../faq.md)。

---

## 下一步

选择你的实现端，按对应教程配置：

- [LLOneBot](llonebot.md)
- [NapCat](napcat.md)
- [Lagrange](lagrange.md)
- [云崽 ws-plugin](yunzai.md)
