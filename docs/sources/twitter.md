# 推特

DDBOT-WSa 新增了对推特（Twitter/X）订阅的支持（实验性）。`master` / `next` 通过 nitter 镜像（mirror）抓取推文；`api` 模式为 `[next-dev]` 新增。

!!! warning "实验性功能"
    依赖第三方 nitter 镜像，稳定性受镜像可用性影响。

---

## 订阅

```
/watch -s twitter <用户名>
```

用户名是不带 `@` 的推特用户名，例如 `elonmusk`。

```bash
# 订阅推特
/watch -s twitter elonmusk

# 取消订阅
/unwatch -s twitter elonmusk
```

只支持 `news` 类型（推文）。

---

## 配置

```yaml
twitter:
  mode: mirror            # [next-dev] 读取；master / next 不读取该键，固定 mirror
  baseUrl:                # mirror 模式下的 nitter 镜像列表
    - "https://nitter.net/"
    - "https://nitter.privacyredirect.com/"
  interval: 300s          # 查询间隔
  userAgent:              # 浏览器 UA，留空用默认
  unsub: false            # [next-dev] 取消订阅时是否自动取关
  retweetFullText: false  # [next-dev] 转发推文是否显示原推文完整内容
```

| 字段 | 说明 |
|------|------|
| `mode` | `[next-dev]` `mirror`（nitter 镜像）/ `api`（Twitter API + cookie），缺省和未知值都回退 `api`；`master` / `next` 不读取该键，固定 mirror |
| `baseUrl` | mirror 模式的 nitter 镜像列表，可配多个轮换 |
| `interval` | 查询间隔。`master` / `next` 未配置时默认 30s、无警告；`[next-dev]` 未配置时默认 120s，显式配置低于 120s 会打印建议下限警告（仍按配置值生效）。过快可能被封 IP |
| `userAgent` | 浏览器 UA，留空用默认 |
| `unsub` | `[next-dev]` 读取：取消订阅时是否自动取关（仅 api 抓取模式非 `per_user` 时生效） |
| `retweetFullText` | `[next-dev]` 转发推文是否显示完整原文 |

!!! warning "`mode` 属于 `[next-dev]`"
    `master` / `next` 的 `lsp/twitter/init.go` 都不读取 `twitter.mode`，只支持 nitter 镜像（`git show origin/master:lsp/twitter/init.go`、`origin/next:lsp/twitter/init.go`）。

    `[next-dev]` 的 `resolveTwitterMode`（`origin/next-dev:lsp/twitter/init.go`）对缺省值和未知值一律回退 `api`，只有显式配置 `mirror` 才使用 nitter 镜像。

!!! note "没有 cf_clearance 配置项"
    当前代码不读取 `twitter.cfclearance` / `cf_clearance`，填了不会生效。

详见 [推特配置](../config/twitter.md)。

### api 模式（`[next-dev]`）

`[next-dev]` 下 `mode: api`（缺省即 api）时直接用 Twitter API（需真实账号 cookie）：

```yaml
twitter:
  mode: api
  apiFetchMode: home_timeline  # home_timeline（缺省）/ per_user，未知值回退 home_timeline
  # api 模式额外字段：
  auth_token:          # Twitter auth_token cookie
  ct0:                 # Twitter ct0 cookie
  bearerToken:         # Bearer Token
  queryId:             # 搜索 API queryId（可从 sw.js 自动获取）
  userTweetsQueryId:   # per_user 模式的用户时间线 queryId
  screenName:          # 账号 screen_name（可选，自动获取）
  queryIdRefreshInterval: 168h  # queryId 缓存刷新间隔
```

!!! tip "自动翻译（`[next-dev]`）"
    `twitter.translate.*`（`enabled` / `targetLang` / `minInterval` / `batchWait`）可把推文自动翻译后再推送。

!!! warning "api 模式需高信誉账号"
    api 模式需要真实 Twitter 账号 cookie，且账号信誉度要高，否则可能被风控。

---

## m3u8 视频推送

推文中的视频（m3u8 格式）可推送到群里，**需要 FFmpeg** 下载合并分片：

模板中处理视频的片段（摘自 `notify.group.twitter.news.tmpl`）：

```text
{{ range $media := $dynamic.Media -}}
  {{ if or (eq $media.Type "video") (eq $media.Type "video(m3u8)") -}}
    {{ printf "%s\n" $dynamic.Url -}}
    {{ cut -}}
    {{ if ne $media.Base64 "" -}}
      {{ video $media.Base64 -}}
    {{ else -}}
      {{ video $media.Url -}}
    {{ end -}}
    {{ fin -}}
  {{ end -}}
{{ end -}}
```

`master` 没有模板，m3u8 下载与转码写在 `lsp/twitter/notify.go` 的内置逻辑里；`[next]` / `[next-dev]` 由 `notify.group.twitter.news.tmpl` 模板渲染，模板对视频统一用 `$media.Url` / `$media.Base64`，**没有 `.video_url` 变量**。

见 [媒体与 FFmpeg](../deploy/connect/media.md)。

---

## 推送模板

- `master`：没有独立模板文件，推送由内置逻辑组装（推文文字、图片、m3u8 视频、多镜像源轮换）
- `[next]` / `[next-dev]`：模板 `notify.group.twitter.news.tmpl`，数据为 `.dynamic`（`User` / `Date` / `Content` / `Media` / `Retweet` / `Quote` 等）；`[next-dev]` 另含 `.dynamic.TranslatedText`

可通过模板进一步自定义，见 [推送模板](../template/notify-tmpl.md)。

!!! tip "风控"
    日文字符多的推文容易触发风控，可通过模板调整显示内容。

---

## 常见问题

### 抓不到推文

- 镜像不可用：换一个或多配几个
- UA 被拒：填入真实浏览器 UA
- Cloudflare 防护：mirror 模式没有可用的绕过配置，换一个不需要 CF 验证的镜像

### 视频推不出来

- 确认已安装 FFmpeg
- 查看日志是否有 ffmpeg 错误

### 风控

- 调大 `interval`（`[next-dev]` 推荐不低于 120s）
- 模板里减少日文字符

---

## 下一步

- [推特配置](../config/twitter.md) -- 完整配置
- [媒体与 FFmpeg](../deploy/connect/media.md) -- m3u8 视频
- [版本与分支](../deploy/branches.md)
- [订阅源总览](index.md)
