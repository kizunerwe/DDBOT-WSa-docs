# 抖音配置

DDBOT-WSa 新增了对抖音直播订阅的支持（测试阶段）。本页讲 `douyin` 段配置。

!!! warning "测试功能"
    抖音订阅处于测试阶段，抖音对人机验证要求较高，配置较繁琐，稳定性不保证。

---

## 配置示例

```yaml
douyin:
  interval: 30s
  userAgent:
  acSignature:
  acNonce:
```

---

## 前置准备

抖音接口需要 `__ac_signature` 和 `__ac_nonce` 两个 cookie 来通过人机验证，你需要从浏览器获取。

### 获取 cookie

1. 浏览器访问 [www.douyin.com](https://www.douyin.com) 并登录
2. F12 -> Application -> Cookies -> `https://www.douyin.com`
3. 找到 `__ac_signature` 和 `__ac_nonce` 两个值
4. 同时记下你浏览器的 User-Agent（访问 [httpbin.org/user-agent](https://httpbin.org/user-agent)）

---

## 字段详解

### `interval` - 检测间隔

```yaml
douyin:
  interval: 30s
```

直播状态检测间隔。**过快可能触发人机验证**，建议保持 30s 以上。

### `userAgent` - 浏览器 UA

```yaml
douyin:
  userAgent: "Mozilla/5.0 ..."
```

访问抖音接口时使用的 User-Agent。

!!! warning "必须与获取 cookie 时的 UA 一致"
    `__ac_signature` 和 `__ac_nonce` 是和 UA 绑定的，UA 不一致会导致验证失败。填入你获取 cookie 时所用的浏览器 UA。

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

!!! danger "不填会怎样"
    `acSignature` 或 `acNonce` 留空时，DDBOT 会把订阅模块标记为 `Stop`，抖音订阅功能不启动，日志会提示。

---

## 订阅抖音直播

配置好后用 `/watch` 订阅：

```
/watch -s douyin <抖音号或直播间号>
```

抖音只支持 `live` 类型（直播）。

---

## 人机验证提示

抖音对人机验证非常敏感，常见情况：

- cookie 过期后需重新获取
- UA 变化导致验证失败
- 频繁请求触发验证码

DDBOT 在检测到人机验证时会**在日志和推送中提示**，此时需要：

1. 重新访问 douyin.com 获取新的 cookie
2. 确认 UA 一致
3. 更新配置后重启

---

## 常见问题

### 抖音订阅没启动

- 检查 `acSignature` 和 `acNonce` 是否都填了
- 看日志是否有 `Stop` 相关提示

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
