# 抖音

DDBOT-WSa 新增了对抖音直播订阅的支持（测试阶段）。

!!! warning "测试功能"
    抖音对人机验证要求较高，配置较繁琐，稳定性不保证。

---

## 订阅

```
/watch -s douyin <抖音号>
```

`master` 只支持 `live`（直播）；`news`（视频）为 `[next]` 新增。不指定 `-t` 时默认 `live`。

```bash
# 订阅抖音直播
/watch -s douyin <抖音号>

# 订阅抖音视频（news，[next]）
/watch -s douyin -t news <抖音号>

# 取消订阅
/unwatch -s douyin <抖音号>
```

---

## 配置

抖音需要从浏览器获取 cookie 来通过人机验证：

```yaml
douyin:
  interval: 30s
  userAgent:
  acSignature:      # __ac_signature cookie
  acNonce:          # __ac_nonce cookie
  # 以下配置 [next] 起才被读取：
  sessionId:        # 会话 cookie，视频/用户信息接口用
  onlyOnlineNotify: false
```

| 字段 | 说明 |
|------|------|
| `interval` | 检测间隔；未配置默认 30s |
| `userAgent` | 浏览器 UA，**必须与获取 cookie 时一致** |
| `acSignature` | `__ac_signature` cookie |
| `acNonce` | `__ac_nonce` cookie |
| `sessionId` | `[next]` 的 `sessionId` cookie，`master` 无此键 |
| `onlyOnlineNotify` | `[next]` 是否不推送 Bot 离线期间的内容 |

`acSignature` 或 `acNonce` **任一**留空时，抖音订阅模块不会启动（`Stop=true`，日志提示「抖音Cookie未设置，将关闭抖音推送功能」）；`sessionId` 留空不影响启动。`master` 只有 `live`，没有 `sessionId` 键，也没有 `news` 类型（`git show origin/master:lsp/douyin/init.go`、`origin/master:lsp/douyin/concern.go`）。

详见 [抖音配置](../config/douyin.md)。

---

## 人机验证

抖音对人机验证非常敏感：

- cookie 过期需重新获取
- UA 变化导致验证失败
- 频繁请求触发验证码

`master` 检测到人机验证时会在**日志**中输出错误提示：字符串「检测到人机验证」由 `ParseUserInfoResp` 作为 error 返回，最终由 `freshLiveInfo` 以「查找用户信息失败：检测到人机验证…」记入日志（`fresh` 循环把该错误 `continue` 掉，因此不会走到 `watchCore error`）；代码中没有把该提示推送到群/私聊的路径。`[next]` / `[next-dev]` 已无该字符串。

---

## 推送模板

模板名：`notify.group.douyin.live.tmpl`

| 变量 | 类型 | 含义 | 分支 |
|------|------|------|------|
| `living` | bool | 是否正在直播 | master |
| `name` | string | 主播昵称 | master |
| `url` | string | 直播间链接 | master |
| `title` | string | 直播标题 | `[next]` |
| `cover` | string | 封面或头像 | `[next]` |

`master` 的 data map 只有 `uid` / `name` / `roomId` / `living` / `url`，模板本身也只用到 `living` / `name` / `url`。

视频（news，`[next]`）模板名：`notify.group.douyin.news.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `name` | string | 作者昵称 |
| `date` | string | 发布时间 |
| `desc` | string | 视频描述 |
| `cover` | string | 封面 |
| `url` | string | 视频链接 |

---

## 常见问题

### 订阅没启动

- 检查 `acSignature` 和 `acNonce` 是否都填了
- 看日志是否有「抖音Cookie未设置」提示

### 一直触发人机验证

- 调大 `interval`（如 60s）
- 确认 UA 与 cookie 一致
- cookie 可能已过期，重新获取

---

## 下一步

- [抖音配置](../config/douyin.md) -- 完整配置与 cookie 获取
- [订阅源总览](index.md)
- [FAQ](../faq.md)
