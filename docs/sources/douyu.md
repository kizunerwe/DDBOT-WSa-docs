# 斗鱼

斗鱼直播订阅，无需额外配置。

---

## 订阅

```
/watch -s douyu <房间号>
```

房间号是直播间 URL 里的数字，例如 `https://www.douyu.com/6655` 的房间号是 `6655`。

```bash
# 订阅斗鱼 6655 直播间
/watch -s douyu 6655

# 取消订阅
/unwatch -s douyu 6655
```

只支持 `live` 类型，无需指定 `-t`。

---

## 推送模板

模板名：`notify.group.douyu.live.tmpl`

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
斗鱼-{{ .name }}正在直播【{{ .title }}】
{{ .url -}}
{{ pic .cover "[封面]" }}
{{- else -}}
斗鱼-{{ .name }}直播结束了
{{ pic .cover "[封面]" }}
{{- end -}}
```

---

## 配置

无需在 `application.yaml` 配置斗鱼相关项。推送行为可用 `/config` 定制：

```
/config at_all -s douyu 6655 on
/config offline_notify -s douyu 6655 on
```

详见 [配置命令](../commands/config-cmd.md)。

---

## 下一步

- [订阅源总览](index.md)
- [推送模板](../template/notify-tmpl.md)
