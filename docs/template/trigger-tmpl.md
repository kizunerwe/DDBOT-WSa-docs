# 事件模板

DDBOT 在收到 QQ 事件（进退群、戳一戳、禁言、群文件上传等）时，可触发对应模板发送消息。默认模板多为空（不发送），按需启用。

---

## 群事件

### 新成员加入群

模板名：`trigger.group.member_in.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `group_code` | int64 | 群号 |
| `group_name` | string | 群名 |
| `member_code` | int64 | 新成员 QQ 号 |
| `member_name` | string | 新成员昵称 |

默认为空（不发送）。

示例：

```text
欢迎 {{ .member_name }} 加入{{ .group_name }}！
```

### 成员退出群

模板名：`trigger.group.member_out.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `group_code` | int64 | 群号 |
| `group_name` | string | 群名 |
| `member_code` | int64 | 退出成员 QQ 号 |
| `member_name` | string | 退出成员昵称 |

默认为空。

### 群戳一戳

模板名：`trigger.group.poke.tmpl`（v1.0.9+）

| 变量 | 类型 | 含义 |
|------|------|------|
| `group_code` | int64 | 群号 |
| `group_name` | string | 群名 |
| `member_code` | int64 | 发送者 QQ 号 |
| `member_name` | string | 发送者昵称 |
| `receiver_code` | int64 | 被戳者 QQ 号 |
| `receiver_name` | string | 被戳者昵称 |

!!! note "所有戳一戳都会收到"
    群内所有戳一戳都会触发该模板。只想处理 BOT 被戳，需用 `receiver_code` 判断：

    ```text
    {{ if eq .receiver_code (bot_uin) -}}
    别戳我！
    {{- end }}
    ```

### BOT 被禁言

模板名：`trigger.group.bot_mute.tmpl`（v0.2.5+，只通知 BOT 管理员）

| 变量 | 类型 | 含义 |
|------|------|------|
| `group_code` | int64 | 群号 |
| `group_name` | string | 群名 |
| `member_code` | int64 | 被禁言者 QQ 号 |
| `member_name` | string | 被禁言者昵称 |
| `operator_code` | int64 | 操作者 QQ 号 |
| `operator_name` | string | 操作者昵称 |
| `mute_duration` | int32 | 禁言时长 |

默认为空。

### 群名片变更

模板名：`trigger.group.card_updated.tmpl`（v0.2.7+）

| 变量 | 类型 | 含义 |
|------|------|------|
| `group_code` | int64 | 群号 |
| `group_name` | string | 群名 |
| `member_code` | int64 | 改名片者 QQ 号 |
| `member_name` | string | 新名片 |
| `old_member_name` | string | 旧名片 |

默认为空。

### 群权限变更

模板名：`trigger.group.admin_changed.tmpl`（v0.2.7+）

| 变量 | 类型 | 含义 |
|------|------|------|
| `group_code` | int64 | 群号 |
| `group_name` | string | 群名 |
| `member_code` | int64 | 被改权限者 QQ 号 |
| `member_name` | string | 昵称 |
| `old_permission` | string | 旧权限 |
| `permission` | string | 新权限 |

`permission` 取值：`群员` / `管理员` / `群主` / `未知权限`

默认为空。

### 群文件上传

模板名：`trigger.group.upload.tmpl`（WSa 新增）

| 变量 | 类型 | 含义 |
|------|------|------|
| `group_code` | int64 | 群号 |
| `group_name` | string | 群名 |
| `member_code` | int64 | 上传者 QQ 号 |
| `member_name` | string | 上传者昵称 |
| `file_name` | string | 文件名 |
| `file_size` | int64 | 文件大小 |
| `file_id` | string | 文件 ID |
| `file_url` | string | 文件下载 URL |
| `file_busId` | int32 | 文件 busId |

### 群消息撤回 `[next-dev]`

模板名：`trigger.group.recall.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `group_code` | int64 | 群号 |
| `group_name` | string | 群名 |
| `author_uin` | int64 | 消息作者 QQ 号 |
| `author_name` | string | 消息作者昵称 |
| `operator_uin` | int64 | 撤回操作者 QQ 号 |
| `operator_name` | string | 撤回操作者昵称 |
| `message_id` | int32 | 被撤回的消息 ID |

### 群成员专属头衔变更 `[next-dev]`

模板名：`trigger.group.special_title_updated.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `group_code` | int64 | 群号 |
| `group_name` | string | 群名 |
| `member_code` | int64 | 成员 QQ 号 |
| `member_name` | string | 成员昵称 |
| `new_title` | string | 新头衔 |

### 群精华消息变更 `[next-dev]`

模板名：`trigger.group.essence_changed.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `group_code` | int64 | 群号 |
| `group_name` | string | 群名 |

### 群解散 `[next-dev]`

模板名：`trigger.group.disband.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `group_code` | int64 | 群号 |
| `group_name` | string | 群名 |
| `operator_code` | int64 | 操作者 QQ 号（可能缺省） |
| `operator_name` | string | 操作者昵称（可能缺省） |

该模板发送给 BOT 管理员，不发送到群内。

### 群消息表情回应 `[next-dev]`

模板名：`trigger.group.emoji_like.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `group_code` | int64 | 群号 |
| `member_code` | int64 | 回应者 QQ 号 |
| `member_name` | string | 回应者昵称 |
| `message_id` | int64 | 被回应的消息 ID |
| `emoji_id` | string | 表情 ID |
| `emoji_count` | int | 表情数量 |
| `is_add` | bool | true 添加 / false 取消 |

### 戳一戳撤回 `[next-dev]`

模板名：`trigger.group.poke_recall.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `group_code` | int64 | 群号 |
| `group_name` | string | 群名 |
| `member_code` | int64 | 发送者 QQ 号 |
| `member_name` | string | 发送者昵称 |
| `receiver_code` | int64 | 被戳者 QQ 号 |
| `receiver_name` | string | 被戳者昵称 |

### 通用群消息处理 `[next-dev]`

模板名：`trigger.group.message.tmpl`（**无内置默认模板**，需自行在 `template/` 下创建）

在群消息处理流程中被调用：非命令消息触发一次（`isCommand=false`）；命令消息在命令执行后触发一次。可用变量为群命令模板的通用变量 `msg` / `group_code` / `group_name` / `member_code` / `member_name` / `command`；**不提供 `template_name`**（此处直接使用 `commonTemplateData()`，不像其他命令模板会补充 `template_name`）。额外提供：

| 变量 | 类型 | 含义 |
|------|------|------|
| `isCommand` | bool | 是否带命令前缀 |
| `isMatched` | bool | 命令消息下恒为 true（实现写死），不能用于判断命令是否匹配到处理函数 |
| `commandName` | string | 命令名 |

未创建该模板文件时不会发送任何内容。

---

## 私聊事件

### 添加新好友

模板名：`trigger.private.new_friend_added.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `member_code` | int64 | 新好友 QQ 号 |
| `member_name` | string | 新好友昵称 |
| `.command.HelpCommand` | string | 帮助命令名（默认 `help`） |

默认模板：

```text
阁下的好友请求已通过，请使用<{{ prefix .command.HelpCommand }}>(不含括号)查看帮助，然后在群成员页面邀请bot加群（bot不会主动加群）。
```

### 接受加群邀请

模板名：`trigger.private.group_invited.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `group_code` | int64 | 群号 |
| `group_name` | string | 群名 |
| `member_code` | int64 | 邀请人 QQ 号 |
| `member_name` | string | 邀请人昵称 |

默认模板：

```text
阁下的群邀请已通过，基于对阁下的信任，阁下已获得本bot在群【{{ .group_name }}】的控制权限，相信阁下不会滥用本bot。
```

### 私聊戳一戳

模板名：`trigger.private.poke.tmpl`（v1.0.9+）

| 变量 | 类型 | 含义 |
|------|------|------|
| `member_code` | int64 | 发送者 QQ 号 |
| `member_name` | string | 发送者昵称 |

默认为空。

### 好友消息撤回 `[next-dev]`

模板名：`trigger.private.friend_recall.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `friend_code` | int64 | 好友 QQ 号 |
| `friend_name` | string | 好友昵称 |
| `message_id` | int32 | 被撤回的消息 ID |

### 个人资料被赞 `[next-dev]`

模板名：`trigger.private.profile_like.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `operator_id` | int64 | 点赞者 QQ 号 |
| `operator_nick` | string | 点赞者昵称 |
| `times` | int | 点赞次数 |

该模板发送给 BOT 管理员。

---

## BOT 状态事件

`notify.bot.online` / `notify.bot.offline` / `notify.bot.send_failed` 这三个模板对应的事件会渲染模板，但调用处丢弃了渲染结果（`_, _ = template.LoadAndExec(...)`），**当前不会发出任何消息**。本节的 `trigger.bot.new_friend_request.tmpl` 是例外，见下。

### BOT 上线

模板名：`notify.bot.online.tmpl`（WSa 新增）

| 变量 | 类型 | 含义 |
|------|------|------|
| `template_name` | string | 模板名 |

触发条件：`master` / `next` 在心跳从离线变为在线时派发 `BotOnlineEvent`（`miraigo/client/client.go`）；`[next-dev]` 的 `DispatchBotOnline` 没有任何调用点，事件不会触发。

### BOT 离线

模板名：`notify.bot.offline.tmpl`（WSa 新增）

| 变量 | 类型 | 含义 |
|------|------|------|
| `template_name` | string | 模板名 |

触发条件：`master` / `next` 在心跳离线或收到离线通知时派发 `BotOfflineEvent`；`[next-dev]` 在实现端上报 `bot_offline` 时派发（`adapter/messenger.go`）。两者都只渲染模板。

### 消息发送失败

模板名：`notify.bot.send_failed.tmpl`（WSa 新增）

`master` / `next`：配置 `bot.sendFailureReminder.enable: true` 后，连续发送失败次数达到 `bot.sendFailureReminder.times` 会派发 `BotSendFailedEvent`（`miraigo/client/client.go` 的 `handleSendFailed`），模板被渲染但结果被丢弃。`[next-dev]`：该特性已整体移除——`bot.sendFailureReminder` 配置键不再生成、订阅段也已删除，`BotSendFailedEvent` 没有任何派发点，事件不会触发。

| 变量 | 类型 | 含义 |
|------|------|------|
| `message` | string | 失败的消息 |
| `target_id` | int64 | 目标 QQ/群号 |
| `target_type` | int | 目标类型（0 群 / 1 好友） |
| `target_name` | string | 目标名称 |
| `times` | int | 失败次数 |

### 新好友申请

模板名：`trigger.bot.new_friend_request.tmpl`（WSa 新增）

| 变量 | 类型 | 含义 |
|------|------|------|
| `request_id` | int64 | 申请 ID |
| `member_code` | int64 | 申请者 QQ 号 |
| `member_name` | string | 申请者昵称 |
| `bot_mode` | string | 当前运行模式 |

该模板渲染结果会通过 `SendMsgToAdmin` 私聊发送给 BOT 管理员（`master` / `next` / `next-dev` 均如此），不受上文「丢弃渲染结果」影响。

---

## 下一步

- [命令模板](command-tmpl.md)
- [推送模板](notify-tmpl.md)
- [模板函数](funcs.md)
- [自定义命令](custom-command.md)
