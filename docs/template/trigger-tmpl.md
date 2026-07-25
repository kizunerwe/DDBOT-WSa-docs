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
| `mute_duration` | int | 禁言时长 |

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
| `file_size` | int | 文件大小 |
| `file_id` | string | 文件 ID |
| `file_url` | string | 文件下载 URL |
| `file_busId` | int | 文件 busId |

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

---

## BOT 状态事件

### BOT 上线

模板名：`notify.bot.online.tmpl`（WSa 新增）

| 变量 | 类型 | 含义 |
|------|------|------|
| `template_name` | string | 模板名 |

### BOT 离线

模板名：`notify.bot.offline.tmpl`（WSa 新增）

| 变量 | 类型 | 含义 |
|------|------|------|
| `template_name` | string | 模板名 |

!!! warning "离线模板注意"
    - 本模板在 heartbeat 事件中触发，需在 BOT 实现端启用心跳上报
    - 每次上报离线都会触发，**需要用 `cooldown` 设置冷却**避免重复通知

    ```text
    {{- if (cooldown "5m" "bot_offline_notify") -}}
    BOT 离线了，请检查！
    {{- end -}}
    ```

### 消息发送失败

模板名：`notify.bot.send_failed.tmpl`（WSa 新增）

需在配置启用：

```yaml
bot:
  sendFailureReminder:
    enable: true
    times: 3
```

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
| `request_id` | int | 申请 ID |
| `member_code` | int64 | 申请者 QQ 号 |
| `member_name` | string | 申请者昵称 |
| `bot_mode` | string | 当前运行模式 |

---

## 下一步

- [命令模板](command-tmpl.md)
- [推送模板](notify-tmpl.md)
- [模板函数](funcs.md)
- [自定义命令](custom-command.md)
