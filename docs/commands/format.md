# 命令格式

DDBOT 的命令格式被设计成贴近 shell 命令，方便理解和使用。

---

## 基本格式

```
/主命令 [命令选项 [选项参数 ...] ...] [主命令参数 ...]
```

### 例子

订阅某个 B 站 UP 主的直播：

```
/watch --site bilibili --type live 1472906636
```

分解：

| 部分 | 内容 |
|------|------|
| 主命令 | `watch` |
| 命令选项 1 | `--site` |
| 选项参数 1 | `bilibili` |
| 命令选项 2 | `--type` |
| 选项参数 2 | `live` |
| 主命令参数 | `1472906636` |

---

## 长选项与短选项

命令选项有两种写法，效果相同：

| 长选项 | 短选项 | 作用 |
|--------|--------|------|
| `--site` | `-s` | 指定订阅网站 |
| `--type` | `-t` | 指定订阅类型 |
| `--group` | `-g` | 指定操作的群（私聊用） |
| `--help` | `-h` | 查看帮助 |
| `--delete` | `-d` | 撤销/删除 |

大多数长选项都能用 `-` 加首字母简写。例如：

```
/watch -s bilibili -t live 1472906636
```

---

## 默认值

部分选项有默认值，不填则用默认：

| 选项 | 默认值 | 适用命令 |
|------|--------|---------|
| `-s` / `--site` | `bilibili` | watch/unwatch |
| `-t` / `--type` | `live` | watch/unwatch |

所以下面的命令等价：

```
/watch 1472906636
/watch -s bilibili 1472906636
/watch -s bilibili -t live 1472906636
```

---

## 前缀匹配

`site` 和 `type` 支持**前缀匹配**，不用打全：

| 输入 | 匹配 |
|------|------|
| `bi` | `bilibili` |
| `we` | `weibo` |
| `n` | `news` |
| `l` | `live` |

```
/watch -s bi -t n 97505    # 等价于 /watch -s bilibili -t news 97505
```

---

## 查看帮助

几乎所有命令都自带帮助，用 `-h` 或 `--help`：

```
/watch -h
```

输出示例：

```
Usage: watch <id>

Arguments:
  <id>

Flags:
  -h, --help               Show context-sensitive help.
  -s, --site="bilibili"    bilibili / douyu / youtube / huya
  -t, --type="live"        news / live
```

- `Flags` 列出所有选项
- `--site="bilibili"` 表示默认值是 bilibili
- `Arguments: <id>` 表示需要一个主命令参数

---

## 群聊与私聊

### 群聊

直接在群里发命令即可，BOT 自动识别当前群：

```
/watch 97505
```

### 私聊

私聊发群命令时，需加 `-g <群号>` 指定操作的群：

```
/watch -g 123456 97505
```

**一句话：用法同群聊，只是多加 `-g <群号>`。**

!!! note "私聊命令"
    部分命令是纯私聊命令（如 `/sysinfo`、`/quit`），不需要也不接受 `-g`。

---

## 命令前缀

默认前缀是 `/`，可通过配置修改：

```yaml
bot:
  commandPrefix: "/"     # 全局前缀

customCommandPrefix:     # 单命令前缀（优先级更高）
  签到: ""
  roll: "Q"
```

详见 [推送与模板开关 - 自定义命令前缀](../config/push-template.md)。

---

## 权限说明

每条命令有默认权限要求，命令文档里会标注：

| 权限 | 含义 |
|------|------|
| 所有人 | 任何群员可用 |
| 群管理员 | QQ 群管理员或 BOT 群管理员 |
| BOT 管理员 | 通过 `/whosyourdaddy` 设置的全局管理员 |

权限可通过 `/grant` 命令调整，见 [权限命令](permission.md)。

---

## 下一步

- [命令速查](index.md) -- 所有命令一览
- [订阅命令](subscribe.md) -- `/watch` 详解
