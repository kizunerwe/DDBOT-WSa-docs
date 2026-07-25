# B 站

B 站是 DDBOT 最常用的订阅源，支持直播（`live`）和动态（`news`）两种类型。

---

## 订阅类型

| 类型 | 说明 | 订阅命令 |
|------|------|---------|
| `live` | 直播状态（开播/下播） | `/watch <UID>` |
| `news` | 动态更新 | `/watch -t news <UID>` |

UID 是 UP 主空间主页 URL 里的数字，例如 `https://space.bilibili.com/97505` 的 UID 是 `97505`。

---

## 订阅示例

```bash
# 订阅直播
/watch 97505

# 订阅动态
/watch -t news 97505

# 同时订阅直播和动态（两条命令）
/watch 97505
/watch -t news 97505
```

---

## 登录与凭证

B 站订阅**推荐配置账号**（cookie 或扫码），不配置也能用但订阅数不要超过 5 个。

| 方式 | 配置 | 适用 |
|------|------|------|
| 扫码登录 | `qrlogin: true` | 测试用，cookie 失效需重新扫码 |
| cookie | `SESSDATA` + `bili_jct` | 推荐，稳定 |
| 账号密码 | `account` + `password` | 目前不可用 |

详见 [B 站配置](../config/bilibili.md)。

!!! info "BOT 会自动关注"
    订阅一个 UP 主后，配置的 B 站账号会自动关注该 UP 主（除非 `disableSub: true`）。建议用小号。

---

## 直播推送

### 推送模板

模板名：`notify.group.bilibili.live.tmpl`

| 变量 | 类型 | 含义 |
|------|------|------|
| `living` | bool | 是否正在直播 |
| `name` | string | 主播昵称 |
| `title` | string | 直播标题 |
| `url` | string | 直播间链接 |
| `cover` | string | 封面或头像 |
| `area_name` | string | 直播分区（WSa 新增） |
| `parent_area_name` | string | 父分区（WSa 新增） |
| `live_time` | - | 开播时间/直播时长 |

默认模板：

```text
{{ if .living -}}
{{ .name }}正在直播【{{ .title }}】
直播分区：{{ .parent_area_name }} - {{ .area_name }}
开始时间：{{ getTime .live_time "" }}
{{ .url -}}
{{ pic .cover "[封面]" }}
{{- else -}}
{{ .name }}直播结束了
直播时长：{{ getTime .live_time "elapsed" }}
{{ pic .cover "[封面]" }}
{{- end -}}
```

### 直播相关配置

- `/config at_all <UID> on` - 开播时 @全体成员
- `/config title_notify <UID> on` - 标题变更时重新推送
- `/config offline_notify <UID> on` - 下播时也推送

---

## 动态推送

### 推送模板

模板名：`notify.group.bilibili.news.tmpl`（WSa 支持自定义）

动态模板较复杂，通过 `.dynamic` 变量传递结构化数据，按动态类型（`.dynamic.Type`）分支渲染：

| Type | 动态类型 |
|------|---------|
| 2 | 转发动态 |
| 4 | 文字动态 |
| 8 | 视频投稿 |
| 64 | 专栏 |
| 256 | 音频 |
| 2048 | Sketch |
| 4200 / 4308 | 直播分享 |
| 4300 | 收藏夹 |
| 1024 | 缺失动态 |
| 1 | 转发 |
| 4302 | 课程 |

详见 [推送模板 - B 站动态推送](../template/notify-tmpl.md) 和默认模板文件 `notify.group.bilibili.news.tmpl`。

### 动态过滤

B 站动态类型繁多，可用 `/config filter` 过滤：

```bash
# 只推送图片、文字、专栏、投稿
/config filter type 97505 图片 文字 专栏 投稿

# 不推送转发
/config filter not_type 97505 转发
```

支持的类型：`专栏`、`转发`、`投稿`、`文字`、`图片`、`直播分享`，也可填类型数字。

### 专栏解析

`autoParsePosts: true` 时，专栏动态会自动解析正文内容发送，而非只发卡片：

```yaml
bilibili:
  autoParsePosts: false
```

---

## 防刷屏机制

B 站动态有几种防刷屏机制：

- **联合投稿去重**：短时间内多人联合投稿，后续动态简化
- **多次转发省略**：短时间推送同一条动态的多次转发，只有第一次显示全部内容
- **图片合并**：多图动态按 `imageMergeMode` 合并成长图

---

## 常见问题

### 订阅失败

- 检查 B 站账号/cookie 是否配置且有效
- 检查 `minFollowerCap` 粉丝数门槛
- 新号可能提示「账户异常」（B 站侧限制）

### 推送延迟高

- 不配账号时订阅数 ≤ 5
- 配账号后可订阅至 2000
- 不要把 `interval` 调太小

### cookie 过期

- 改用扫码登录（`qrlogin: true`）
- 或重新获取 cookie 填入

详见 [B 站配置](../config/bilibili.md) 和 [FAQ](../faq.md)。

---

## 下一步

- [B 站配置](../config/bilibili.md) -- 完整配置字段
- [配置命令](../commands/config-cmd.md) -- `/config` 用法
- [推送模板](../template/notify-tmpl.md) -- 自定义推送格式
