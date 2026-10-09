# 微博

微博动态订阅。

!!! tip "各分支的微博模式不同"
    `next` 分支支持 `guest` / `login` 两种运行模式；`api` 模式、SUB 过期检测自动恢复与告警通知系统为 `next-dev` 新增。旧版 `master` 只有游客 cookie 自动获取模式，详见 [版本与分支](../deploy/branches.md)。

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

## cookie 机制

`master`（以及 `next` / `next-dev` 的 `guest` 模式）下，DDBOT 启动时自动获取微博游客 cookie（`FreshCookie`，失败重试 3 次）：

- 失败会重试 3 次
- 之后每小时自动刷新一次
- 无需手动干预

### 模式配置（各分支不同）

`guest` / `login` 为 `[next]` 已有的两种模式（`master` 只有游客 cookie 自动获取）；`api` 模式、`login` 的 SUB 自动刷新（`autorefresh`）以及配置块中标注 `[next-dev]` 的键均为 `[next-dev]` 新增。

```yaml
weibo:
  mode: guest             # guest / login（next）/ api（next-dev）
  sub:                    # login 模式可填 sub cookie
  qrlogin: true           # login 模式可扫码
  autorefresh: false      # [next-dev] login/api 模式自动刷新 SUB
  apiModeBaseURL: ""      # [next-dev] api 模式的外部 API 地址
  cookieRefreshAPI: ""    # [next-dev] autorefresh 时用于获取 SUB 的外部 API
  snapcastURL: ""         # [next-dev] guest 模式生成 rid 的 SnapCast 服务
  interval: 60s           # 站点级轮询间隔，仅 [next] / [next-dev] 读取
  disableCookieAlert: false  # [next-dev] 设为 true 关闭 Cookie/SUB 失效告警
```

| 模式 | 说明 |
|------|------|
| `guest` | 访客模式，自动生成临时 Cookie（与旧版 master 行为接近） |
| `login` | `[next]` 登录模式，配置 `sub` 或扫码，支持自动刷新 |
| `api` | `[next-dev]` API 模式，从外部 API 获取数据，无需 Cookie（推荐） |

!!! note "interval 的生效路径"
    - `weibo.interval` 只在 `[next]` / `[next-dev]` 生效（`UseEmitQueueWithSiteInterval("weibo")`）：优先 `weibo.interval`，未配置回退 `concern.emitInterval`（自带示例 5s），两者都缺失时 EmitQueue 用内置默认 5s。
    - `master` 走全局 `concern.emitInterval`，不读取 `weibo.interval`。
    - `cfg.GetWeiboInterval()` 里的 30s 默认值没有任何调用者，不生效。
    - `[next-dev]` 的 `guest` 模式若 `weibo.interval` 低于 60s 会打印告警（日志称「已自动调整为 60s」，但代码并不改动实际间隔）。
    - Cookie 自动刷新固定每小时一次，与 `interval` 无关。

!!! note "模式未配置时的行为"
    `[next]` 未配置 `weibo.mode` 时固定使用 `guest`；`[next-dev]` 未配置时若已填 `weibo.sub` 则自动使用 `login`，否则 `guest`。

!!! note "`alertGroupId` / `alertQQList` 已失效（`[next-dev]`）"
    `[next-dev]` 里 `weibo.alertGroupId` / `weibo.alertQQList` 的 getter 仍在（`alertGroupId` 的注释已标 Deprecated），但都没有任何调用者，**填写不起作用**。告警目标改为用 `/config weibo_alert` 控制（群内 `true`/`false`/`show`）；`weibo.disableCookieAlert: true` 可整体关闭告警。私聊的 `set -g` / `set -q` 也只写入数据库、不改变告警去向，详见 [配置命令](../commands/config-cmd.md)。

---

## 推送说明

微博推送会：

- 发送动态文字内容
- 附带原微博（转发动态）
- 发送动态里的图片（原图质量）

转发动态的推送样式与 B 站统一（v0.0.9 起，见[更新日志](../changelog.md)）。

---

## 常见问题

- 推送异常时先检查网络与微博 cookie 是否有效
- 尝试更新到最新版 DDBOT

### 用户信息丢失

DDBOT-WSa 对微博用户信息丢失有补救机制（v0.4.0 之后的 WSa 版本加入）：`freshNews` 发现缓存的用户信息 UID 为 0 时会重新拉取，仍为 0 则用默认昵称「weibo用户」继续推送（`lsp/weibo/concern.go`）。如仍遇到可反馈 [Issues](https://github.com/cnxysoft/DDBOT-WSa/issues)。

### 图片不发送

- 检查网络
- 更新到最新版（早期 WSa 版本存在微博推送不发送图片的问题，已在 v0.0.9 修复，见[更新日志](../changelog.md)）

---

## 下一步

- [订阅源总览](index.md)
- [推送模板](../template/notify-tmpl.md)
- [版本与分支](../deploy/branches.md) -- next / next-dev 分支的微博模式差异
