# 推特配置

DDBOT-WSa 新增了对推特（Twitter/X）订阅的支持（实验性）。`master` / `next` 仅支持 nitter 镜像（mirror）模式；标注 `[next-dev]` 的字段只在 `next-dev` 存在。本页讲 `twitter` 段配置。

!!! warning "实验性功能"
    推特订阅处于实验阶段，依赖第三方 nitter 镜像，稳定性受镜像可用性影响。

---

## 配置示例

```yaml
twitter:
  mode: mirror               # [next-dev] mirror / api；master / next 不读取该键，固定 mirror
  baseUrl:
    - "https://nitter.net/"
    - "https://nitter.privacyredirect.com/"
    - "https://nitter.tiekoetter.com/"
  interval: 30s
  userAgent:
  unsub: false               # [next] 起生成；仅 [next-dev] 读取
  retweetFullText: false     # [next-dev] api 模式：转发推文显示原推文完整内容
  translate:                 # [next-dev] 推文自动翻译
    enabled: false
    minInterval: 5s
    targetLang: zh
    batchWait: 20s           # 留空则按 minInterval × 2 + 10s 计算
  queryIdRefreshInterval: 168h  # [next-dev] api 模式：queryId 缓存刷新间隔
  apiFetchMode: per_user     # [next-dev] api 查询方式：home_timeline / per_user
```

---

## 字段详解

### `mode` - 运行模式（`[next-dev]`）

```yaml
twitter:
  mode: mirror    # mirror / api
```

`[next]` 起生成该键，但 `master` / `next` 不读取它，始终按 mirror（nitter 镜像）模式运行。`[next-dev]` 才真正生效：显式 `mirror` 用镜像；其它值（包括留空、非法值）都回退 `api`（`resolveTwitterMode`）。

api 模式需要 `twitter.auth_token` / `ct0` / `bearerToken` 等账号 cookie，见 [推特订阅源](../sources/twitter.md)。

### `baseUrl` - nitter 镜像列表

```yaml
twitter:
  baseUrl:
    - "https://nitter.net/"
    - "https://nitter.privacyredirect.com/"
```

推特内容通过 [nitter](https://github.com/nickvdp/nitter)（推特前端镜像）抓取。可配置**多个镜像**，每次请求从列表中**随机选取**一个，提高可用性。

| 说明 | 备注 |
|------|------|
| 生成配置首项 | `nitter.net` |
| 未配置时的代码兜底 | `master` / `next`：`lightbrd.com` 与 `nitter.net`；`[next-dev]`：`nitter.tiekoetter.com` 与 `nitter.catsarch.com`（均随机取一） |
| 末尾 `/` | URL 末尾建议带 `/` |

!!! tip "镜像不可用时"
    nitter 镜像经常变动，如果某个镜像挂了，换一个或加多个备用。

### `interval` - 检测间隔

```yaml
twitter:
  interval: 30s
```

推特动态的查询间隔。`master` 生成 `300s`、`next` / `next-dev` 生成 `30s`。

未配置时：`master` / `next` 代码默认 `30s` 且不告警；`[next-dev]` 代码默认 `120s`，低于 120s 会告警。**过快可能导致 IP 被暂时封禁**，可适当调大。

### `userAgent` - 浏览器 UA

```yaml
twitter:
  userAgent: "Mozilla/5.0 ..."
```

访问 nitter 镜像时使用的 User-Agent。

- 留空则使用默认 UA
- 部分镜像会校验 UA，如果访问失败可填入你浏览器真实的 UA

??? question "如何查看自己的 UA"
    浏览器访问 [https://httpbin.org/user-agent](https://httpbin.org/user-agent) 即可看到。

### `unsub` - 取消订阅时自动取关（`[next-dev]`）

```yaml
twitter:
  unsub: false
```

`[next]` 起生成该键，但只有 `[next-dev]` 真正读取并执行自动取关（`lsp/twitter/concern.go`）；在 `next` 上设置无效。`master` 无此键。`[next-dev]` 下取关还要求当前是 api 模式且 `apiFetchMode` 不是 `per_user`（即 `home_timeline`）；生成配置默认 `apiFetchMode: per_user`，因此在默认配置下设置 `unsub: true` 不会取关。

### `cfclearance` - 已失效

早期版本支持通过 `twitter.cfclearance` 注入 Cloudflare 的 `cf_clearance` cookie。**当前代码已无此配置键，填写不起任何作用**：代码只在抓取时识别 Cloudflare 拦截并报错（错误信息 `cf_clearance has expired!`），无法通过配置注入。

如果镜像被 Cloudflare 拦截，请换用其它镜像。

### `retweetFullText` - 转发推文显示原文全文（`[next-dev]`）

api 模式下，X API 返回的转发推文正文只含 `RT @用户: ` + 原推文前 140 字的截断摘要。开启后改用原推文的完整文本。

```yaml
twitter:
  retweetFullText: false
```

| 值 | 说明 |
|----|------|
| `false`（默认） | 显示截断摘要 |
| `true` | 显示原推文完整内容 |

!!! note "仅 api 模式"
    mirror 模式（nitter）抓到的本就是原文全文，该字段不生效。api 模式说明见 [推特订阅源](../sources/twitter.md)。

### `translate` - 推文自动翻译（`[next-dev]`）

```yaml
twitter:
  translate:
    enabled: false
    minInterval: 5s
    targetLang: zh
    batchWait: 20s
```

启用后，对非中文推文（语言标记不是中文，且中文字符占比不高于 30%）在抓取阶段异步预翻译，推送时在正文后追加 `[翻译] ...` 一行。优先调用 X 官方翻译接口，失败时回退 Google 翻译。

| 字段 | 默认 | 说明 |
|------|------|------|
| `enabled` | `false` | 是否启用自动翻译 |
| `minInterval` | `5s` | 两次翻译请求之间的最小间隔，过小可能触发风控 |
| `targetLang` | `zh` | 翻译目标语言 |
| `batchWait` | `minInterval × 2 + 10s` | 推送渲染时等待预翻译结果的兜底时长，超时则本次不显示译文 |

!!! note "说明"
    - X 官方翻译接口需要 api 模式（`mode: api`）与账号 cookie；mirror 模式直接使用 Google 翻译。
    - 翻译在抓取阶段后台进行，不阻塞推送；`batchWait` 只决定推送时最多等待多久。
    - 译文由模板变量 `TranslatedText` 输出，默认模板已包含；自定义模板需自行添加（见 [推送模板](../template/notify-tmpl.md)）。

### `queryIdRefreshInterval` - queryId 缓存刷新间隔（`[next-dev]`）

```yaml
twitter:
  queryIdRefreshInterval: 168h
```

api 模式下，Twitter 搜索接口的 `queryId` 会从 `sw.js` 自动提取并缓存，缓存超过该间隔后重新提取。默认 `168h`（7 天），仅 api 模式有意义。

### `apiFetchMode` - API 查询方式（`[next-dev]`）

```yaml
twitter:
  apiFetchMode: per_user    # home_timeline / per_user
```

api 模式下选择查询方式，无法识别的值会告警并回退 `home_timeline`。另有 `twitter.userTweetsQueryId` 可覆盖 per_user 模式的 queryId（`[next-dev]`）。

---

## 订阅推特

配置好后，用 `/watch` 命令订阅：

```text
/watch -s twitter <用户名>
```

例如订阅 `elonmusk`：

```text
/watch -s twitter elonmusk
```

推特只支持 `news` 类型（动态），无需指定 `-t`。

---

## m3u8 视频推送

推特动态中的视频（m3u8 格式）可推送到群里，但需要 FFmpeg 来下载合并分片。

- `master`：直接调用 PATH 中的 `ffmpeg`（`lsp/twitter/notify.go`），必须自行安装到 PATH。
- `[next]` 起：源码提供了内置 ffmpeg 的 cgo 构建（`ffmpeg/ffmpeg_cgo.go`）和调用外部命令的实现（`ffmpeg/ffmpeg_cli.go`）；正式发布工作流（`release.yaml`）未启用 cgo（未设置 `CGO_ENABLED=1`），产出的是 CLI 版本、需要 PATH 中有 `ffmpeg`；`next-dev` 的 CI 预览发布（`release-preview.yml`）会同时打包 `-cli` 与 `-cgo` 两种产物。

见 [媒体与 FFmpeg](../deploy/connect/media.md)。

---

## 常见问题

### 抓不到推文

- 检查镜像是否可用（浏览器直接访问试试）
- 换一个镜像或多配几个
- 检查 `userAgent` 是否被镜像拒绝
- 启用 Cloudflare 的镜像当前无法通过配置绕过，请换用其它镜像

### 视频推不出来

- 确认已安装 FFmpeg 且在 PATH 或程序目录
- 查看日志是否有 ffmpeg 相关错误

---

## 下一步

- [推特订阅源说明](../sources/twitter.md) -- 订阅机制与模板变量
- [媒体与 FFmpeg](../deploy/connect/media.md) -- m3u8 视频推送
- [推送模板](../template/notify-tmpl.md) -- 自定义推特推送格式
