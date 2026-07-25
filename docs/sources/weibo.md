# 微博

微博动态订阅。

!!! tip "next / next-dev 分支大幅增强"
    `next` 分支重构了微博模块，新增三种运行模式（`guest`/`login`/`api`）、SUB 过期检测与自动 Cookie 恢复、告警通知系统。`next-dev` 进一步新增移动端 API 支持。详见 [版本与分支](../deploy/branches.md#next-分支的新增内容)。

    master 分支仍为简单的游客 cookie 自动获取模式，本页主要描述 master 行为。

---

## 订阅

```
/watch -s weibo <用户UID>
```

用户 UID 是微博主页 URL 里的数字，例如 `https://weibo.com/u/5462373877` 的 UID 是 `5462373877`。

```bash
# 订阅微博动态
/watch -s weibo 5462373877

# 取消订阅
/unwatch -s weibo 5462373877
```

只支持 `news` 类型（动态）。

---

## cookie 机制（master）

DDBOT 启动时自动获取微博游客 cookie：

- 失败会重试 3 次
- 之后每小时自动刷新一次
- 无需手动干预

### next 分支的三种模式

```yaml
weibo:
  mode: guest             # guest / login / api
  sub:                    # login 模式可填 sub cookie
  qrlogin: true           # login 模式可扫码
  autorefresh: false      # login/api 模式自动刷新 SUB
  apiModeBaseURL: ""      # api 模式的外部 API 地址
  snapcastURL: ""         # guest 模式生成 rid 的 SnapCast 服务
  disableCookieAlert: false
  alertGroupId: 0
```

| 模式 | 说明 |
|------|------|
| `guest` | 访客模式，自动生成临时 Cookie（与 master 行为接近） |
| `login` | 登录模式，配置 `sub` 或扫码，支持自动刷新 |
| `api` | API 模式，从外部 API 获取数据，无需 Cookie（推荐） |

---

## 推送说明

微博推送会：

- 发送动态文字内容
- 附带原微博（转发动态）
- 发送动态里的图片（原图质量）

转发动态的推送样式与 B 站统一。

---

## 常见问题

### 用户信息丢失

DDBOT-WSa 对微博用户信息丢失有补救机制（v0.4.0+），如仍遇到可反馈 [Issues](https://github.com/cnxysoft/DDBOT-WSa/issues)。

### 图片不发送

- 检查网络
- 更新到最新版（早期版本有图片不发送的 bug，已修复）

---

## 下一步

- [订阅源总览](index.md)
- [推送模板](../template/notify-tmpl.md)
- [版本与分支](../deploy/branches.md) -- next 分支的微博三模式
