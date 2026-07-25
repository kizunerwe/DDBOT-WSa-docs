# 虎牙

虎牙直播订阅，无需额外配置。

---

## 订阅

```
/watch -s huya <主播ID>
```

主播 ID 是直播间 URL 里的字符串，例如 `https://www.huya.com/xiaoleyan` 的 ID 是 `xiaoleyan`。

```bash
# 订阅虎牙乐爷的直播
/watch -s huya xiaoleyan

# 取消订阅
/unwatch -s huya xiaoleyan
```

只支持 `live` 类型。

---

## 推送模板

模板名：`notify.group.huya.live.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `living` | bool | 是否正在直播 |
| `name` | string | 主播昵称 |
| `title` | string | 直播标题 |
| `url` | string | 直播间链接 |
| `cover` | string | 封面或头像 |

默认模板：

```text
{{ if .living -}}
虎牙-{{ .name }}正在直播【{{ .title }}】
{{ .url -}}
{{ pic .cover "[封面]" }}
{{- else -}}
虎牙-{{ .name }}直播结束了
{{ pic .cover "[封面]" }}
{{- end -}}
```

---

## 配置

无需额外配置。推送行为可用 `/config` 定制：

```
/config at_all -s huya xiaoleyan on
```

详见 [配置命令](../commands/config-cmd.md)。

---

## 下一步

- [订阅源总览](index.md)
- [推送模板](../template/notify-tmpl.md)
