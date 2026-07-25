# 推送模板

自定义各订阅源的推送格式。推送模板的变量因订阅源和类型而异。

---

## 直播推送（通用）

以下订阅源的直播推送模板结构相同：

- `notify.group.bilibili.live.tmpl`
- `notify.group.acfun.live.tmpl`
- `notify.group.douyu.live.tmpl`
- `notify.group.huya.live.tmpl`
- `notify.group.douyin.live.tmpl`

通用变量：

| 变量 | 类型 | 含义 |
|------|------|------|
| `living` | bool | 是否正在直播 |
| `name` | string | 主播昵称 |
| `title` | string | 直播标题 |
| `url` | string | 直播间链接 |
| `cover` | string | 封面或头像 |

### B 站直播额外变量（WSa）

| 变量 | 类型 | 含义 |
|------|------|------|
| `area_name` | string | 直播分区 |
| `parent_area_name` | string | 父分区 |
| `live_time` | - | 开播时间/直播时长 |

### 默认模板（以 B 站为例）

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

### 其它订阅源默认模板

斗鱼：

```text
{{ if .living -}}
斗鱼-{{ .name }}正在直播【{{ .title }}】
{{ .url -}}
{{ pic .cover "[封面]" }}
{{- else -}}
斗鱼-{{ .name }}直播结束了
{{ pic .cover "[封面]" }}
{{- end -}}
```

虎牙、ACFun、抖音格式类似，把「斗鱼」换成对应平台名。

---

## B 站动态推送

模板名：`notify.group.bilibili.news.tmpl`（WSa 支持自定义）

B 站动态模板较复杂，通过 `.dynamic` 变量传递结构化数据，按 `.dynamic.Type` 分支渲染。

### 主要变量

| 变量 | 含义 |
|------|------|
| `.dynamic.Type` | 动态类型数字 |
| `.dynamic.User.Name` | 发布者昵称 |
| `.dynamic.OriginUser.Name` | 原动态作者（转发时） |
| `.dynamic.Date` | 发布时间 |
| `.dynamic.Title` | 标题 |
| `.dynamic.Content` | 文字内容 |
| `.dynamic.WithOrigin` | 是否有原动态 |
| `.dynamic.DynamicUrl` | 动态链接 |
| `.msg` | 上一条相关消息（用于回复） |
| `.parsePost` | 是否解析专栏正文 |

### 动态类型

| Type | 类型 | 主要字段 |
|------|------|---------|
| 2 | 转发动态 | `.dynamic.Image`/`.dynamic.Text` |
| 4 | 文字动态 | `.dynamic.Text.Content` |
| 8 | 视频投稿 | `.dynamic.Video`（Title/Desc/CoverUrl/Action/Dynamic） |
| 64 | 专栏 | `.dynamic.Post`（Title/Summary/ImageUrls） |
| 256 | 音频 | `.dynamic.Music`（Title/Intro/Author/CoverUrl） |
| 2048 | Sketch | `.dynamic.Sketch` |
| 4200 / 4308 | 直播分享 | `.dynamic.Live`（Title/CoverUrl） |
| 4300 | 收藏夹 | `.dynamic.MyList`（Title/CoverUrl） |
| 1024 | 缺失动态 | `.dynamic.Miss.Tips` |
| 1 | 转发 | 简化格式 |
| 4302 | 课程 | `.dynamic.Course` |

### 图片字段

`.dynamic.Image` 含：

- `Bytes` - 图片字节（可直接 `pic`）
- `ImageUrls` - 图片 URL 列表
- `Description` - 图片描述

```text
{{ if .dynamic.Image.Bytes -}}
    {{ pic .dynamic.Image.Bytes -}}
{{ else if .dynamic.Image.ImageUrls -}}
    {{ range $v := .dynamic.Image.ImageUrls -}}
        {{ pic $v -}}
    {{ end -}}
{{ end -}}
```

### 附加信息 `.dynamic.Addons`

动态可能附带商品、预约、投票、附加视频等，通过 `.dynamic.Addons` 数组按 `.Type` 分支：

| Addon Type | 含义 |
|------------|------|
| 1 | 商品 |
| 6 | 预约/抽奖 |
| 2 | 相关推荐 |
| 3 | 投票 |
| 5 | 附加视频 |

### 完整默认模板

默认模板很长，按所有动态类型分支处理。建议直接参考源码 `lsp/template/default/notify.group.bilibili.news.tmpl`，在其基础上修改。

### 简化示例

只推送文字和链接：

```text
{{ .dynamic.User.Name }}发布了新动态：
{{ .dynamic.Date }}
{{ .dynamic.Content }}
{{ .dynamic.DynamicUrl }}
```

---

## 推特推送

推特推送使用专门的消息组装逻辑（在代码中），支持：

- 推文文字
- 图片
- m3u8 视频（需 ffmpeg）
- 名称显示策略（`nameStrategy`）

可通过覆盖相关逻辑自定义，详见 [推特订阅源](../sources/twitter.md)。

---

## 自定义示例

### B 站直播简洁版

```text
{{ if .living -}}
🔴 {{ .name }} 正在直播
{{ .title }}
{{ .url -}}
{{- else -}}
{{ .name }} 下播了
{{- end -}}
```

### 斗鱼带分区

```text
{{ if .living -}}
【斗鱼】{{ .name }}开播啦！
{{ .title }}
{{ .url -}}
{{ pic .cover }}
{{- else -}}
【斗鱼】{{ .name }}下播了
{{- end -}}
```

---

## 下一步

- [命令模板](command-tmpl.md) -- 命令回复模板
- [事件模板](trigger-tmpl.md) -- 事件触发模板
- [模板函数](funcs.md) -- 所有可用函数
