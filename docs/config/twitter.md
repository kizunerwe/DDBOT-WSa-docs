# 推特配置

DDBOT-WSa 新增了对推特（Twitter/X）订阅的支持（实验性）。本页讲 `twitter` 段配置。

!!! warning "实验性功能"
    推特订阅处于实验阶段，依赖第三方 nitter 镜像，稳定性受镜像可用性影响。

---

## 配置示例

```yaml
twitter:
  baseUrl:
    - "https://nitter.net/"
    - "https://nitter.privacyredirect.com/"
    - "https://nitter.tiekoetter.com/"
  interval: 300s
  userAgent:
  cfclearance:
```

---

## 字段详解

### `baseUrl` - nitter 镜像列表

```yaml
twitter:
  baseUrl:
    - "https://nitter.net/"
    - "https://nitter.privacyredirect.com/"
```

推特内容通过 [nitter](https://github.com/nickvdp/nitter)（推特前端镜像）抓取。可配置**多个镜像**，DDBOT 会轮换使用，提高可用性。

| 说明 | 备注 |
|------|------|
| 默认镜像 | 官方 `nitter.net` |
| 第三方镜像 | 可能有额外校验，建议同时配多个 |
| 末尾 `/` | URL 末尾建议带 `/` |

!!! tip "镜像不可用时"
    nitter 镜像经常变动，如果某个镜像挂了，换一个或加多个备用。也可使用 `lightbrd` 等替代镜像。

### `interval` - 检测间隔

```yaml
twitter:
  interval: 300s
```

推特动态的查询间隔。**过快可能导致 IP 被暂时封禁**，建议保持 300s 以上。

### `userAgent` - 浏览器 UA

```yaml
twitter:
  userAgent: "Mozilla/5.0 ..."
```

访问 nitter 镜像时使用的 User-Agent。

- 留空则使用默认 UA
- 部分镜像会校验 UA，如果访问失败可填入你浏览器真实的 UA

??? question "如何查看自己的 UA"
    浏览器访问 [https://httpbin.org/user-agent](https://httpbin.org/user-agent) 即可看到。

### `cfclearance` - Cloudflare 验证

```yaml
twitter:
  cfclearance: "你的cf_clearance"
```

部分 nitter 镜像（如 `lightbrd.com`）启用了 Cloudflare 防护，需要提供 `cf_clearance` cookie 才能访问。

??? question "如何获取 cf_clearance"
    1. 浏览器访问目标镜像（如 `https://lightbrd.com/`）
    2. 完成 Cloudflare 验证
    3. F12 -> Application -> Cookies -> 找到 `cf_clearance`
    4. 复制值填入配置

    !!! warning "cf_clearance 有时效"
        该 cookie 会过期，失效后需重新获取。

---

## 订阅推特

配置好后，用 `/watch` 命令订阅：

```
/watch -s twitter <用户名>
```

例如订阅 `elonmusk`：

```
/watch -s twitter elonmusk
```

推特只支持 `news` 类型（动态），无需指定 `-t`。

---

## m3u8 视频推送

推特动态中的视频（m3u8 格式）可推送到群里，但**需要安装 FFmpeg** 来下载合并分片。

见 [媒体与 FFmpeg](../deploy/connect/media.md)。

---

## 常见问题

### 抓不到推文

- 检查镜像是否可用（浏览器直接访问试试）
- 换一个镜像或多配几个
- 检查 `userAgent` 是否被镜像拒绝
- 启用了 Cloudflare 的镜像需配 `cfclearance`

### 视频推不出来

- 确认已安装 FFmpeg 且在 PATH 或程序目录
- 查看日志是否有 ffmpeg 相关错误

### 风控

推特动态里日文字符较多时容易触发风控，可通过模板调整显示内容（见 [推送模板](../template/notify-tmpl.md)）。

---

## 下一步

- [推特订阅源说明](../sources/twitter.md) -- 订阅机制与模板变量
- [媒体与 FFmpeg](../deploy/connect/media.md) -- m3u8 视频推送
- [推送模板](../template/notify-tmpl.md) -- 自定义推特推送格式
