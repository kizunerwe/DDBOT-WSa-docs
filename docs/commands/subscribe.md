# 订阅命令

`/watch`、`/unwatch`、`/list` 是 DDBOT 最常用的命令，用于管理订阅。

---

## `/watch` - 订阅推送

| 默认权限 | 默认启用 | 可禁用 |
|---------|---------|--------|
| 群管理员 / BOT 管理员 | 是 | 是 |

**与 `/unwatch` 共享权限。**

### 格式

```
/watch [-s <网站>] [-t <类型>] <id>
```

| 选项 | 默认 | 说明 |
|------|------|------|
| `-s` / `--site` | `bilibili` | 订阅网站 |
| `-t` / `--type` | `live` | 订阅类型 |

### 例子

订阅 B 站直播：

```
/watch 97505
```

订阅 B 站动态：

```
/watch -t news 97505
```

订阅斗鱼 6655 直播间：

```
/watch -s douyu 6655
```

订阅 YouTube 频道直播：

```
/watch -s youtube UCvEX2UICvFAa_T6pqizC20g
```

订阅 YouTube 频道视频：

```
/watch -s youtube -t news UCvEX2UICvFAa_T6pqizC20g
```

订阅虎牙直播：

```
/watch -s huya xiaoleyan
```

订阅微博动态：

```
/watch -s weibo 5462373877
```

订阅推特：

```
/watch -s twitter elonmusk
```

订阅抖音直播：

```
/watch -s douyin <抖音号>
```

### 私聊版本

私聊时加 `-g <群号>` 指定操作的群：

```
/watch -g 123456 -t news 97505
```

---

## `/unwatch` - 取消订阅

| 默认权限 | 默认启用 | 可禁用 |
|---------|---------|--------|
| 群管理员 / BOT 管理员 | 是 | 是 |

**与 `/watch` 共享权限。**

用法和 `/watch` 完全一样，把 `watch` 换成 `unwatch` 即可。

### 例子

取消订阅 B 站直播：

```
/unwatch 97505
```

取消订阅 B 站动态：

```
/unwatch -t news 97505
```

取消订阅 YouTube 视频：

```
/unwatch -s youtube -t news UCvEX2UICvFAa_T6pqizC20g
```

### 私聊版本

```
/unwatch -g 123456 -t news 97505
```

---

## `/list` - 查看订阅列表

| 默认权限 | 默认启用 | 可禁用 |
|---------|---------|--------|
| 所有人 | 是 | 是 |

### 格式

```
/list [-s <网站>]
```

### 例子

查看所有订阅：

```
/list
```

只看 B 站订阅：

```
/list -s bilibili
```

### 私聊版本

查看指定群的订阅：

```
/list -g 123456
```

查看指定群的 B 站订阅：

```
/list -g 123456 -s bilibili
```

---

## 各订阅源的 id

| 订阅源 | site | id 来源 |
|--------|------|--------|
| B 站 | `bilibili` | UP 主 UID（空间主页 URL 里的数字） |
| 斗鱼 | `douyu` | 房间号 |
| 虎牙 | `huya` | 主播 ID（URL `huya.com/<id>`） |
| ACFun | `acfun` | 主播 ID |
| YouTube | `youtube` | channel ID（`UC...`）或 `@handle` |
| 微博 | `weibo` | 用户 UID（`weibo.com/u/<uid>`） |
| TwitCasting | `twitcasting` | 用户 ID |
| 推特 | `twitter` | 用户名（不带 @） |
| 抖音 | `douyin` | 抖音号 |

详细说明见 [订阅源](../sources/index.md) 各页面。

---

## 订阅后的进阶配置

订阅后可用 `/config` 定制推送行为，详见 [配置命令](config-cmd.md)：

- `/config at_all <id> on` -- 直播推送时 @全体
- `/config filter not_type <id> 转发` -- 不推送转发动态
- `/config offline_notify <id> on` -- 下播也推送
- `/config title_notify <id> on` -- 标题变更时重新推送

---

## 常见问题

### 订阅失败

- B 站：检查是否配置账号/cookie，或粉丝数是否满足 `minFollowerCap`
- 抖音：检查 `acSignature`/`acNonce` 是否填写
- YouTube：国内需配代理

### 订阅了但没推送

- 检查 `/config filter` 是否设了过滤
- 检查日志是否有 `notify` 记录和发送失败
- 检查 BOT 是否被禁言

详见 [FAQ](../faq.md)。

---

## 下一步

- [配置命令](config-cmd.md) -- 定制推送行为
- [订阅源](../sources/index.md) -- 各订阅源详细说明
- [命令格式](format.md) -- 命令语法
