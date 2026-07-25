# B 站配置

B 站是 DDBOT 最常用的订阅源，配置项也最多。本页讲 `bilibili` 段的所有字段。

---

## 配置示例

```yaml
bilibili:
  SESSDATA:                  # B 站 cookie
  bili_jct:                  # B 站 cookie
  qrlogin: true              # 扫码登录
  interval: 25s              # 检测间隔
  imageMergeMode: "auto"     # 图片合并模式
  hiddenSub: false           # 悄悄关注
  unsub: false               # 自动取关
  minFollowerCap: 0          # 最低粉丝数门槛
  disableSub: false          # 禁止自动关注
  onlyOnlineNotify: false    # 仅推送在线期间的更新
  autoParsePosts: false      # 自动解析专栏
```

---

## 登录方式

B 站支持三种登录方式，**任选一种**：

### 方式一：扫码登录（推荐测试用）

```yaml
bilibili:
  qrlogin: true
  # SESSDATA 和 bili_jct 留空
```

启动后日志会显示二维码，用 B 站 App 扫码登录。cookie 失效时清空 `SESSDATA` 和 `bili_jct` 重启即可再次扫码。

!!! note "扫码登录的局限"
    - 不填账号密码，仅扫码
    - cookie 失效后需重新扫码

### 方式二：cookie

从浏览器获取 B 站登录后的 cookie，填入：

```yaml
bilibili:
  SESSDATA: "你的SESSDATA"
  bili_jct: "你的bili_jct"
  qrlogin: false
```

??? question "如何获取 cookie"
    1. 浏览器登录 [bilibili.com](https://www.bilibili.com)
    2. 按 F12 打开开发者工具 -> Application（应用）-> Cookies -> `https://www.bilibili.com`
    3. 找到 `SESSDATA` 和 `bili_jct` 两个值，复制填入配置

    !!! danger "cookie 等价于账号凭证"
        `SESSDATA` 和 `bili_jct` 等价于你的 B 站账号凭证，**绝对不要透露给他人或上传到公开平台**，否则账号会被盗。

    !!! warning "获取 cookie 后不要点退出登录"
        点退出会让 cookie 失效。可用浏览器「清除历史记录」功能，或在隐私窗口里登录获取。

### 方式三：账号密码

```yaml
bilibili:
  account: "你的B站账号"
  password: "你的B站密码"
```

!!! warning "目前不可用"
    B 站账号密码自动登录目前处于失效状态，建议使用扫码或 cookie 方式。

---

## 字段详解

### `interval` - 检测间隔

```yaml
bilibili:
  interval: 25s
```

直播状态和动态的检测间隔。**过快可能导致 IP 被暂时封禁**，建议保持默认 25s。

### `imageMergeMode` - 图片合并

```yaml
bilibili:
  imageMergeMode: "auto"   # auto / only9 / off
```

动态含多张图片时是否合并成长图，减少刷屏：

| 模式 | 行为 |
|------|------|
| `auto` | 默认，存在较刷屏的图片时自动合并 |
| `only9` | 仅当恰好 9 张图片时合并 |
| `off` | 不合并 |

### `hiddenSub` - 悄悄关注

```yaml
bilibili:
  hiddenSub: false
```

订阅时是否用「悄悄关注」而非普通关注。默认 `false`。

### `unsub` - 自动取关

```yaml
bilibili:
  unsub: false
```

取消订阅时是否自动取关该 UP 主。

!!! warning "多 bot 共用账号时不要开"
    如果你的 B 站账号被多个 bot 同时使用，开启自动取关会导致其他 bot 的推送丢失。默认 `false`。

### `minFollowerCap` - 粉丝数门槛

```yaml
bilibili:
  minFollowerCap: 0
```

订阅的 B 站用户需要满足的最低粉丝数。

| 值 | 行为 |
|----|------|
| `0`（默认） | 不限制 |
| `-1` | 无限制 |
| 正整数 | 订阅时若粉丝数不足则拒绝 |

主要用于过滤无意义的测试订阅（防止用直播间 ID 误订阅）。

### `disableSub` - 禁止自动关注

```yaml
bilibili:
  disableSub: false
```

| 值 | 行为 |
|----|------|
| `false`（默认） | 订阅时自动关注该 UP 主 |
| `true` | 禁止 DDBOT 去关注，只能订阅账号已关注的用户 |

开启后，要订阅新用户需手动在 B 站关注，然后 DDBOT 才能订阅。

### `onlyOnlineNotify` - 仅在线期间推送

```yaml
bilibili:
  onlyOnlineNotify: false
```

| 值 | 行为 |
|----|------|
| `false`（默认） | 推送 BOT 离线期间积攒的动态和直播 |
| `true` | 不推送离线期间的更新 |

### `autoParsePosts` - 自动解析专栏

```yaml
bilibili:
  autoParsePosts: false
```

| 值 | 行为 |
|----|------|
| `false`（默认） | 专栏动态按普通动态推送 |
| `true` | 自动解析专栏内容，发送专栏正文而非动态卡片 |

---

## BOT 使用 B 站账号的说明

订阅 B 站用户后，配置的 B 站账号会**自动关注**该用户（除非 `disableSub: true`）。BOT 会用到账号的以下功能：

- 关注用户 / 取消关注用户
- 查看关注列表

!!! tip "建议用小号"
    推荐使用新注册的 B 站小号，避免主号被风控。刚注册的号关注时可能提示「账户异常」，这是 B 站侧限制，无法在 DDBOT 解决。

---

## 订阅数量建议

| 场景 | 建议订阅数 |
|------|-----------|
| 不配置账号 | ≤ 5 个（否则延迟大幅上升） |
| 配置账号 | 最多 2000（已验证） |

大规模订阅务必配置账号，且 `interval` 不要调得太小。

---

## 常见问题

### B 站登录失败

- 检查 cookie 是否复制完整（可能很长）
- 确认 cookie 没有过期
- 确认没有误点 B 站「退出登录」
- 改用扫码登录

### 关注失败提示「账户异常」

B 站对新号或异常号有限制，建议：

- 用小号挂机一段时间后再用
- 手动在 B 站关注，然后 `disableSub: true`

详见 [FAQ](../faq.md)。

---

## 下一步

- [B 站订阅源说明](../sources/bilibili.md) -- live/news 类型、过滤、专栏
- [命令手册 - 订阅命令](../commands/subscribe.md) -- `/watch`、`/config`
- [FAQ](../faq.md) -- B 站相关排障
