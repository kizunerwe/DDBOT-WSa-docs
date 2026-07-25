# 模板入门

DDBOT 的模板系统让你能自定义所有推送、命令、事件的输出格式。本页帮你快速上手。

---

## 启用模板

模板默认关闭，需在 `application.yaml` 启用：

```yaml
template:
  enable: true
```

启用后启动日志会显示「已启用模板」，并自动在 DDBOT 目录下创建 `template/` 文件夹。

```yaml
template:
  enable: true
```

---

## 模板文件

模板是 `template/` 文件夹下、后缀为 `.tmpl` 的文本文件。

- **文件名** = 模板名（如 `notify.group.bilibili.live.tmpl`）
- **文件内容** = 模板内容
- 用任意文本编辑器编辑

### 热重载

DDBOT 会**监控** `template/` 目录，对模板的创建/修改/删除**无需重启**即可生效，自动使用最新模板。

!!! note "去掉 BOM"
    模板文件如果带 BOM，可能导致意外换行。DDBOT 会自动去除 BOM，但建议编辑器保存为 UTF-8 无 BOM。

---

## 第一个模板

### 文字模板

创建 `template/test.tmpl`，内容：

```text
这是一段文字，也是一段模板
```

触发该模板时，会发送：「这是一段文字，也是一段模板」。

### 发送图片

用 `{{ pic "uri" }}` 函数发送图片：

```text
发送一张图片 {{ pic "https://i2.hdslb.com/bfs/face/0bd7082c8c9a14ef460e64d5f74ee439c16c0e88.jpg" }}
```

`pic` 支持 http/https 链接、本地路径、base64 字符串。路径是文件夹时会随机选一张。

### 分段消息

用 `{{ cut }}` 把消息切成多条分段发送：

```text
一、啦啦啦
{{- cut -}}
二、啦啦啦
```

会发送两条消息：「一、啦啦啦」和「二、啦啦啦」。

!!! tip "短横线 `-`"
    `{{-` 和 `-}}` 里的短横线用于控制前后换行符，避免多余空行。

---

## 模板语法

DDBOT 模板基于 Go 标准库 `text/template`，所有 `{{ ... }}` 都有特殊意义。

### 变量

用 `.` 引用模板变量，变量随模板场景不同而不同：

```text
{{ .name }}正在直播
```

### 条件判断

```text
{{ if .success -}}
签到成功！
{{- else -}}
明天再来吧
{{- end }}
```

### 回复消息

```text
{{- reply .msg -}}
你好
```

`reply` 会以回复原消息的方式发送。

### 更多语法

完整语法见 [语法与变量](syntax.md)，全部函数见 [模板函数](funcs.md)。

---

## 模板分类

| 分类 | 文件名前缀 | 作用 |
|------|-----------|------|
| [命令模板](command-tmpl.md) | `command.group.*` / `command.private.*` | 自定义命令回复 |
| [推送模板](notify-tmpl.md) | `notify.group.*` | 自定义推送格式 |
| [事件模板](trigger-tmpl.md) | `trigger.*` | 自定义事件触发 |
| [自定义命令](custom-command.md) | `custom.command.*` | 全新的自定义命令 |
| [定时消息](cronjob.md) | `custom.cronjob.*` | 定时发送消息 |

---

## 内置默认模板

DDBOT 内置了所有模板的默认版本（编译进二进制）。在 `template/` 目录创建**同名**文件即可覆盖默认模板。

内置默认模板列表（位于源码 `lsp/template/default/`）：

- `command.group.checkin.tmpl`
- `command.group.help.tmpl`
- `command.group.list.tmpl`
- `command.group.lsp.tmpl`
- `command.private.help.tmpl`
- `command.private.ping.tmpl`
- `command.private.unknown_cmd_tips.tmpl`
- `notify.group.acfun.live.tmpl`
- `notify.group.bilibili.live.tmpl`
- `notify.group.bilibili.news.tmpl`
- `notify.group.douyin.live.tmpl`
- `notify.group.douyu.live.tmpl`
- `notify.group.huya.live.tmpl`
- `trigger.group.member_in.tmpl`
- `trigger.group.member_out.tmpl`
- `trigger.group.poke.tmpl`
- `trigger.private.group_invited.tmpl`
- `trigger.private.new_friend_added.tmpl`
- `trigger.private.poke.tmpl`

---

## 下一步

- [语法与变量](syntax.md) -- 完整语法和通用变量
- [模板函数](funcs.md) -- 所有可用函数
- [命令模板](command-tmpl.md) -- 自定义命令回复
- [推送模板](notify-tmpl.md) -- 自定义推送格式
