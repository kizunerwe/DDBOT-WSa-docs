# 其它订阅源配置

本页讲 TwitCasting、YouTube、微博、斗鱼、虎牙、ACFun 这些订阅源的配置。多数订阅源无需额外配置即可使用，YouTube/微博有可选优化项。

---

## TwitCasting

```yaml
twitcasting:
  clientId: abc
  clientSecret: xyz
  broadcaster:
    title: false
    created: false
    image: false
  nameStrategy: "name"
```

[TwitCasting](https://twitcasting.tv/) 需要 API 凭证（App 的 `clientId` / `clientSecret`）；代码只消费这两个值，并在启动时调用 API 校验凭证是否有效（无效则订阅源不启动）。凭证的申请流程见 [开发者页面](https://twitcasting.tv/developer.php)（外部流程，本仓库代码无法验证）。

`[next]` 起该段会出现在自动生成的配置里；`master` 生成文件没有此段，需要时手动添加（键本身在 `master` 也会被读取）。

### 字段说明

| 字段 | 说明 |
|------|------|
| `clientId` / `clientSecret` | App 凭证，必填才能启用 |
| `broadcaster.title` | 是否推送标题，默认 `false` |
| `broadcaster.created` | 是否推送开播时间，默认 `false`（需显式设 `true`） |
| `broadcaster.image` | 是否推送直播封面（墙内无法获取，建议有代理才开启） |
| `nameStrategy` | 名称显示：`name`（用户名）/ `userid`（用户ID）/ `both`（两者） |


---

## YouTube

YouTube **无需额外配置**即可订阅，但有以下注意点：

### 直连与代理

- 能直连 YouTube（如海外服务器）：直接用
- 无法直连（国内）：需配置代理，见 [代理与图片池](proxy-imagepool.md)

### 兼容新老 ID

DDBOT-WSa 兼容 YouTube 新老 channel ID 订阅：

- 老 ID：`UCxxxxxxxx`
- 新 ID：`@handle`

订阅示例：

```text
/watch -s youtube UCvEX2UICvFAa_T6pqizC20g
/watch -s youtube @SomeChannel
```

### `onlyOnlineNotify`（`[next]`）

`[next]` 起生成 `youtube.onlyOnlineNotify`（默认 `true`，键缺失时代码默认 `false`）；`master` 不读取该键。

---

## 微博

微博**无需配置 cookie**，DDBOT 会自动获取游客 cookie 并每小时刷新。

```yaml
# 微博无需在 application.yaml 中配置
# 直接订阅即可
```

订阅示例：

```text
/watch -s weibo 5462373877
```

!!! note "游客 cookie"
    DDBOT 启动时会自动获取微博游客 cookie，失败会重试 3 次，之后每小时刷新一次。无需手动干预。

`[next]` 起新增 `weibo.mode`、`weibo.interval`、`weibo.sub`、`weibo.qrlogin`、`weibo.onlyOnlineNotify` 等配置项；`master` 只有游客 cookie 模式，不读取这些键。字段含义见 [配置总览](index.md)。

---

## 斗鱼

无需配置，直接订阅：

```text
/watch -s douyu 6655
```

参数为斗鱼房间号。

---

## 虎牙

无需配置，直接订阅：

```text
/watch -s huya xiaoleyan
```

参数为虎牙主播 ID（URL 里的 `https://www.huya.com/<主播ID>`）。

---

## ACFun

无需配置，直接订阅：

```text
/watch -s acfun <主播ID>
```

### `disableSub` - 禁止自动关注（`[next]`）

`[next]` 起可以配置 A 站账号（`acfun.account` / `password`），DDBOT 订阅动态时会自动关注该用户。设为 `true` 可禁止自动关注：

```yaml
acfun:
  disableSub: false   # 默认 false
```

禁止后，只能订阅你已在 A 站手动关注的用户：订阅未关注的用户会直接失败并提示「该用户未在关注列表内」；周期同步关注列表时也不会自动关注，未关注的目标会被跳过并记录日志。

!!! note "生效范围"
    订阅命令的自动关注只在 `ctype == News`（动态）分支处理；但每小时一次的 `SyncSub` 会对**所有订阅类型**（含仅 live 的订阅）跳过自动关注。

`master` 不读取 `acfun.account` / `disableSub`，生成配置里也没有这些键。

---

## 各订阅源对照表

| 订阅源 | site | 类型 | 是否需配置 | ID 类型 |
|--------|------|------|-----------|---------|
| B 站 | `bilibili` | live / news | 推荐（账号或 cookie） | UID |
| 斗鱼 | `douyu` | live | 否 | 房间号 |
| 虎牙 | `huya` | live | 否 | 主播 ID |
| ACFun | `acfun` | live（`[next]` 起 live / news） | 否 | 主播 ID |
| YouTube | `youtube` | live / news | 视情况（代理） | channel ID / `@handle` |
| 微博 | `weibo` | news | 否 | 用户 UID |
| TwitCasting | `twitcasting` | live | 是（App 凭证） | 用户 ID |
| 推特 | `twitter` | news | 是（镜像） | 用户名 |
| 抖音 | `douyin` | live（`[next]` 起 live / news） | 是（cookie） | 抖音号 |

各订阅源的详细说明见 [订阅源](../sources/index.md) 对应页面。

---

## 下一步

- [订阅源总览](../sources/index.md) -- 各订阅源详细说明
- [代理与图片池](proxy-imagepool.md) -- YouTube 等需代理的场景
- [命令手册 - 订阅命令](../commands/subscribe.md) -- `/watch` 用法
