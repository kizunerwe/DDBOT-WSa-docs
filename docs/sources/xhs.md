# 小红书

小红书直播与图文/视频动态订阅。

!!! note "该订阅源仅 `next-dev` 提供"
    小红书订阅源只在 `next-dev` 分支存在；`master` / `next` 不含该订阅源。分支差异见 [版本与分支](../deploy/branches.md)。本页内容均属 `[next-dev]`。

!!! warning "需要 cookie"
    获取直播房间信息需要配置登录 cookie（至少 `a1`），未配置时直播订阅可能拿不到房间信息。

---

## 订阅

```
/watch -s xhs <用户ID>
```

用户 ID 是小红书主页的标识。`<用户ID>` 也可以填**本群已订阅**用户的昵称 / 小红书号 / red_id，框架会从本地已订阅缓存中解析出对应 UID（`ResolveSubscribedID`；只能匹配本群已订阅且缓存了信息的用户，重名会报错）。

```bash
# 订阅直播（默认类型）
/watch -s xhs <用户ID>

# 订阅图文/视频动态
/watch -s xhs -t news <用户ID>

# 取消订阅
/unwatch -s xhs <用户ID>
```

支持 `live` 与 `news` 两种类型；不指定 `-t` 时默认 `live`（`live` 的字母序在 `news` 之前）。

---

## 配置

```yaml
xhs:
  cookies:
    a1: ""            # 直播房间信息需要（至少填这个）
    web_session: ""   # 可选
    webId: ""         # 可选
  interval: 15s       # 轮询间隔，示例值
  onlyOnlineNotify: false
```

| 字段 | 说明 |
|------|------|
| `cookies.a1` | 小红书 `a1` cookie |
| `cookies.web_session` | 可选，登录态 |
| `cookies.webId` | 可选 |
| `interval` | 站点级轮询间隔：优先 `xhs.interval`，未配置回退 `concern.emitInterval`，两者都缺失时 EmitQueue 用内置默认 5s |
| `onlyOnlineNotify` | 是否只推送 Bot 在线期间的内容 |

---

## 推送模板

直播模板：`notify.group.xhs.live.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `living` | bool | 是否正在直播 |
| `name` | string | 昵称 |
| `title` | string | 直播标题 |
| `url` | string | 直播间链接 |
| `cover` | string | 封面 |

data map 还提供 `uid`、`display_member_count`、`display_praise_count`、`display_viewer_count`。

动态模板：`notify.group.xhs.news.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `name` | string | 昵称 |
| `title` | string | 标题 |
| `desc` | string | 正文描述 |
| `cover` | string | 封面 |
| `pictures` | []string | 图片列表 |
| `url` | string | 笔记链接 |
| `is_video` | bool | 是否为视频笔记 |

---

## 下一步

- [订阅源总览](index.md)
- [推送模板](../template/notify-tmpl.md)
- [版本与分支](../deploy/branches.md)
