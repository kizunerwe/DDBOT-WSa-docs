# ACFun

ACFun 直播订阅，master 分支无需额外配置。

!!! tip "next / next-dev 分支增强"
    `next` 及以后分支新增了 **ACFUN 动态（news）推送**，并支持配置 ACFUN 账号实现关注/动态订阅。详见 [版本与分支](../deploy/branches.md#next-分支的新增内容)。

---

## 订阅

```
/watch -s acfun <主播ID>
```

```bash
# 订阅
/watch -s acfun <主播ID>

# 取消订阅
/unwatch -s acfun <主播ID>
```

master 分支只支持 `live` 类型。

### next 分支：订阅动态

```
/watch -s acfun -t news <主播ID>
```

订阅动态需要配置 ACFUN 账号：

```yaml
acfun:
  account:        # ACFUN 账号
  password:       # ACFUN 密码
  authKey:        # 可选
  acPassToken:    # 可选
  unsub: false
  interval: 25s
  onlyOnlineNotify: false
```

---

## 推送模板

模板名：`notify.group.acfun.live.tmpl`

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
ACFUN-{{ .name }}正在直播【{{ .title }}】
{{ .url -}}
{{ pic .cover "[封面]" }}
{{- else -}}
ACFUN-{{ .name }}直播结束了
{{ pic .cover "[封面]" }}
{{- end -}}
```

---

## 配置

master 分支无需额外配置。推送行为可用 `/config` 定制。

next 分支订阅动态需配置 ACFUN 账号，见上文。

详见 [配置命令](../commands/config-cmd.md) 与 [版本与分支](../deploy/branches.md)。

---

## 下一步

- [订阅源总览](index.md)
- [推送模板](../template/notify-tmpl.md)
- [版本与分支](../deploy/branches.md) -- next 分支的 ACFUN 动态
