# 推特

DDBOT-WSa 新增了对推特（Twitter/X）订阅的支持（实验性）。通过 nitter 镜像抓取推文。

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

## 配置（master：mirror 模式）

```yaml
twitter:
  baseUrl:
    - "https://nitter.net/"
    - "https://nitter.privacyredirect.com/"
  interval: 300s
  userAgent:
  cfclearance:
```

| 字段 | 说明 |
|------|------|
| `baseUrl` | nitter 镜像列表，可配多个轮换 |
| `interval` | 查询间隔，过快可能被封 IP |
| `userAgent` | 浏览器 UA，留空用默认 |
| `cfclearance` | Cloudflare 验证 cookie（启用 CF 的镜像需要） |

详见 [推特配置](../config/twitter.md)。

### next 分支：api 模式

`next` 分支新增 `mode` 字段，支持直接用 Twitter API（需真实账号 cookie）：

```yaml
twitter:
  mode: mirror         # mirror（默认）/ api
  unsub: false         # 取消订阅时是否自动取关

  # api 模式额外字段：
  auth_token:          # Twitter auth_token cookie
  ct0:                 # Twitter ct0 cookie
  bearerToken:         # Bearer Token
  queryId:             # 搜索 API queryId
  screenName:          # 账号 screen_name（可选）
```

!!! warning "api 模式需高信誉账号"
    api 模式需要真实 Twitter 账号 cookie，且账号信誉度要高，否则可能被风控。

---

## m3u8 视频推送

推文中的视频（m3u8 格式）可推送到群里，**需要 FFmpeg** 下载合并分片：

```
{{ video .video_url }}
```

见 [媒体与 FFmpeg](../deploy/connect/media.md)。

---

## 推送模板

推特推送使用专门的消息组装逻辑，支持：

- 推文文字内容
- 推文图片
- m3u8 视频（需 ffmpeg）
- 多镜像源轮换

可通过模板进一步自定义，见 [推送模板](../template/notify-tmpl.md)。

!!! tip "风控"
    日文字符多的推文容易触发风控，可通过模板调整显示内容。

---

## 常见问题

### 抓不到推文

- 镜像不可用：换一个或多配几个
- UA 被拒：填入真实浏览器 UA
- CF 防护：配 `cfclearance`

### 视频推不出来

- 确认已安装 FFmpeg
- 查看日志是否有 ffmpeg 错误

### 风控

- 调大 `interval`
- 模板里减少日文字符
- 用 `nameStrategy` 控制名称显示

---

## 下一步

- [推特配置](../config/twitter.md) -- 完整配置
- [媒体与 FFmpeg](../deploy/connect/media.md) -- m3u8 视频
- [版本与分支](../deploy/branches.md) -- next 分支的 api 模式
- [订阅源总览](index.md)
