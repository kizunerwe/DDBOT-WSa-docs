# 快速开始

本页帮你用最短时间跑通 DDBOT-WSa：**下载运行 → 连接 OneBot 实现端 → 完成第一次订阅**。

!!! tip "前置说明"
    DDBOT-WSa **不直接登录任何 IM**，它通过 WebSocket 把消息交给一个 OneBot 11 实现端去收发。所以你需要准备两样东西：

    1. **DDBOT-WSa 程序**（本框架）
    2. **一个 OneBot 实现端**（负责连接 IM 平台，如 QQ）

    如果你还没有实现端，可参考 [连接 OneBot](../deploy/connect/index.md) 选择一个适合自己环境的实现端。

---

## 第 1 步：下载并运行 DDBOT

### 下载

到 [Releases](https://github.com/cnxysoft/DDBOT-WSa/releases) 下载适合你系统的版本，按 `<系统>-<架构>` 匹配：

| 系统 | 架构 | 文件名示例 |
|------|------|-----------|
| Windows 10/11/Server | 64 位 | `DDBOT-WSa-fix_A041-windows-amd64.zip` |
| Windows | 32 位 | `DDBOT-WSa-fix_A041-windows-386.zip` |
| Linux | 64 位 | `DDBOT-WSa-fix_A041-linux-amd64.tar.gz` |
| Linux | ARM64 | `DDBOT-WSa-fix_A041-linux-arm64.tar.gz` |
| macOS | Intel | `DDBOT-WSa-fix_A041-darwin-amd64.tar.gz` |

!!! note "文件名可能不同"
    不同版本/分支的 Release 命名格式略有差异（早期版本是 `DDBOT_windows_amd64.exe`，next-dev 预览版是 `DDBOT-darwin-amd64-cgo.zip`），按系统+架构匹配即可，前缀差异不影响使用。详见 [安装 - 版本命名规则](../deploy/install.md#版本命名规则)。

不知道选哪个？Windows 64 位用户直接选 `windows-amd64`。

### 运行

解压后你会得到一个可执行文件：

=== "Windows"

    双击 `DDBOT.exe` 运行。

    首次运行会在当前目录生成两个文件：

    - `device.json` -- 历史遗留占位文件，DDBOT-WSa 中无实质作用，无需特别保护
    - `application.yaml` -- 配置文件

=== "Linux"

    ```bash
    chmod +x ./DDBOT
    ./DDBOT
    ```

    首次运行会生成 `application.yaml`（以及占位的 `device.json`）。

!!! info "首次运行无报错即可"
    此时不需要修改任何配置，DDBOT 会用默认配置启动，并监听 WebSocket 端口 `15630`。看到日志里出现 `WebSocket server started on ws://.../ws` 就说明启动成功。

---

## 第 2 步：连接 OneBot 实现端

DDBOT 默认以 **正向 WebSocket（ws-server）** 模式监听 `0.0.0.0:15630`，路径为 `/ws`。
你需要让 OneBot 实现端**反向连接**到这个地址。

常见的实现端及其配置要点：

| 实现端 | 配置要点 | 详细教程 |
|--------|---------|---------|
| **LLOneBot / LLBot** | 开启反向 WS，地址填 `ws://127.0.0.1:15630/ws` | [LLOneBot 对接](../deploy/connect/llonebot.md) |
| **NapCat** | 网络配置里添加反向 WS 地址 | [NapCat 对接](../deploy/connect/napcat.md) |
| **Lagrange** | 配置 `Type: ReverseWebSocket` | [Lagrange 对接](../deploy/connect/lagrange.md) |
| **云崽 ws-plugin** | 安装 ws-plugin 后配置连接 | [云崽对接](../deploy/connect/yunzai.md) |

完整列表与选择建议见 [连接 OneBot](../deploy/connect/index.md)。

连接成功后，DDBOT 日志会显示 `有新的ws连接了!!`，实现端那边也会显示已连接。

!!! question "正向 / 反向 WebSocket 有什么区别？"
    - **正向（ws-server）**：DDBOT 开端口等实现端连过来（默认）
    - **反向（ws-reverse）**：DDBOT 主动去连实现端的地址

    两者二选一即可，新手用默认的正向即可。详见 [连接 OneBot](../deploy/connect/index.md)。

---

## 第 3 步：设置管理员并订阅

### 设置管理员

用你自己的 QQ（管理员账号）私聊 **BOT 账号**发送：

```
/whosyourdaddy
```

BOT 回复「成功 - 您已成为bot管理员」即完成。

!!! warning "只有 BOT 没有管理员时此命令才生效"
    后续再发 `/whosyourdaddy` 不会改变管理员。如需更换，见 [首次配置](../deploy/first-config.md)。

### 完成第一次订阅

1. **拉 BOT 进一个测试群**（或邀请它加群，公开模式下会自动同意）
2. 在群里发送命令订阅 B 站直播（以示例 UID `97505` 为例）：

    ```
    /watch 97505
    ```

3. 查看订阅列表：

    ```
    /list
    ```

4. 当该 UP 主开播时，BOT 会在群里推送消息。

### 推荐的进阶配置

订阅 B 站动态（默认只订阅直播）：

```
/watch -t news 97505
```

不推送转发动态（避免刷屏）：

```
/config filter not_type 97505 转发
```

直播推送时 @全体成员（需把 BOT 设为群管理员）：

```
/config at_all 97505 on
```

减少 BOT 在群里的啰嗦输出：

```
/silence
```

---

## 下一步

- :material-book-open-variant: 完整了解 DDBOT 的设计理念与架构 -- [了解 DDBOT](../deploy/intro.md)
- :material-cog: 逐项配置 `application.yaml` -- [配置参考](../config/index.md)
- :material-console: 学习所有命令 -- [命令手册](../commands/index.md)
- :material-server: 对接细节与排障 -- [部署与连接](../deploy/intro.md)

遇到问题先看 [常见问题](../faq.md)，仍未解决再到 [Issues](https://github.com/cnxysoft/DDBOT-WSa/issues) 反馈或加交流群 `980848391`。
