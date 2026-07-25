# 命令速查

DDBOT 所有命令的速查表。详细用法见左侧子页面。

!!! info "命令前缀"
    默认前缀 `/`，可通过 `bot.commandPrefix` 或 `customCommandPrefix` 修改。本页示例均用 `/`。

!!! info "群聊 vs 私聊"
    - **群命令**：在群里发，部分需群管理员/BOT 管理员权限
    - **私聊命令**：私聊 BOT 发，需加 `-g <群号>` 指定操作的群
    - **管理员命令**：仅私聊，仅 BOT 管理员可用

---

## 订阅命令

| 命令 | 作用 | 权限 |
|------|------|------|
| `/watch [选项] <id>` | 订阅推送 | 群管理员/BOT管理员 |
| `/unwatch [选项] <id>` | 取消订阅 | 群管理员/BOT管理员 |
| `/list [选项]` | 查看订阅列表 | 所有人 |

```bash
/watch 97505                      # 订阅 B 站直播
/watch -t news 97505              # 订阅 B 站动态
/watch -s douyu 6655              # 订阅斗鱼
/watch -s weibo 5462373877        # 订阅微博
/watch -s twitter elonmusk        # 订阅推特
```

详见 [订阅命令](subscribe.md)。

---

## 配置命令

| 命令 | 作用 |
|------|------|
| `/config at_all <id> on\|off` | 开关 @全体成员 |
| `/config at <id> add\|remove\|show\|clear <QQ...>` | @特定成员 |
| `/config title_notify <id> on` | 标题变更推送 |
| `/config offline_notify <id> on` | 下播推送 |
| `/config filter type\|not_type\|text\|show\|clear <id> ...` | 动态过滤 |

```bash
/config at_all 97505 on
/config filter not_type 97505 转发
```

详见 [配置命令](config-cmd.md)。

---

## 权限命令

| 命令 | 作用 | 权限 |
|------|------|------|
| `/grant -c <命令> <QQ>` | 授予命令权限 | 群管理员/BOT管理员 |
| `/grant -d -c <命令> <QQ>` | 撤销命令权限 | 群管理员/BOT管理员 |
| `/grant -r GroupAdmin <QQ>` | 授予BOT群管理员 | BOT群管理员 |
| `/enable <命令>` | 启用命令 | BOT群管理员 |
| `/disable <命令>` | 禁用命令 | BOT群管理员 |
| `/silence` | 沉默模式 | BOT群管理员 |
| `/admin` | 查看管理员 | 所有人 |

详见 [权限命令](permission.md)。

---

## 娱乐命令

| 命令 | 作用 | 权限 |
|------|------|------|
| `/roll [范围]` | 随机数/随机选择 | 所有人 |
| `/签到` | 每日签到 | 所有人 |
| `/查询积分` | 查看积分 | 所有人 |
| `/倒放 [图片]` | gif 倒放 | 所有人 |
| `/lsp` | 彩蛋 | 所有人 |
| `/色图` | 随机图片（默认禁用） | 所有人 |
| `/help` | 帮助信息 | 所有人 |

详见 [娱乐命令](fun.md)。

---

## 管理员命令（仅私聊）

| 命令 | 作用 |
|------|------|
| `/whosyourdaddy` | 设置管理员（仅无管理员时） |
| `/sysinfo` | 查看 BOT 信息 |
| `/ping` | ping BOT |
| `/log <条数> [-k 关键字]` | 查日志 |
| `/quit <群号> [-f]` | 退出群/清数据 |
| `/mode [公开\|私人\|审核]` | 设置运行模式 |
| `/好友申请 [ID] [--reject] [--all]` | 管理好友申请 |
| `/群邀请 [ID] [--reject] [--all]` | 管理群邀请 |
| `/silence [-d] [-g <群号>]` | 沉默模式 |
| `/检测异常订阅` | 检查异常订阅 |
| `/清除订阅 [选项]` | 清除订阅 |
| `/disable --global <命令>` | 全局禁用命令 |
| `/enable --global <命令>` | 全局启用命令 |
| `/退订更新` | 关闭更新通知 |

详见 [管理员命令](admin.md)。

---

## 私聊操作指定群

私聊使用群命令时，加 `-g <群号>` 指定操作的群：

```bash
/watch -g 123456 97505            # 在群 123456 订阅
/list -g 123456                   # 查看群 123456 的订阅
/config -g 123456 at_all 97505 on # 在群 123456 配置
/silence -g 123456                # 对群 123456 开沉默
```

---

## 命令帮助

几乎所有命令都支持 `-h` / `--help` 查看自身帮助：

```
/watch -h
/config -h
```

---

## 下一步

- [命令格式](format.md) -- 命令语法详解
- 各分类详细页面
- [常见问题](../faq.md)
