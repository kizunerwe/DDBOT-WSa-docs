# 配置命令

`/config` 命令用于定制单个订阅的推送行为。

| 默认权限 | 默认启用 | 可禁用 |
|---------|---------|--------|
| 群管理员 / BOT 管理员 | 是 | 是 |

---

## 配置 @全体成员

**同一个主播每 2 小时内只会 @全体成员一次。**

```
/config at_all <id> on     # 开启
/config at_all <id> off    # 关闭
```

例：

```
/config at_all 97505 on
```

!!! warning "需把 BOT 设为群管理员"
    否则配置了也无法 @全体成员。@全体次数用尽时也不会生效。

---

## 配置 @特定成员

当 @全体无法生效时（未设置、未管理员、次数用完），改为 @特定成员。

### 添加要 @的成员

```
/config at <id> add <QQ号...>
```

可一次填多个 QQ 号：

```
/config at 97505 add 10000 10001
```

### 删除成员

```
/config at 97505 remove 10000 10001
```

### 查看配置的成员

```
/config at 97505 show
```

### 清空

```
/config at 97505 clear
```

---

## 配置直播间标题变更推送

主播改标题时重新推送：

```
/config title_notify <id> on
```

---

## 配置下播推送

主播下播时也推送一条：

```
/config offline_notify <id> on
```

---

## 配置增强推送（`[next]`）

从 `next` 分支起提供（`master` 无此子命令）。开启后推送会附带更完整的扩展信息（默认不启用）：

```
/config extend_notify <id> on
/config extend_notify <id> off
```

!!! note "仅部分订阅源支持"
    目前只有 **B 站直播**实现增强信息；`[next-dev]` 另有**小红书直播**。其它订阅源开启后没有实际效果。

---

## 配置动态过滤（B 站）

### 同一时间只能设置一种过滤器

`master` 上只能同时设置一种过滤器（类型或关键字），多次设置以最后一次为准。

只推送指定类型的动态：

```
/config filter type <id> <类型...>
```

例：只推送图片、文字、专栏、投稿：

```
/config filter type 97505 图片 文字 专栏 投稿
```

不推送指定类型的动态：

```
/config filter not_type <id> <类型...>
```

例：不推送转发：

```
/config filter not_type 97505 转发
```

按关键字过滤（只推送包含任意关键字的动态）：

```
/config filter text <id> <关键字...>
```

例：

```
/config filter text 97505 关键字1 关键字2
```

### `[next]`：多种过滤器可同时存在

从 `next` 分支起，类型规则（`type` / `not_type`）与关键字规则（`text` / `not_text`）可以同时保留：多条规则之间取 **AND**（每条都要通过），同一条规则内的多个值取 **OR**。

不推送含指定关键字的动态：

```
/config filter not_text <id> <关键字...>
```

例：

```
/config filter not_text 97505 抽奖 广告
```

### 查看当前过滤器

```
/config filter show <id>
```

### 清空过滤器

```
/config filter clear <id>
```

### 支持的动态类型

B 站动态类型繁多，DDBOT 支持以下常见类型：

| 类型 | 说明 |
|------|------|
| `专栏` | 专栏文章 |
| `转发` | 转发动态 |
| `投稿` | 视频投稿 |
| `文字` | 纯文字动态 |
| `图片` | 带图动态 |
| `音频` | 带音频动态（`[next-dev]`） |
| `直播分享` | 直播间分享 |

也可以填动态类型数字（如 `4098`、`4308` 等），详见 B 站动态类型文档。

---

## 配置微博 Cookie/SUB 告警（`[next-dev]`）

### 群内开关

控制本群是否接收微博 Cookie 失效 / SUB 刷新等告警通知（**需要 BOT 管理员权限**，不指定参数时默认查看状态）：

```
/config weibo_alert show         # 查看本群状态
/config weibo_alert true         # 开启本群告警（也接受 1 / on / enable）
/config weibo_alert false        # 关闭本群告警（也接受 0 / off / disable）
```

### 私聊查看告警目标

私聊下 `/config weibo_alert` 只用于**查看**当前真正生效的告警发送对象（**需要 BOT 管理员权限**）：

```
/config weibo_alert show         # 列出实际发送对象
```

输出分两部分：**BOT 管理员（私聊）** 与 **已启用告警的群**（通过群内 `/config weibo_alert true` 开启的群）。

!!! note "早期版本的 `set` / `clear` 已移除"
    `[next-dev]` 曾提供 `/config weibo_alert set -g <群号>` / `set -q <QQ号>` / `clear` 登记一个「自定义目标」，但那个目标只写进本地数据库、投递逻辑从不读取，**登记了也不会改变告警去向**；同时子命令的 `-g` 与根级 `-g` 短名冲突，群目标实际登记不上。这套写入路径已删除，现在只有 `show`。要控制去向，用群内开关（`true`/`false`）与 BOT 管理员列表。

---

## 私聊版本

私聊时加 `-g <群号>`：

```
/config -g 123456 at_all 97505 on
/config -g 123456 filter not_type 97505 转发
```

---

## 完整示例

订阅示例 UP 主的 B 站并完整配置：

```bash
# 1. 订阅直播
/watch 97505

# 2. 订阅动态
/watch -t news 97505

# 3. 不推送转发动态（避免刷屏）
/config filter not_type 97505 转发

# 4. 直播推送时 @全体（需 BOT 为群管理员）
/config at_all 97505 on

# 5. 开启下播推送
/config offline_notify 97505 on

# 6. @全体失效时 @特定人
/config at 97505 add 10000 10001
```

---

## 下一步

- [订阅命令](subscribe.md) -- `/watch`、`/list`
- [权限命令](permission.md) -- `/grant`、`/enable`
- [订阅源 - B 站](../sources/bilibili.md) -- B 站动态类型详解
