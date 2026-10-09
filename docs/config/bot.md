# bot 与连接配置

本页讲 `bot`、`websocket`、`debug`、`message-marker`、`qq-logs` 这几组配置。

---

## `bot` - 基本设置

```yaml
bot:
  commandPrefix: "/"         # 命令前缀，默认 /
  onDisconnected: ""         # 写 "exit" 直接退出；留空时 master / next 尝试重连（失败也退出），[next-dev] 仅记录日志
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
| `commandPrefix` | string | `/` | 命令触发前缀。可改成其他符号 |
| `onDisconnected` | string | 空 | 写 `exit` 直接退出程序。留空或缺失时：`master` / `next` 会先尝试会话重连（`bot.ReLogin`），重连失败同样会退出；`[next-dev]` 仅记录日志、不退出 |
| `onJoinGroup.rename` | string | `【bot】`（生成值） | 进群后自动改群名片，留空则不改。最长 60 字节（中文约 20 字），超出部分按字节截断 |

!!! warning "`bot.account` / `bot.password`"
    这两个键不在自动生成的配置里，QQ 连接由 OneBot 实现端负责。`master` / `next` 只有内置的 MiraiGo 初始化会读到它们（`bot/bot/bot.go`），而登录调用已停用（根 `bot.go` 里 `bot.Login()` 被注释），填了不起作用；`[next-dev]` 起代码完全不读取这两个键。

### `sendFailureReminder` - 发送失败提醒

```yaml
bot:
  sendFailureReminder:
    enable: false
    times: 3
```

消息发送失败累计达到 `times` 次后，触发模板 `notify.bot.send_failed.tmpl` 通知管理员。模板可用变量：`message`、`target_id`、`target_type`、`target_name`、`times`。

!!! note "分支差异"
    - `master` / `next`：功能完整接线（读取 `enable`/`times`，发布 `BotSendFailedEvent`，订阅后渲染模板）。
    - `[next-dev]`：仅有事件订阅、没有发布点，`enable`/`times` 不会触发提醒。

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
| `expire` | 离线消息有效期。生成配置写的是 `30m`；未配置时 `master` / `next` 解析为 0，所有缓存消息都会被当作过期丢弃；`[next-dev]` 未配置或值非法时回退 `30m` |

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
| `token` | Access Token，与实现端一致。`master` / `next` 下始终可选；`[next-dev]` 下监听非回环地址时必须配置 |
| `ws-server` | 正向模式监听地址。`master` / `next` 默认 `0.0.0.0:15630`（全部网卡），路径固定 `/ws`；`[next-dev]` 默认 `127.0.0.1:15630`（仅本机），且监听非回环地址时**必须配置 `token`**，否则拒绝启动 |
| `ws-reverse` | 反向模式连接地址，默认 `ws://localhost:3001`（LLOneBot 默认端口） |

!!! tip "新手默认即可"
    保持默认 `ws-server` 模式，让实现端反向连到 `ws://127.0.0.1:15630/ws`。

    详见 [连接 OneBot](../deploy/connect/index.md)。

!!! warning "监听地址与 token（`[next-dev]`）"
    `master` / `next` 默认监听 `0.0.0.0:15630`（全部网卡），`token` 始终可选，BOT 不会因缺少 `token` 拒绝启动。

    `[next-dev]` 默认收紧为 `127.0.0.1:15630`（仅本机）：未配置 `token` 时只允许监听回环地址；若改为 `0.0.0.0:15630`、具体网卡或公网 IP，必须同时配置 `websocket.token`，否则 BOT **拒绝启动**——避免无鉴权的控制端口暴露出去。

    `[next-dev]` 的 ws-server 还**始终**校验浏览器 `Origin`（与是否配置 `token` 无关）：带 `Origin` 的请求必须与请求 `Host` 同源，否则拒绝；不发送 `Origin` 的实现端不受影响。`master` / `next` 不校验 `Origin`。

### 正向模式（默认）

DDBOT 在本地开 HTTP/WS 服务器，等实现端连过来：

```yaml
websocket:
  mode: ws-server
  ws-server: 0.0.0.0:15630   # master / next 默认
```

完整 WS 地址：`ws://127.0.0.1:15630/ws`。`master` / `next` 默认监听全部网卡，其它机器可直接连入；`[next-dev]` 默认仅本机，要让其它机器连入需把 `ws-server` 改成 `0.0.0.0:15630`（或具体网卡 IP）并配置 `websocket.token`。

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

!!! note "分支差异"
    `master` / `next` 生成该段并生效；`[next-dev]` 起代码已移除该配置键，填写不起作用。

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

!!! warning "`[next-dev]` 未实现"
    `[next-dev]` 的适配器模式下自动已读尚未实现（`msg-marker/marker.go` 只打印一行日志、没有任何标记调用），即使 `disable: false` 也不会自动已读。

!!! note "为什么要禁用"
    自动已读会消耗一定资源，且部分实现端对已读上报支持不完善。如遇问题可禁用。

---

## `qq-logs` - 聊天内容日志

```yaml
qq-logs:
  enable: false   # master / next 需手写；[next-dev] 生成的配置写作 true
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
| `debug` | 调试信息 |
| `trace` | 最详细，谨慎使用 |

排障时临时调到 `debug`，平时用 `info`。

---

## 下一步

- [连接 OneBot](../deploy/connect/index.md) -- 实现端对接教程
- [B 站配置](bilibili.md) -- 配置 B 站账号
- [推送与模板开关](push-template.md) -- 推送调度与模板
