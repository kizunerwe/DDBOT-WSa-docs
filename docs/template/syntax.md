# 语法与变量

DDBOT 模板基于 Go 标准库 [`text/template`](https://pkg.go.dev/text/template)，并扩展了大量函数。本页讲常用语法和通用变量。

---

## 基本语法

所有 `{{ ... }}` 都会被解析，其余文字原样输出。

### 输出变量

```text
{{ .name }}
```

`.` 表示当前上下文，`.name` 表示上下文的 `name` 字段。

### 赋值变量

```text
{{- $x := "hello" -}}
{{ $x }}
```

用 `$名字 := 值` 定义变量，用 `$名字` 引用。

### 管道

函数可以管道串联：

```text
{{ "hello" | upper }}
```

等价于 `upper "hello"`，输出 `HELLO`。

---

## 控制结构

### 条件 `if`

```text
{{ if .success -}}
成功
{{- else if .retry -}}
重试中
{{- else -}}
失败
{{- end }}
```

### 循环 `range`

```text
{{ range .args -}}
参数：{{ . }}
{{- end }}
```

`range` 遍历列表，`{{ . }}` 是当前元素。也可获取索引：

```text
{{ range $i, $v := .args -}}
{{ $i }}: {{ $v }}
{{- end }}
```

### 定义/引用模板

```text
{{ define "greeting" }}你好{{ end }}
{{ template "greeting" }}
```

---

## 空白控制

`{{-` 去除前面的空白，`-}}` 去除后面的空白：

```text
{{- "无前导换行" -}}
```

```text
一、啦啦啦
{{- cut -}}
二、啦啦啦
```

`{{- cut -}}` 会吃掉前后的换行，避免消息里出现多余空行。

---

## 通用变量

所有命令模板都能用以下变量：

| 变量 | 类型 | 含义 | 版本 |
|------|------|------|------|
| `group_code` | int | 触发命令的群号（私聊为空） | - |
| `group_name` | string | 群名称（私聊为空） | - |
| `member_code` | int | 触发命令的成员 QQ 号 | - |
| `member_name` | string | 成员名称 | - |
| `cmd` | string | 命令名 | v1.0.7+ |
| `args` | []string | 命令参数数组 | v1.0.7+ |
| `at_targets` | []int64 | @的成员 QQ 号 | v1.0.8+ |
| `full_args` | string | 完整参数字符串（含空格） | v1.0.8+ |
| `template_name` | string | 当前模板名 | v1.0.9+ |

### `args` vs `full_args`

触发 `/test a b c` 时：

- `{{ .args }}` 是数组 `["a", "b", "c"]`，需配合 `index` 用
- `{{ .full_args }}` 是字符串 `"a b c"`

```text
{{ index .args 0 }}          {{/* 输出 a */}}
{{ .full_args }}             {{/* 输出 a b c */}}
```

---

## 特殊函数

### `reply` - 回复消息

```text
{{ reply .msg }}
```

以回复原消息的方式发送。常用于命令回复开头。

### `cut` - 分段消息

```text
第一段
{{- cut -}}
第二段
```

把消息切分成多条发送。

### `prefix` - 命令前缀

```text
{{ prefix }}watch
```

引用配置中的命令前缀（默认 `/`），方便在帮助文本里写命令。

### `pic` - 发送图片

```text
{{ pic "https://example.com/img.jpg" }}
{{ pic "/path/to/img.jpg" }}
{{ pic "base64字符串" }}
```

支持 URL、本地路径、base64。路径是文件夹时随机选一张。

---

## 注释

```text
{{/* 这是注释，不输出 */}}
```

---

## 字符串

支持反引号原始字符串：

```text
{{ `多行
字符串` }}
```

---

## 下一步

- [模板函数](funcs.md) -- 所有可用函数
- [命令模板](command-tmpl.md) -- 各命令的专用变量
- [Go text/template 官方文档](https://pkg.go.dev/text/template)
