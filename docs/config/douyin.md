# 抖音配置

DDBOT-WSa 新增了对抖音订阅的支持（测试阶段）：`master` 只有直播，`[next]` 起新增动态（news）与 `sessionId`。本页讲 `douyin` 段配置。

!!! warning "测试功能"
    抖音订阅处于测试阶段，抖音对人机验证要求较高，配置较繁琐，稳定性不保证。

---

## 配置示例

```yaml
douyin:
  acSignature:
  acNonce:
  userAgent:
  interval: 30s
  sessionId:                # [next]，master 无此键
  onlyOnlineNotify: false   # [next]
```

---

## 前置准备

抖音接口用 `__ac_signature`、`__ac_nonce` 两个 cookie 通过人机验证；`sessionid`（对应下方 `sessionId`，`[next]`）用于获取用户/直播信息。都需要从浏览器获取。

### 获取 cookie

1. 浏览器访问 [www.douyin.com](https://www.douyin.com) 并登录
2. F12 -> Application -> Cookies -> `https://www.douyin.com`
3. 找到 `__ac_signature`、`__ac_nonce` 两个值；`[next]` 另需 `sessionid`（填入 `sessionId`）
4. 同时记下你浏览器的 User-Agent（访问 [httpbin.org/user-agent](https://httpbin.org/user-agent)）

---

## 字段详解

### `interval` - 检测间隔

```yaml
douyin:
  interval: 30s
```

直播状态检测间隔（`[next]` 起同时用于动态）。**过快可能触发人机验证**，建议保持 30s 以上。

### `userAgent` - 浏览器 UA

```yaml
douyin:
  userAgent: "Mozilla/5.0 ..."
```

访问抖音接口时使用的 User-Agent。

!!! warning "建议与获取 cookie 时的 UA 一致"
    `__ac_signature` 和 `__ac_nonce` 与 UA 相关，UA 不一致时验证容易失败（抖音侧行为，非本仓库代码可验证）。建议填入你获取 cookie 时所用的浏览器 UA。

### `acSignature` - 签名 cookie

```yaml
douyin:
  acSignature: "你的__ac_signature"
```

对应抖音 cookie 里的 `__ac_signature` 字段。

### `acNonce` - 随机数 cookie

```yaml
douyin:
  acNonce: "你的__ac_nonce"
```

对应抖音 cookie 里的 `__ac_nonce` 字段。

### `sessionId` - 会话 cookie（`[next]`）

```yaml
douyin:
  sessionId: "你的sessionid"
```

对应抖音 cookie 里的 `sessionid` 字段，由 `lsp/douyin/init.go` 读取 `douyin.sessionId`，自动生成的 `application.yaml` 也含此项。缺失时不会把模块标记为 `Stop`，但部分接口可能取不到数据。

`master` 的 `lsp/douyin/init.go` 不读取该键，生成的配置里也没有此项。

### `onlyOnlineNotify` - 仅在线期间推送（`[next]`）

```yaml
douyin:
  onlyOnlineNotify: false
```

| 值 | 行为 |
|----|------|
| `false`（默认） | 推送 BOT 离线期间积攒的动态和直播 |
| `true` | 不推送离线期间的更新 |

`master` 不读取该键，生成配置里也没有此项。

!!! danger "不填会怎样"
    `acSignature` 或 `acNonce` 留空时，DDBOT 会把订阅模块标记为 `Stop`，抖音订阅功能不启动，日志会提示；`[next]` 的 `sessionId` 缺失不会触发 `Stop`。

---

## 订阅抖音直播 / 动态

配置好后用 `/watch` 订阅：

```text
/watch -s douyin <抖音号或直播间号>       # 默认 live（直播）
/watch -s douyin -t news <抖音号>         # news（动态，[next]）
```

`master` 的 `Types()` 只返回 `{Live}`；`[next]` 起返回 `{Live, News}`。

---

## 人机验证提示

抖音对人机验证非常敏感，常见情况：

- cookie 过期后需重新获取
- UA 变化导致验证失败
- 频繁请求触发验证码

`master` 检测到人机验证时会返回错误（`lsp/douyin/getUserInfo.go`），记录到日志。此时需要：

1. 重新访问 douyin.com 获取新的 cookie
2. 确认 UA 一致
3. 更新配置后重启

`[next]` 起该检测代码已不在 `lsp/douyin` 中。

---

## 常见问题

### 抖音订阅没启动

- 检查 `acSignature` 和 `acNonce` 是否都填了
- 看日志是否出现「抖音Cookie未设置，将关闭抖音推送功能」

### 一直触发人机验证

- 调大 `interval`（如 60s）
- 确认 UA 与 cookie 一致
- cookie 可能已过期，重新获取

### 直播状态不准

抖音接口返回有时延迟，属正常现象。

---

## 下一步

- [抖音订阅源说明](../sources/douyin.md) -- 订阅机制
- [FAQ](../faq.md) -- 排障
