# 代理与图片池

本页讲 `proxy`、`localProxyPool`、`pyProxyPool`、`imagePool`、`localPool`、`loliconPool` 这些配置。

---

## 代理池

代理池用于访问需要翻墙的订阅源（如 YouTube、pixiv）。DDBOT 把代理分为两类：

- **oversea** - 可翻墙代理，用于访问境外网站
- **mainland** - 国内代理，用于直连国内网站

```yaml
proxy:
  type: "off"   # localProxyPool / pyProxyPool / off
```

| 值 | 说明 |
|----|------|
| `off`（默认） | 不使用代理 |
| `localProxyPool` | 使用固定代理（自己填地址） |
| `pyProxyPool` | 使用 [py 代理池](https://github.com/jhao104/proxy_pool) 动态获取代理 |

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
  host: http://127.0.0.1:5010
```

需要你自行部署 proxy_pool 服务，DDBOT 通过其 API 获取代理。

---

## 图片池

图片池用于 `/色图` 命令（默认禁用），从图库获取随机图片。

```yaml
imagePool:
  type: "off"   # localPool / loliconPool / off
```

| 值 | 说明 |
|----|------|
| `off`（默认） | 不启用图片池 |
| `localPool` | 使用本地图片目录 |
| `loliconPool` | 使用 [api.lolicon.app](https://api.lolicon.app/) |

### `localPool` - 本地图库

```yaml
localPool:
  imageDir: "/path/to/images"
```

指定本地图片目录，`/色图` 命令会从中随机选一张发送。

### `loliconPool` - lolicon 图库

```yaml
loliconPool:
  apikey:          # 已不需要，留空
  cacheMin: 10
  cacheMax: 50
  proxy:
```

| 字段 | 说明 |
|------|------|
| `apikey` | 由于图库更新，此字段不再需要，留空即可 |
| `cacheMin` | 最小缓存数量 |
| `cacheMax` | 最大缓存数量 |
| `proxy` | 访问 lolicon API 用的代理（可选） |

---

## 模板中的代理控制

在模板的 HTTP 请求函数里，可通过特殊参数控制单次请求的代理：

```
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
| 推特 | 视镜像 | nitter 镜像若在国内可访问则无需 |
| TwitCasting | 视情况 | 部分接口需翻墙 |

!!! tip "海外服务器"
    如果你的 DDBOT 跑在海外服务器，所有订阅源都能直连，无需配代理。

---

## 下一步

- [其它订阅源配置](other-sources.md) -- 各订阅源是否需要代理
- [模板函数](../template/funcs.md) -- HTTP 请求的代理参数
- [命令手册 - 娱乐命令](../commands/fun.md) -- `/色图` 命令
