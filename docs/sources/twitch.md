# Twitch

[Twitch](https://www.twitch.tv/) 直播订阅，需要注册 App 获取凭证。

!!! note "该订阅源仅 `next-dev` 提供"
    Twitch 订阅源只在 `next-dev` 分支存在；`master` / `next` 不含该订阅源。分支差异见 [版本与分支](../deploy/branches.md)。本页内容均属 `[next-dev]`。

---

## 前置准备

1. 访问 [Twitch Developer Console](https://dev.twitch.tv/console/apps)
2. 新增一个应用，获取 `clientId` 和 `clientSecret`
3. 填入 `application.yaml` 的 `twitch` 段

缺少整个 `twitch` 配置段，或 `clientId` / `clientSecret` **任一**为空时，Twitch 订阅模块都不会启动。

---

## 订阅

```
/watch -s twitch <登录名>
```

登录名是频道地址里的字符串，例如 `https://www.twitch.tv/shroud` 的登录名是 `shroud`。

```bash
# 订阅直播
/watch -s twitch shroud

# 取消订阅
/unwatch -s twitch shroud
```

只支持 `live` 类型。

---

## 配置

```yaml
twitch:
  clientId:               # Twitch 应用的 Client ID
  clientSecret:           # Twitch 应用的 Client Secret
  interval: 30s           # 轮询间隔，未配置默认 30s
  onlyOnlineNotify: false
```

| 字段 | 说明 |
|------|------|
| `clientId` / `clientSecret` | App 凭证，必填（任一为空即不启动） |
| `interval` | 轮询间隔，未配置默认 30s（直接读 `twitch.interval`，不回退 `concern.emitInterval`） |
| `onlyOnlineNotify` | 是否不推送 Bot 离线期间的直播 |

---

## 推送模板

模板名：`notify.group.twitch.live.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `living` | bool | 是否正在直播 |
| `login` | string | 频道登录名 |
| `name` | string | 显示名称 |
| `title` | string | 直播标题 |
| `game_name` | string | 游戏/分类名称 |
| `viewer_count` | int | 观看人数 |
| `thumbnail` | string | 封面图 URL |
| `profile_image` | string | 头像 URL |
| `offline_image` | string | 离线封面 URL |

data map 还提供 `title_changed`、`live_changed` 两个 bool。

---

## 下一步

- [订阅源总览](index.md)
- [推送模板](../template/notify-tmpl.md)
- [版本与分支](../deploy/branches.md)
