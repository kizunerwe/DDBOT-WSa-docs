# TwitCasting

[TwitCasting](https://twitcasting.tv/) 直播订阅，需要注册 App 获取凭证。

---

## 前置准备

1. 访问 [TwitCasting Developer](https://twitcasting.tv/developer.php)
2. 新增一个 App，填入所需资料
3. 获取 `clientId` 和 `clientSecret`

详见 [TwitCasting API 文档](https://apiv2-doc.twitcasting.tv/#registration)。

---

## 配置

```yaml
twitcasting:
  clientId: abc
  clientSecret: xyz
  broadcaster:
    title: false
    created: true
    image: false
  nameStrategy: "name"
```

| 字段 | 说明 |
|------|------|
| `clientId` / `clientSecret` | App 凭证，必填 |
| `broadcaster.title` | 是否推送标题（有风控风险，默认 false） |
| `broadcaster.created` | 是否推送开播时间（默认 true） |
| `broadcaster.image` | 是否推送封面（墙内无法获取，建议有代理才开启） |
| `nameStrategy` | 名称显示：`name`/`userid`/`both` |

!!! tip "风控"
    日文字符多的内容容易触发风控，建议保守配置 `broadcaster`。

---

## 订阅

```
/watch -s twitcasting <用户ID>
```

只支持 `live` 类型。

---

## 致谢

TwitCasting 订阅由 [@eric2788](https://github.com/eric2788) 贡献。

---

## 下一步

- [其它订阅源配置](../config/other-sources.md) -- TwitCasting 配置详解
- [订阅源总览](index.md)
