# ACFun

ACFun 直播与动态订阅。

!!! tip "分支差异"
    `master` 只支持直播（`live`）；`[next]` 起支持 **ACFUN 动态（news）推送**，并可配置 ACFUN 账号实现关注/动态订阅。详见 [版本与分支](../deploy/branches.md)。

---

## 订阅

```
/watch -s acfun <主播ID>
```

```bash
# 订阅直播
/watch -s acfun <主播ID>

# 取消订阅
/unwatch -s acfun <主播ID>
```

`master` 只支持 `live`；`[next]` 起支持 `live` 与 `news`，不指定 `-t` 时默认 `live`。

### 订阅动态（`[next]`）

```
/watch -s acfun -t news <主播ID>
```

`[next]` 起订阅动态需要配置 ACFUN 账号或 cookie（仅订阅直播无需配置）：

```yaml
acfun:
  interval: 25s      # 轮询间隔，master 即读取；未配置默认 20s
  # 以下配置 [next] 起才被读取：
  account:           # ACFUN 账号
  password:          # ACFUN 密码
  authKey:           # 可选 cookie
  acPassToken:       # 可选 cookie
  unsub: false       # 取消订阅时是否取关
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

`[next]` 动态推送模板名为 `notify.group.acfun.news.tmpl`（`master` 无该模板）。

---

## 配置

`master` 订阅直播无需额外配置，可通过 `acfun.interval` 调整轮询间隔。推送行为可用 `/config` 定制。

`[next]` 订阅动态需配置 ACFUN 账号（见上文）。

详见 [配置命令](../commands/config-cmd.md) 与 [版本与分支](../deploy/branches.md)。

---

## 下一步

- [订阅源总览](index.md)
- [推送模板](../template/notify-tmpl.md)
- [版本与分支](../deploy/branches.md) -- next 分支的 ACFUN 动态
