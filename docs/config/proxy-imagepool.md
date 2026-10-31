# 代理与图片池

本页讲 `proxy`、`localProxyPool`、`pyProxyPool`、`imagePool`、`localPool`、`loliconPool` 这些配置。

---

## 代理池

代理池用于访问需要翻墙的订阅源（如 YouTube、pixiv）。DDBOT 把代理分为两类：

- **oversea** - 可翻墙代理，用于访问境外网站
- **mainland** - 国内代理，用于直连国内网站

```yaml
proxy:
  type: localProxyPool   # localProxyPool / pyProxyPool / off
```

| 值 | 说明 |
|----|------|
| `localProxyPool` | 使用固定代理（自己填地址） |
| `pyProxyPool` | 使用 [py 代理池](https://github.com/jhao104/proxy_pool) 动态获取代理 |
| `off` | 不使用代理 |

!!! warning "未配置时"
    `master` / `next` 未配置 `proxy.type`（空值）或填了其它值时，会记录 `unknown proxy type`，**不启用任何代理**。

`[next-dev]` 起新增 `systemProxy`，并在未配置 `proxy.type` 时默认使用它，见下。

### `systemProxy` - 自动检测系统代理（`[next-dev]`）

`[next-dev]` 起 `proxy.type` 取值增加 `systemProxy`，且**未配置时按 `systemProxy` 处理**。检测方式按平台编译期互斥：

- 非 Windows（`detect_other.go`）：先环境变量（`https_proxy` / `HTTPS_PROXY` / `http_proxy` / `HTTP_PROXY` / `all_proxy` / `ALL_PROXY`，按此优先级），再 Linux GNOME 桌面代理（`gsettings`，仅 manual 手动模式）。
- Windows（`detect_windows.go`）：只读注册表 Internet Settings（`ProxyEnable` + `ProxyServer`），不读环境变量。

检测到的代理只作为 **oversea**（海外）代理使用，国内网站仍直连。

!!! warning "检测不到时"
    代理客户端只监听端口不代表系统代理已开启。检测不到会记录警告，此时可改用 `localProxyPool` 填静态代理，或设为 `off` 显式关闭。终端运行时也可先 `export https_proxy=http://127.0.0.1:端口` 再启动 DDBOT。

`master` / `next` 没有 `systemProxy` 这个取值。

!!! note "需要重启"
    `proxy.type` 在启动时初始化代理池，修改后需重启生效。

### `localProxyPool` - 固定代理

```yaml
localProxyPool:
  oversea:               # 翻墙代理
    - 127.0.0.1:8888
  mainland:              # 国内代理
    - 127.0.0.1:8888
```

填写你自己的代理地址（IP:端口）。会自动识别 URI schema，没 schema 默认 http，也支持 https 和 socks5。

示例（用 Clash/V2Ray 的本地端口）：

```yaml
localProxyPool:
  oversea:
    - http://127.0.0.1:7890
  mainland:
    - http://127.0.0.1:7890
```

### `pyProxyPool` - 动态代理池

对接 [jhao104/proxy_pool](https://github.com/jhao104/proxy_pool) 项目，动态获取代理：

```yaml
pyProxyPool:
  host: http://127.0.0.1:8000
```

需要你自行部署 proxy_pool 服务，DDBOT 通过其 API 获取代理。`host` 没有代码默认值，未配置时为空。

---

## 图片池

图片池用于 `/色图` 命令（默认禁用），从图库获取随机图片。

```yaml
imagePool:
  type: "off"   # localPool / loliconPool / off
```

| 值 | 说明 |
|----|------|
| `localPool` | 使用本地图片目录 |
| `loliconPool` | 使用 [api.lolicon.app](https://api.lolicon.app/) |
| `off` | 显式关闭图片池 |

!!! note "不配置时"
    `imagePool.type` 未配置（空值）或填了其它无效值时不会启用图片池，代码会记录「未知的图片池」；只有显式写 `off` 才是明确的关闭。

`[next-dev]` 生成的配置里写的是 `type: loliconPool`（早期版本曾生成 `lolicon`——代码不认这个取值，会落入「未知的图片池」分支、图片池静默不启用，已修正）。`master` / `next` 生成文件没有 `imagePool` 段，需要时手写。该段由 `[next-dev]` 生成。

### `localPool` - 本地图库

```yaml
localPool:
  imageDir: "/path/to/images"
```

指定本地图片目录，`/色图` 命令会从中随机选一张发送。

### `loliconPool` - lolicon 图库

```yaml
loliconPool:
  apikey:          # 代码注释称已不再需要且不再校验，留空即可
  cacheMin: 20
  cacheMax: 50
  proxy:
```

| 字段 | 说明 |
|------|------|
| `apikey` | 代码注释注明「目前不再需要 apikey 了」且不再校验它（`image_pool/lolicon_pool/pool.go`），留空即可；代码仍会把它作为 `apikey` 参数发送（`api.go`），图库是否采用由外部服务决定 |
| `cacheMin` | 最小缓存数量，代码默认 20 |
| `cacheMax` | 最大缓存数量，代码默认 50 |
| `proxy` | 随请求提交给 lolicon API 的 `proxy` 参数（可选）；本机访问 API 始终走代理池的海外代理 |

---

## 模板中的代理控制

在模板的 HTTP 请求函数里，可通过特殊参数控制单次请求的代理：

```text
DDBOT_REQ_PROXY: prefer_mainland      # 用国内代理
DDBOT_REQ_PROXY: prefer_oversea       # 用翻墙代理
DDBOT_REQ_PROXY: prefer_none          # 不用代理
DDBOT_REQ_PROXY: prefer_any           # 随机选
DDBOT_REQ_PROXY: http://localhost:7890  # 直接用指定代理
```

详见 [模板函数 - HTTP 请求](../template/funcs.md)。

---

## 何时需要代理

| 订阅源 | 是否需代理 | 说明 |
|--------|-----------|------|
| B 站、斗鱼、虎牙、ACFun、微博、抖音 | 否 | 国内网站，直连 |
| YouTube | **是**（国内） | 需翻墙 |
| TwitCasting | 视情况 | 直播封面等资源走海外代理（`PreferOversea`），无代理时封面获取失败 |
| 推特 | 视镜像 | nitter 镜像若在国内可访问则无需 |

---

## 下一步

- [其它订阅源配置](other-sources.md) -- 各订阅源是否需要代理
- [模板函数](../template/funcs.md) -- HTTP 请求的代理参数
- [命令手册 - 娱乐命令](../commands/fun.md) -- `/色图` 命令
