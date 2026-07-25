# 安装

DDBOT-WSa 是一个 Go 编译的单文件二进制程序，无外部依赖（除可选的 FFmpeg）。你可以下载预编译版本，也可以从源码编译。

---

## 方式一：下载预编译版本（推荐）

到 [Releases](https://github.com/cnxysoft/DDBOT-WSa/releases) 下载适合你系统的版本。

### 版本命名规则

Release 资产命名因版本和分支而异，常见格式：

- **master 分支**（如 `fix_A041`）：`DDBOT-WSa-fix_A041-windows-amd64.zip`、`DDBOT-WSa-fix_A041-linux-amd64.tar.gz`
- **较早的 master 版本**（如 `fix_A038` 及之前）：`DDBOT_windows_amd64.exe`、`DDBOT_linux_amd64`（无 `-WSa-` 前缀，无压缩包）
- **next-dev 预览版**：`DDBOT-darwin-amd64-cgo.zip` / `DDBOT-darwin-amd64-cli.zip`（区分 cgo / cli 构建）

下载时按 `<系统>-<架构>` 匹配即可，文件名前缀的差异不影响使用。

| 系统 | 架构 | 适用 |
|------|------|------|
| `windows` | `amd64` | Windows 10/11/Server，64 位（最常见） |
| `windows` | `386` | Windows 32 位 |
| `windows` | `arm64` | Windows on ARM |
| `linux` | `amd64` | Linux 服务器，64 位 |
| `linux` | `arm64` | 树莓派 / ARM 服务器 |
| `linux` | `arm` | 旧 ARM 设备 |
| `darwin` | `amd64` | macOS Intel |
| `darwin` | `arm64` | macOS Apple Silicon |

??? question "不知道自己的架构？"
    - **Windows**：按 `Win + Pause` 看「系统类型」，或 PowerShell 运行 `$env:PROCESSOR_ARCHITECTURE`，`AMD64` 即 64 位
    - **Linux**：终端运行 `uname -m`，`x86_64` 选 `amd64`，`aarch64` 选 `arm64`
    - **macOS**：Apple 芯片（M1/M2/M3）选 `darwin-arm64`，Intel 芯片选 `darwin-amd64`

### 解压运行

=== "Windows"

    1. 解压 zip，得到 `DDBOT.exe`
    2. 双击运行，或在该目录打开 PowerShell 运行 `.\DDBOT.exe`
    3. 首次运行会生成 `application.yaml`（和占位的 `device.json`）

=== "Linux"

    ```bash
    # 解压
    tar -xzf DDBOT-WSa-fix_AXXX-linux-amd64.tar.gz
    # 赋予执行权限
    chmod +x ./DDBOT
    # 运行
    ./DDBOT
    ```

=== "macOS"

    ```bash
    tar -xzf DDBOT-WSa-fix_AXXX-darwin-arm64.tar.gz
    chmod +x ./DDBOT
    ./DDBOT
    ```

    !!! warning "macOS Gatekeeper 拦截"
        首次运行可能提示「无法验证开发者」。在「系统设置 -> 隐私与安全性」里点「仍要打开」，或终端运行 `xattr -d com.apple.quarantine ./DDBOT`。

---

## 方式二：从源码编译

需要 [Go 1.24+](https://go.dev/dl/)。

```bash
git clone https://github.com/cnxysoft/DDBOT-WSa.git
cd DDBOT-WSa
make build
```

如果没有 `make`，直接：

```bash
go build -o DDBOT ./cmd
```

!!! note "为什么需要 Go 1.24"
    `go.mod` 指定了 `go 1.24.5`，部分依赖（如本地 replace 的 `miraigo`）用到了较新特性。

---

## 可选：安装 FFmpeg

**仅在以下场景需要**：

- 推特订阅推送 m3u8 视频（WSa 新增）
- 模板里使用视频/语音转换相关功能

安装后把 `ffmpeg` 放到 DDBOT 程序所在目录，或加入系统 `PATH`。

=== "Windows"

    到 [ffmpeg.org](https://ffmpeg.org/download.html) 下载 Windows 预编译版，解压后把 `bin/ffmpeg.exe` 放到 DDBOT 目录或加入 PATH。

=== "Linux"

    ```bash
    sudo apt install ffmpeg       # Debian/Ubuntu
    sudo yum install ffmpeg       # CentOS/RHEL
    ```

验证：

```bash
ffmpeg -version
```

---

## 运行目录结构

正常运行后，DDBOT 所在目录大致如下：

```
你的目录/
├── DDBOT / DDBOT.exe      # 程序本体
├── application.yaml        # 配置文件（首次生成）
├── device.json             # 历史遗留占位文件（无实质作用）
├── .lsp.db                 # 数据库（订阅/权限/配置，运行后生成）
├── logs/                   # 日志目录（保留 7 天）
└── template/               # 模板目录（启用模板后生成）
```

!!! danger "重要文件警告"
    - **`.lsp.db`** 是所有订阅和权限数据，删除即恢复出厂设置，**务必定期备份**
    - **`device.json`** 是历史遗留的占位文件，DDBOT-WSa 不再依赖它进行 QQ 登录，删除无实质影响

---

## 后台常驻运行

=== "Linux (screen)"

    ```bash
    # 安装 screen
    sudo apt install screen     # Debian/Ubuntu
    sudo yum install screen     # CentOS

    # 创建会话
    screen -S ddbot
    ./DDBOT

    # 离开会话（保持运行）：按 Ctrl+A 然后按 D
    # 重新进入：screen -r ddbot
    # 停止：screen -S ddbot -X quit
    ```

=== "Linux (systemd)"

    创建 `/etc/systemd/system/ddbot.service`：

    ```ini
    [Unit]
    Description=DDBOT-WSa
    After=network.target

    [Service]
    Type=simple
    WorkingDirectory=/path/to/ddbot
    ExecStart=/path/to/ddbot/DDBOT
    Restart=on-failure
    RestartSec=10

    [Install]
    WantedBy=multi-user.target
    ```

    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable --now ddbot
    sudo systemctl status ddbot
    ```

=== "Windows"

    直接双击运行保持窗口打开即可。需要后台运行可用 [nssm](https://nssm.cc/) 注册为服务：

    ```powershell
    nssm install ddbot "D:\path\to\DDBOT.exe"
    nssm start ddbot
    ```

---

## 验证启动成功

看到类似下面的日志即表示启动成功：

```
WebSocket server started on ws://0.0.0.0:15630/ws
DDBOT启动完成
```

接下来去 [首次配置](first-config.md) 设置管理员，然后 [连接 OneBot](connect/index.md)。

---

## CLI 参数

DDBOT 支持以下命令行参数（用 `kong` 解析）：

| 参数 | 作用 |
|------|------|
| `--version` / `-v` | 打印版本信息 |
| `--debug` | 启动 debug 模式，开启 pprof（`localhost:6060`），并限制命令触发范围 |
| `--set-admin <QQ>` | 离线设置管理员（BOT 未运行时） |
| `--sync-bilibili` | 同步 B 站账号关注（迁移账号时用） |
| `--play` | 运行测试函数（开发用） |

示例：

```bash
./DDBOT --version
./DDBOT --set-admin 12345678
./DDBOT --debug
```
