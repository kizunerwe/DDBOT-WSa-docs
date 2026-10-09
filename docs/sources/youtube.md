# YouTube

YouTube 订阅支持直播（`live`）和视频更新（`news`）。

---

## 订阅

```
/watch -s youtube <channel ID>
```

DDBOT-WSa 兼容 YouTube 新老 channel ID：

- 老 ID：`UCxxxxxxxx`（以 `UC` 开头）
- 新 ID：`@handle`（以 `@` 开头）

```bash
# 订阅直播（老 ID）
/watch -s youtube UCvEX2UICvFAa_T6pqizC20g

# 订阅视频更新
/watch -s youtube -t news UCvEX2UICvFAa_T6pqizC20g

# 订阅直播（新 ID）
/watch -s youtube @SomeChannel
```

---

## 代理要求

- **海外服务器**：直连即可
- **国内服务器**：需配置代理才能访问 YouTube，见 [代理与图片池](../config/proxy-imagepool.md)

```yaml
proxy:
  type: localProxyPool
localProxyPool:
  oversea:
    - http://127.0.0.1:7890
```

---

## 推送模板

`master` 的 YouTube 推送使用内置文案（`lsp/youtube/model.go` 的 `GetMSG`，硬编码「YTB-xxx正在直播 / 发布了直播预约 / 发布了新视频」），`lsp/template/default/` 下没有 youtube 模板文件。

`[next]` 起改为模板渲染，新增以下模板。

直播模板：`notify.group.youtube.live.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `living` | bool | 是否正在直播（false 时表示直播预约） |
| `name` | string | 频道名 |
| `title` | string | 标题 |
| `url` | string | 链接 |
| `cover` | string | 封面 |
| `timestamp` | int64 | 开播/预约时间戳 |

视频（news）模板：`notify.group.youtube.news.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `name` | string | 频道名 |
| `title` | string | 视频标题 |
| `cover` | string | 封面 |
| `url` | string | 视频链接 |

`[next-dev]` 的 data map 额外提供 `is_shorts`（是否为 Shorts）、`publish_timestamp`（发布时间戳）、`duration_seconds`（时长，秒）与 `video_timestamp`；news 模板会使用前三个。`next` 与 `master` 无这些变量（`master` 也没有该模板）。

---

## 常见问题

### 推送失效

- 检查代理是否可用（国内）
- YouTube 接口偶尔变动，更新到最新版 DDBOT

### 新老 ID

- 老 ID 订阅失败时可尝试新 ID（`@handle`）
- 反之亦然

---

## 下一步

- [订阅源总览](index.md)
- [代理与图片池](../config/proxy-imagepool.md)
