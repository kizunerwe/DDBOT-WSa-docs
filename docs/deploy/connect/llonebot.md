# LLOneBot / LLBot 对接

!!! warning "项目已更名"
    原来的 **LLOneBot** 已更名为 **Lucky Lillia Bot（LLBot）**，新官网 `https://LuckyLillia.com`，新仓库 [LLOneBot/LuckyLilliaBot](https://github.com/LLOneBot/LuckyLilliaBot)。旧仓库 [LLOneBot/LLOneBot](https://github.com/LLOneBot/LLOneBot) 仍可访问但已转为新项目。

    本页仍按习惯称其为 LLOneBot，配置方式对更名后的 LLBot 同样适用。

LLOneBot 是基于 NTQQ 的 OneBot 11 实现，运行在 Windows 桌面 QQ 上，适合个人桌面用户。

---

## 前置准备

1. 一台 Windows 电脑
2. 安装 [NTQQ](https://im.qq.com/pcqq/index.shtml)（新版 QQ 桌面端）并登录 BOT 的 QQ 账号
3. 下载 LLBot / LLOneBot 最新版：[GitHub Releases](https://github.com/LLOneBot/LuckyLilliaBot/releases)

---

## 安装

1. 打开已登录的 NTQQ
2. 进入「设置 -> 关于 -> 插件管理」（或按 LLBot 文档说明的方式加载插件）
3. 重启 NTQQ 使插件生效

详细安装步骤请参考 [LLBot 官方文档](https://LuckyLillia.com)。

---

## 配置反向 WS 连接 DDBOT

LLBot 的配置文件在 NTQQ 数据目录的 `data` 文件夹下，**修改后会自动重载**，无需重启 QQ 和 LLBot。

在 OneBot 11 配置里，开启**反向 WebSocket** 并把地址指向 DDBOT：

```json
{
  "ob11": {
    "enable": true,
    "token": "",
    "wsReverseUrls": ["ws://127.0.0.1:15630/ws"],
    "enableWsReverse": true,
    "enableWs": true,
    "wsPort": 3001,
    "enableHttp": true,
    "httpPort": 3000
  }
}
```

关键项：

| 字段 | 值 | 说明 |
|------|----|------|
| `enableWsReverse` | `true` | 开启反向 WS（连向 DDBOT） |
| `wsReverseUrls` | `["ws://127.0.0.1:15630/ws"]` | DDBOT 的 WS 地址 |
| `token` | 与 DDBOT 一致 | 若 DDBOT 配了 `websocket.token`，这里填同样的值 |

!!! tip "DDBOT 侧配置"
    DDBOT 的 `application.yaml` 保持默认即可：

    ```yaml
    websocket:
      mode: ws-server
      ws-server: 0.0.0.0:15630
    ```

---

## 验证连接

1. 确保 DDBOT 已启动，日志显示 `WebSocket server started on ws://0.0.0.0:15630/ws`
2. LLBot 配置保存后，DDBOT 日志应出现 `有新的ws连接了!!`
3. 私聊 BOT 发送 `/ping`，收到 `pong` 即表示双向通信正常

---

## 常见问题

### 连接不上

- 确认 DDBOT 的 `mode` 是 `ws-server`（默认）
- 确认 `wsReverseUrls` 地址完全正确，包括 `/ws` 路径
- 确认端口 15630 未被占用

### 收不到消息

- 确认 NTQQ 已登录 BOT 账号
- 确认 LLBot 插件已启用
- 检查 LLBot 日志是否有报错

### 媒体发不出

视频/语音/文件发送需要 FFmpeg，见 [媒体与 FFmpeg](media.md)。

---

## 参考

- LLBot 官网：https://LuckyLillia.com
- LLBot GitHub：https://github.com/LLOneBot/LuckyLilliaBot
- 旧仓库（已更名）：https://github.com/LLOneBot/LLOneBot
