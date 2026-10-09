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

短选项由各命令单独声明，常见的有 `-s` / `-t` / `-g` / `-h` / `-d` / `-r` / `-a`，**并非所有长选项都有短写**（例如 `/enable --global`、`/清除订阅 --abnormal` 就没有短选项）。例如：

```
/watch -s bilibili -t live 1472906636
```

---

## 默认值

部分选项有默认值，不填则用默认：

| 选项 | 默认值 | 适用命令 |
|------|--------|---------|
| `-s` / `--site` | `bilibili` | watch/unwatch |
| `-t` / `--type` | 该订阅源类型中按字母序最靠前的一个 | watch/unwatch |

`-t` 不填时并不是恒为 `live`：代码取该订阅源类型集合按字母序排序后的第一个。master 中 B 站、YouTube、斗鱼、虎牙、ACFun、抖音、TwitCasting 默认 `live`；微博、推特默认 `news`（推特的类型名是 `Tweets`，字符串值为 `news`——命令里要写 `-t news`，写 `Tweets` 不匹配）。`[next-dev]` 另提供小红书 `xhs`（默认 `live`）、小黑盒 `xhh`（默认 `news`）、Twitch `twitch`（默认 `live`）。

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

帮助文本由 kong 从命令定义生成，结构为 `Usage:` / 说明行 / `Arguments:` / `Flags:`；下面按 `watch` 的代码定义写出各段内容（网站列表由程序在运行时动态拼接，具体排版以实际输出为准）：

```
Usage: watch <id>

当前支持的网站：<以 / 连接的当前已注册订阅源>

Arguments:
  <id>

Flags:
  -h, --help             Show context-sensitive help.
  -s, --site="bilibili"  网站参数
  -t, --type=""          类型参数
```

- `Flags` 列出所有选项
- `--site="bilibili"` 表示默认值是 bilibili
- `--type=""` 表示 `-t` 本身没有固定默认值，不填时按订阅源的第一个类型解析
- `Arguments: <id>` 表示需要一个主命令参数

!!! note "网站列表是动态生成的"
    `-h` 里的网站列表由程序根据当前已注册的订阅源拼接而成，顺序取决于各订阅源包 `init` 的注册顺序，实际以运行时输出为准。`master` / `next` 共 9 个：bilibili、douyu、huya、acfun、youtube、weibo、twitcasting、twitter、douyin；`[next-dev]` 另加 xhs、xhh、twitch，共 12 个。

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
