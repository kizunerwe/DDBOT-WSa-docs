# 自定义命令

得益于模板的高度定制化能力，DDBOT 支持通过模板创建全新的自定义命令。

---

## 配置

自定义命令需同时启用模板，并在 `application.yaml` 的 `autoreply` 段定义命令名：

```yaml
template:
  enable: true

autoreply:
  group:
    command: ["群命令1", "群命令2", "群主女装"]
  private:
    command: ["私聊命令1", "私聊命令2"]
```

- `group.command` - 群聊自定义命令列表
- `private.command` - 私聊自定义命令列表

---

## 创建模板文件

每个自定义命令对应一个模板文件：

- 群命令 `群命令1` -> `template/custom.command.group.群命令1.tmpl`
- 群命令 `群命令2` -> `template/custom.command.group.群命令2.tmpl`
- 私聊命令 `私聊命令1` -> `template/custom.command.private.私聊命令1.tmpl`

触发 `/群命令1` 时，发送模板 `custom.command.group.群命令1.tmpl` 的内容。

---

## 可用变量

自定义命令可使用 [通用变量](syntax.md)：

| 变量 | 含义 |
|------|------|
| `group_code` / `group_name` | 群信息 |
| `member_code` / `member_name` | 触发者 |
| `cmd` | 命令名 |
| `args` | 参数数组 |
| `full_args` | 完整参数字符串 |
| `at_targets` | @的成员 QQ 号 |

---

## 示例

### 群主女装

配置：

```yaml
autoreply:
  group:
    command: ["群主女装"]
```

创建 `template/custom.command.group.群主女装.tmpl`：

```text
{{- reply .msg -}}
{{ pic "https://example.com/qunzhu_nvzhuang.jpg" }}
群主女装照来啦！
```

群内发 `/群主女装` 即可触发。

### 带@的命令

```text
{{- if gt (len .at_targets) 0 -}}
{{- $t := index .at_targets 0 -}}
{{- $info := member_info .group_code $t -}}
你@了{{ $info.name }}
{{ $info.name }} 的性别是 {{ if eq $info.gender 2 }}男生{{ else if eq $info.gender 1 }}女生{{ else }}秘密{{ end }}
{{- else -}}
请@TA使用命令喵
{{- end -}}
```

### 带参数的命令

触发 `/查天气 北京`：

```text
{{- $city := index .args 0 -}}
{{- if $city -}}
查询 {{ $city }} 的天气...
{{- $j := httpGet "https://example.com/weather" (dict "city" $city) | toGJson -}}
温度：{{ ($j.Get "temp").String }}
{{- else -}}
请输入城市名，如：{{ prefix }}查天气 北京
{{- end -}}
```

---

## 配合自定义前缀

通过 `customCommandPrefix` 可让自定义命令无前缀触发：

```yaml
customCommandPrefix:
  签到: ""
  群主女装: ""
```

这样直接发「群主女装」即可触发，无需 `/`。

详见 [推送与模板开关 - 自定义命令前缀](../config/push-template.md)。

---

## 注意事项

- 自定义命令**不支持参数解析**为命令选项（`-s` 等），但可通过 `args`/`full_args` 在模板里自行处理
- 命令名不能与内置命令冲突
- 遇到非文字内容（图片等）会跳过该内容，不会停止解析

---

## 下一步

- [定时消息](cronjob.md) -- 定时发送模板消息
- [模板函数](funcs.md) -- 所有可用函数
- [语法与变量](syntax.md) -- 通用变量
