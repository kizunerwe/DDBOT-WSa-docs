# 命令模板

自定义命令的回复格式。所有命令模板都可用 [通用变量](syntax.md)。

---

## `/签到`

模板名：`command.group.checkin.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `success` | bool | 本次签到是否成功（一天内首次成功） |
| `score` | int | 当前积分 |

默认模板：

```text
{{ reply .msg }}{{if .success}}签到成功！获得1积分，当前积分为{{.score}}{{else}}明天再来吧，当前积分为{{.score}}{{end}}
```

---

## `/help`（群聊版）

模板名：`command.group.help.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| 无 | | |

默认模板：

```text
DDBOT是一个多功能单推专用推送机器人，支持b站、斗鱼、油管、虎牙推送
```

---

## `/help`（私聊版）

模板名：`command.private.help.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| 无 | | |

??? note "默认模板"
    ```text
    常见订阅用法：
    以示例UID:97505为例
    首先订阅直播信息：{{ prefix }}watch 97505
    然后订阅动态信息：{{ prefix }}watch -t news 97505
    由于通常动态内容较多，可以选择不推送转发的动态
    {{ prefix }}config filter not_type 97505 转发
    还可以选择开启直播推送时@全体成员：
    {{ prefix }}config at_all 97505 on
    以及开启下播推送：
    {{ prefix }}config offline_notify 97505 on
    BOT还支持更多功能，详细命令介绍请查看命令文档：
    https://cnxysoft.github.io/DDBOT-WSa/commands/index
    使用时请把示例UID换成你需要的UID
    当您完成所有配置后，可以使用{{ prefix }}silence命令，让bot专注于推送，在群内发言更少
    {{- cut -}}
    B站专栏介绍：https://www.bilibili.com/read/cv10602230
    如果您有任何疑问或者建议，请反馈到交流群：755612788（已满）、980848391
    ```

---

## `/lsp`

模板名：`command.group.lsp.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| 无 | | |

默认模板：

```text
{{ reply .msg -}}
LSP竟然是你
```

---

## `/ping`（私聊）

模板名：`command.private.ping.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| 无 | | |

默认模板：

```text
pong
```

---

## `/list`（群聊）

模板名：`command.group.list.tmpl`（WSa 支持自定义）

可使用 `outputIList` 输出原版列表，或用 `getIListJson` 自定义渲染：

```text
{{ outputIList .msg_context .group_code "bilibili" }}
```

或：

```text
{{- $json := getIListJson .group_code "bilibili" -}}
{{- $list := jsonToDictOrArray $json true -}}
{{ range $list -}}
{{ .name }} ({{ .id }})
{{- end }}
```

详见 [模板函数 - 列表输出](funcs.md)。

---

## 私聊无效指令提示

模板名：`command.private.unknown_cmd_tips.tmpl`（WSa 新增）

私聊收到无效命令时触发。

| 变量 | 类型 | 含义 |
|------|------|------|
| `cmd` | string | 命令名 |
| `args` | []string | 参数 |
| `full_args` | string | 完整参数 |
| `help_cmd` | string | 帮助命令名 |

---

## 自定义示例

### 签到带运气

```text
{{- reply .msg -}}
{{ if .success -}}
签到成功！获得1积分，当前{{.score}}分
今日运势：{{ roll 1 100 }}
{{- else -}}
明天再来吧，当前{{.score}}分
{{- end }}
```

### help 带群信息

```text
{{- reply .msg -}}
欢迎在群【{{ .group_name }}】使用 DDBOT
输入 {{ prefix }}watch <UID> 订阅B站直播
输入 {{ prefix }}list 查看订阅
```

---

## 下一步

- [通用变量](syntax.md)
- [模板函数](funcs.md)
- [自定义命令](custom-command.md) -- 创建全新的命令
