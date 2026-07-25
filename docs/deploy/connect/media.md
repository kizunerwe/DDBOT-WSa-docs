# 媒体与 FFmpeg

DDBOT-WSa 在推送和模板中支持发送图片、视频、语音、文件等多种媒体。本页说明媒体发送机制和 FFmpeg 的安装。

---

## 媒体发送方式

从 v0.3.8 起，所有媒体统一由 **OneBot 实现端直接处理**，DDBOT 支持三种投递方式：

| 方式 | 说明 | 适用 |
|------|------|------|
| **URL** | 传一个可访问的 http/https 链接给实现端下载 | 网络图片、远端文件，超时 120s |
| **Base64** | 把文件内容 base64 编码后传给实现端 | 本地文件、需精确控制的媒体 |
| **本地路径** | 传一个本地路径，由实现端读取 | DDBOT 与实现端同机时最方便 |

对于本地文件，DDBOT 默认按路径发送；也可配合模板函数 `openFile` 读取后自动转 Base64 发送。

### 各媒体类型支持

| 类型 | URL | Base64 | 本地路径 | 备注 |
|------|:---:|:------:|:--------:|------|
| 图片 | ✅ | ✅ | ✅ | 支持 jpg/png/gif，文件夹随机 |
| 视频 | ✅ | ✅ | ✅ | 本地 `.mp4` 可自动转 B64；文件夹随机 |
| 语音 | ✅ | ✅ | ✅ | 本地 `.mp3`/`.wav`/`.ogg` 可自动转 B64 |
| 文件 | ✅ | ✅ | ✅ | 任意文件，文件夹随机 |

!!! warning "文件大小未做检查"
    目前未对文件大小进行限制，发送大文件可能导致发送失败，请自行测试。

---

## FFmpeg 的作用

FFmpeg **仅在以下场景需要**：

1. **推特订阅推送 m3u8 视频**（WSa 新增功能）-- 需要下载并合并分片
2. 模板中涉及视频/语音格式转换的场景

不使用这些功能则无需安装 FFmpeg。

---

## 安装 FFmpeg

=== "Windows"

    1. 到 [ffmpeg.org/download.html](https://ffmpeg.org/download.html) 下载 Windows 预编译版（gyan.dev 或 BtbN 构建）
    2. 解压，把 `bin\ffmpeg.exe` 复制到 **DDBOT 程序所在目录**
    3. 或把 `bin` 目录加入系统 `PATH`

    验证（在 DDBOT 目录打开 PowerShell）：

    ```powershell
    .\ffmpeg.exe -version
    ```

=== "Linux"

    ```bash
    # Debian/Ubuntu
    sudo apt update && sudo apt install -y ffmpeg

    # CentOS/RHEL（需 EPEL 或 RPM Fusion）
    sudo yum install -y ffmpeg
    ```

    验证：

    ```bash
    ffmpeg -version
    ```

=== "macOS"

    ```bash
    brew install ffmpeg
    ```

=== "Docker"

    若 DDBOT 跑在容器里，在 Dockerfile 里加：

    ```dockerfile
    RUN apt-get update && apt-get install -y ffmpeg && rm -rf /var/lib/apt/lists/*
    ```

---

## 验证 FFmpeg 可被 DDBOT 调用

DDBOT 启动时不会主动检查 FFmpeg，只有在需要转码/下载 m3u8 时才会调用。确认方式：

- 把 `ffmpeg`（或 `ffmpeg.exe`）放在 DDBOT 程序目录，或
- 确保 `ffmpeg` 在系统 `PATH` 中可被直接执行

如果推特视频推送失败且日志提示找不到 ffmpeg，按上面方式安装即可。

---

## 模板中的媒体函数

在模板里发送媒体用这些函数（详见 [模板函数](../../template/funcs.md)）：

| 函数 | 作用 |
|------|------|
| `{{ pic "url或路径" }}` | 发送图片，支持 http/https/本地路径/base64，路径为文件夹时随机选一张 |
| `{{ video "路径" }}` | 发送视频 |
| `{{ record "路径" }}` | 发送语音 |
| `{{ file "路径" }}` | 发送任意文件 |
| `{{ remoteDownloadFile "url" }}` | 让实现端下载远端文件，返回服务器绝对路径 |
| `{{ openFile "路径" }}` | 读取本地文件为 `[]byte`，配合媒体函数转 Base64 发送 |

---

## 图片合并模式

B 站动态推送支持图片合并，减少刷屏，在 `application.yaml` 配置：

```yaml
bilibili:
  imageMergeMode: "auto"   # auto / only9 / off
```

| 模式 | 行为 |
|------|------|
| `auto` | 默认，存在较刷屏的图片时合并 |
| `only9` | 仅当恰好 9 张图片时合并 |
| `off` | 不合并 |

---

## 下一步

- [模板函数](../../template/funcs.md) -- 媒体相关函数的完整用法
- [配置参考](../../config/index.md) -- `bilibili.imageMergeMode` 等配置
