# 小黑盒

小黑盒（xhh）动态订阅。

!!! note "该订阅源仅 `next-dev` 提供"
    小黑盒订阅源只在 `next-dev` 分支存在；`master` / `next` 不含该订阅源。分支差异见 [版本与分支](../deploy/branches.md)。本页内容均属 `[next-dev]`。

!!! info "配置键名"
    站点名是 `xhh`，但配置键统一在 `heybox.*` 下。

---

## 订阅

```
/watch -s xhh <用户ID>
```

```bash
# 订阅动态
/watch -s xhh <用户ID>

# 取消订阅
/unwatch -s xhh <用户ID>
```

只支持 `news` 类型（动态）。

---

## 配置

```yaml
heybox:
  x_xhh_tokenid:      # 可选，未配置时自动生成并持久化
  interval: 30s       # 轮询间隔，示例值
  onlyOnlineNotify: false
```

| 字段 | 说明 |
|------|------|
| `x_xhh_tokenid` | 小黑盒 token，可选；留空时依次尝试已持久化的 smidv2，仍没有则自动生成并写入数据库 |
| `interval` | 站点级轮询间隔：优先 `heybox.interval`，未配置回退 `concern.emitInterval`，两者都缺失时 EmitQueue 用内置默认 5s。`cfg.GetHeyboxInterval()` 里的 30s 默认值没有任何调用者，不生效 |
| `onlyOnlineNotify` | 是否只推送 Bot 上线后的动态 |

---

## 推送模板

模板名：`notify.group.xhh.news.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `dynamic.User.Name` | string | 作者昵称 |
| `dynamic.Date` | string | 发布时间 |
| `dynamic.Title` | string | 标题 |
| `dynamic.Description` | string | 正文 |
| `dynamic.HasVideo` | bool | 是否含视频 |
| `dynamic.VideoThumb` | string | 视频封面 |
| `dynamic.VideoURL` | string | 视频链接 |
| `dynamic.Images` | []string | 图片列表 |
| `dynamic.ShareURL` | string | 动态链接 |

---

## 下一步

- [订阅源总览](index.md)
- [推送模板](../template/notify-tmpl.md)
- [版本与分支](../deploy/branches.md)
