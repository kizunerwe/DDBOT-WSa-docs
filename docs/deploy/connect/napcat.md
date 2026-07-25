# NapCat 对接

[NapCat](https://napneko.github.io/) 是基于 QQNT 的跨平台 OneBot 11 实现，支持 Windows / Linux / Docker，适合服务器和桌面用户。

---

## 安装 NapCat

参考 [NapCat 官方文档](https://napneko.github.io/) 选择安装方式：

- **Windows**：下载壳框架并加载 NapCat
- **Linux**：使用官方一键脚本或 Docker
- **Docker**：`docker run` 方式部署

安装后登录 BOT 的 QQ 账号。

---

## 配置反向 WS 连接 DDBOT

在 NapCat 的网络配置中，添加一个**反向 WebSocket（Reverse WebSocket）**连接：

| 配置项 | 值 |
|--------|----|
| 类型 | 反向 WebSocket |
| 地址 | `ws://127.0.0.1:15630/ws` |
| Token | 与 DDBOT 的 `websocket.token` 一致（DDBOT 未配则留空） |

也可以直接编辑 NapCat 的配置文件（`onebot11.json` 或 WebUI 配置），添加反向 WS 项：

```json
{
  "network": {
    "websocketClients": [
      {
        "enable": true,
        "url": "ws://127.0.0.1:15630/ws",
        "token": ""
      }
    ]
  }
}
```

!!! tip "DDBOT 侧配置"
    DDBOT 保持默认正向模式：

    ```yaml
    websocket:
      mode: ws-server
      ws-server: 0.0.0.0:15630
    ```

---

## 验证连接

1. DDBOT 日志出现 `有新的ws连接了!!`
2. 私聊 BOT 发 `/ping`，收到 `pong` 即正常

---

## 常见问题

### NapCat 适配

NapCat 与 LLOneBot 同源，消息格式高度兼容。DDBOT-WSa 已适配新版 NapCat 的 `pokeRaw`、表情消息等。

### 连接频繁断开

- 检查 token 是否一致
- 检查 NapCat 是否正常运行、QQ 是否掉线
- 确认 NapCat 心跳间隔未设得过短

### 表情/图片识别异常

DDBOT 已支持区分常规图片和表情图片（NapCat），如仍有问题请到 [Issues](https://github.com/cnxysoft/DDBOT-WSa/issues) 反馈。

---

## 参考

- NapCat 官方文档：https://napneko.github.io/
- NapCat GitHub：https://github.com/NapNeko/NapCatQQ
