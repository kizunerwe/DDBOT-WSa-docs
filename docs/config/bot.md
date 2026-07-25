# bot 与连接配置

本页讲 `bot`、`websocket`、`reloadDelay`、`debug`、`message-marker`、`qq-logs` 这几组配置。

---

## `bot` - 基本设置

```yaml
bot:
  account:                   # QQ 号，不填则扫码登录
  password:                  # QQ 密码
  commandPrefix: "/"         # 命令前缀，默认 /
  onDisconnected: "exit"     # 历史遗留，见下方说明
  onJoinGroup:
    rename: "【bot】"         # 进群自动改名
  sendFailureReminder:       # 发送失败提醒
    enable: false
    times: 3
  offlineQueue:              # 离线缓存
    enable: false
    expire: 30m
```

| 字段 | 类型 | 默认 | 说明 |
|------|------|------|------|
| `account` | int | 空 | BOT 的 QQ 号。留空则首次扫码登录（仅推荐测试） |
| `password` | string | 空 | BOT 的 QQ 密码。与 account 配合使用 |
| `commandPrefix` | string | `/` | 命令触发前缀。可改成其他符号 |
| `onDisconnected` | string | `exit` | 历史遗留配置，DDBOT-WSa 已不处理协议层重连，一般无需关心，见 [升级与维护](../deploy/upgrade.md#进程保活) |
| `onJoinGroup.rename` | string | `【bot】` | 进群后自动改群名片，留空则不改。最长 60 字符 |

### `sendFailureReminder` - 发送失败提醒

```yaml
bot:
  sendFailureReminder:
    enable: false
    times: 3
```

消息发送失败达到 `times` 次后，触发模板 `notify.bot.send_failed.tmpl` 通知管理员。模板可用变量：`message`、`target_id`、`target_type`、`target_name`、`times`。

需要自行创建该模板，见 [事件模板](../template/trigger-tmpl.md)。

### `offlineQueue` - 离线缓存

```yaml
bot:
  offlineQueue:
    enable: false
    expire: 30m
```

BOT 离线期间暂存要发送的消息，上线后补发。

| 字段 | 说明 |
|------|------|
| `enable` | 是否启用 |
| `expire` | 离线消息有效期，超期的消息上线后不再补发 |

!!! warning "离线缓存期间不能重启 DDBOT"
    缓存存在内存里，重启 DDBOT 会清空缓存。仅适用于「短暂掉线后自动重连」的场景。

---

## `websocket` - OneBot 连接

这是 DDBOT-WSa 区别于纯血 DDBOT 的核心配置。

```yaml
websocket:
  mode: ws-server            # ws-server / ws-reverse
  token:                     # Access Token（可选）
  ws-server: 0.0.0.0:15630   # 正向模式监听地址
  ws-reverse: ws://localhost:3001  # 反向模式连接地址
```

| 字段 | 说明 |
|------|------|
| `mode` | `ws-server`（正向，DDBOT 开端口等连）或 `ws-reverse`（反向，DDBOT 主动连） |
| `token` | Access Token，与实现端一致。留空则不校验 |
| `ws-server` | 正向模式监听地址，路径固定 `/ws`。默认 `0.0.0.0:15630` |
| `ws-reverse` | 反向模式连接地址，默认 `ws://localhost:3001`（LLOneBot 默认端口） |

!!! tip "新手默认即可"
    保持默认 `ws-server` 模式，让实现端反向连到 `ws://127.0.0.1:15630/ws`。

    详见 [连接 OneBot](../deploy/connect/index.md)。

### 正向模式（默认）

DDBOT 在本地开 HTTP/WS 服务器，等实现端连过来：

```yaml
websocket:
  mode: ws-server
  ws-server: 0.0.0.0:15630
```

完整 WS 地址：`ws://<你的IP>:15630/ws`

### 反向模式

DDBOT 主动连实现端：

```yaml
websocket:
  mode: ws-reverse
  ws-reverse: ws://localhost:3001
```

DDBOT 连接时会带 `Authorization: Bearer <token>` Header。

---

## `reloadDelay` - 延迟加载

```yaml
reloadDelay:
  enable: true
  time: 3s
```

连上 OneBot 实现端后，延迟加载好友/群/群员信息，避免断线后不刷新。

| 字段 | 说明 |
|------|------|
| `enable` | 是否启用 |
| `time` | 延迟时间，默认 3 秒 |

---

## `debug` - 调试白名单

```yaml
debug:
  group:
    - 0
  uin:
    - 0
```

仅当以 `--debug` 启动时生效。**只有列出的群号和 QQ 号能触发命令**，便于开发调试。

需要配合命令行参数：

```bash
./DDBOT --debug
```

启动后会在 `localhost:6060` 开启 pprof，可用 `go tool pprof` 分析性能。

---

## `message-marker` - 自动已读

```yaml
message-marker:
  disable: false
```

控制是否自动标记群/私聊消息为已读。

| 值 | 行为 |
|----|------|
| `false`（默认） | 开启自动已读 |
| `true` | 禁用自动已读 |

!!! note "为什么要禁用"
    自动已读会消耗一定资源，且部分实现端对已读上报支持不完善。如遇问题可禁用。

---

## `qq-logs` - 聊天内容日志

```yaml
qq-logs:
  enable: false
```

是否在命令行/日志里展示收到的 QQ 聊天内容。

| 值 | 行为 |
|----|------|
| `false`（默认） | 不展示聊天内容 |
| `true` | 展示收到的消息文本 |

!!! warning "隐私"
    开启后聊天内容会写入日志文件，注意隐私。仅排障时临时开启。

---

## `logLevel` - 日志等级

```yaml
logLevel: info   # trace / debug / info / warn / error
```

| 等级 | 说明 |
|------|------|
| `error` | 只看错误，最干净 |
| `warn` | 警告 + 错误 |
| `info` | 常规信息（推荐） |
| `debug` | 调试信息，含部分 base64 |
| `trace` | 最详细，含大量 base64，谨慎使用 |

排障时临时调到 `debug`，平时用 `info`。

---

## 下一步

- [连接 OneBot](../deploy/connect/index.md) -- 实现端对接教程
- [B 站配置](bilibili.md) -- 配置 B 站账号
- [推送与模板开关](push-template.md) -- 推送调度与模板
