# 抖音

DDBOT-WSa 新增了对抖音直播订阅的支持（测试阶段）。

!!! warning "测试功能"
    抖音对人机验证要求较高，配置较繁琐，稳定性不保证。

---

## 订阅

```
/watch -s douyin <抖音号>
```

只支持 `live` 类型（直播）。

```bash
# 订阅抖音直播
/watch -s douyin <抖音号>

# 取消订阅
/unwatch -s douyin <抖音号>
```

---

## 配置（master）

抖音需要从浏览器获取 cookie 来通过人机验证：

```yaml
douyin:
  interval: 30s
  userAgent:
  acSignature:    # __ac_signature cookie
  acNonce:        # __ac_nonce cookie
```

| 字段 | 说明 |
|------|------|
| `interval` | 检测间隔，过快触发验证 |
| `userAgent` | 浏览器 UA，**必须与获取 cookie 时一致** |
| `acSignature` | `__ac_signature` cookie |
| `acNonce` | `__ac_nonce` cookie |

`acSignature` 或 `acNonce` 留空时，抖音订阅模块不会启动。

详见 [抖音配置](../config/douyin.md)。

!!! tip "next 分支新增 sessionId"
    `next` 分支额外需要 `sessionId` cookie，并重构了已推送推文的过滤逻辑：

    ```yaml
    douyin:
      acSignature:
      acNonce:
      sessionId:        # next 分支新增
      userAgent:
      interval: 30s
      onlyOnlineNotify: false
    ```

---

## 人机验证

抖音对人机验证非常敏感：

- cookie 过期需重新获取
- UA 变化导致验证失败
- 频繁请求触发验证码

DDBOT 检测到人机验证时会**在日志和推送中提示**，此时需重新获取 cookie 并更新配置。

---

## 推送模板

模板名：`notify.group.douyin.live.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `living` | bool | 是否正在直播 |
| `name` | string | 主播昵称 |
| `title` | string | 直播标题 |
| `url` | string | 直播间链接 |
| `cover` | string | 封面或头像 |

---

## 常见问题

### 订阅没启动

- 检查 `acSignature` 和 `acNonce` 是否都填了
- 看日志是否有 `Stop` 提示

### 一直触发人机验证

- 调大 `interval`（如 60s）
- 确认 UA 与 cookie 一致
- cookie 可能已过期，重新获取

---

## 下一步

- [抖音配置](../config/douyin.md) -- 完整配置与 cookie 获取
- [订阅源总览](index.md)
- [FAQ](../faq.md)
