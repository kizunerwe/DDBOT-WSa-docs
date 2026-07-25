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

YouTube 推送使用默认格式（暂无独立模板文件，可通过通用机制自定义）。

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
