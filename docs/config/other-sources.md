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
    created: true
    image: false
  nameStrategy: "name"
```

[TwitCasting](https://twitcasting.tv/) 需要注册一个 App 获取凭证。

### 获取 clientId / clientSecret

1. 访问 [TwitCasting Developer](https://twitcasting.tv/developer.php)
2. 新增一个 App，填入所需资料
3. 获取 `clientId` 和 `clientSecret`

详见 [TwitCasting API 文档](https://apiv2-doc.twitcasting.tv/#registration)。

### 字段说明

| 字段 | 说明 |
|------|------|
| `clientId` / `clientSecret` | App 凭证，必填才能启用 |
| `broadcaster.title` | 是否推送标题（有风控风险，默认 false） |
| `broadcaster.created` | 是否推送开播时间（默认 true） |
| `broadcaster.image` | 是否推送直播封面（墙内无法获取，建议有代理才开启） |
| `nameStrategy` | 名称显示：`name`（用户名）/ `userid`（用户ID）/ `both`（两者） |

!!! tip "风控"
    日文字符多的标题/名称容易触发风控，建议保守配置。

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

```
/watch -s youtube UCvEX2UICvFAa_T6pqizC20g
/watch -s youtube @SomeChannel
```

---

## 微博

微博**无需配置 cookie**，DDBOT 会自动获取游客 cookie 并每小时刷新。

```yaml
# 微博无需在 application.yaml 中配置
# 直接订阅即可
```

订阅示例：

```
/watch -s weibo 5462373877
```

!!! note "游客 cookie"
    DDBOT 启动时会自动获取微博游客 cookie，失败会重试 3 次，之后每小时刷新一次。无需手动干预。

---

## 斗鱼

无需配置，直接订阅：

```
/watch -s douyu 6655
```

参数为斗鱼房间号。

---

## 虎牙

无需配置，直接订阅：

```
/watch -s huya xiaoleyan
```

参数为虎牙主播 ID（URL 里的 `https://www.huya.com/<主播ID>`）。

---

## ACFun

无需配置，直接订阅：

```
/watch -s acfun <主播ID>
```

---

## 各订阅源对照表

| 订阅源 | site | 类型 | 是否需配置 | ID 类型 |
|--------|------|------|-----------|---------|
| B 站 | `bilibili` | live / news | 推荐（账号或 cookie） | UID |
| 斗鱼 | `douyu` | live | 否 | 房间号 |
| 虎牙 | `huya` | live | 否 | 主播 ID |
| ACFun | `acfun` | live | 否 | 主播 ID |
| YouTube | `youtube` | live / news | 视情况（代理） | channel ID / @handle |
| 微博 | `weibo` | news | 否 | 用户 UID |
| TwitCasting | `twitcasting` | live | 是（App 凭证） | 用户 ID |
| 推特 | `twitter` | news | 是（镜像） | 用户名 |
| 抖音 | `douyin` | live | 是（cookie） | 抖音号 |

各订阅源的详细说明见 [订阅源](../sources/index.md) 对应页面。

---

## 下一步

- [订阅源总览](../sources/index.md) -- 各订阅源详细说明
- [代理与图片池](proxy-imagepool.md) -- YouTube 等需代理的场景
- [命令手册 - 订阅命令](../commands/subscribe.md) -- `/watch` 用法
