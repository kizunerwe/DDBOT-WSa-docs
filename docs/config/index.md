# 配置总览

DDBOT-WSa 的配置文件是 `application.yaml`，首次运行自动生成最小配置。本页给出完整配置字段速查，各模块的详细说明见左侧子页面。

---

## 配置文件位置

- 路径：DDBOT **当前工作目录**下的 `application.yaml`
- 也可放在 `./config/application.yaml`
- 修改后部分配置支持热重载（如 `customCommandPrefix`、`cronjob`），其余需重启

!!! warning "YAML 格式"
    - 删除 `#` 及后面的注释
    - 冒号后必须有一个英文空格
    - 所有标点用英文半角
    - 格式错误会导致启动闪退

---

## 完整配置示例

```yaml
bot:
  account:                   # bot 的 QQ 号，不填则扫码登录
  password:                  # bot 的 QQ 密码
  commandPrefix: "/"         # 命令前缀，默认 /
  onDisconnected: "exit"     # 掉线处理：exit 退出，留空尝试重连
  onJoinGroup:
    rename: "【bot】"         # 进群自动改名，留空不改
  sendFailureReminder:       # 发送失败提醒
    enable: false
    times: 3
  offlineQueue:              # 离线缓存
    enable: false
    expire: 30m

bilibili:
  SESSDATA:                  # B 站 cookie
  bili_jct:                  # B 站 cookie
  account:                   # B 站账号（目前不可用，用 cookie）
  password:                  # B 站密码（目前不可用）
  qrlogin: true              # 扫码登录 B 站
  interval: 25s              # 检测间隔
  imageMergeMode: "auto"     # 图片合并：auto / only9 / off
  hiddenSub: false           # 悄悄关注
  unsub: false               # 自动取关
  minFollowerCap: 0          # 最低粉丝数门槛
  disableSub: false          # 禁止自动关注
  onlyOnlineNotify: false    # 仅推送在线期间的更新
  autoParsePosts: false      # 自动解析专栏内容

twitter:                     # WSa 新增
  baseUrl:
    - "https://nitter.net/"
  interval: 300s
  userAgent:
  cfclearance:

douyin:                      # WSa 新增（测试）
  interval: 30s
  userAgent:
  acSignature:
  acNonce:

twitcasting:
  clientId:
  clientSecret:
  broadcaster:
    title: false
    created: true
    image: false
  nameStrategy: "name"

concern:
  emitInterval: 5s           # 订阅刷新频率

dispatch:
  largeNotifyLimit: 50       # 巨量推送判定阈值

notify:
  parallel: 1                # 推送并发数

template:
  enable: false              # 启用模板

autoreply:
  group:
    command: []
  private:
    command: []

cronjob:                     # 定时消息
  - cron: "* * * * *"
    templateName: "定时1"
    target:
      private: [123]
      group: []

customCommandPrefix:         # 自定义命令前缀
  签到: ""

imagePool:
  type: "off"                # localPool / loliconPool / off

localPool:
  imageDir:                  # 本地图片目录

loliconPool:
  apikey:                    # 已不需要，留空
  cacheMin: 10
  cacheMax: 50

proxy:
  type: "off"                # localProxyPool / pyProxyPool / off

localProxyPool:
  oversea:                   # 翻墙代理
    - 127.0.0.1:8888
  mainland:                  # 国内代理
    - 127.0.0.1:8888

pyProxyPool:
  host: http://127.0.0.1:5010

websocket:                   # OneBot 连接（WSa 核心）
  mode: ws-server            # ws-server / ws-reverse
  token:
  ws-server: 0.0.0.0:15630
  ws-reverse: ws://localhost:3001

reloadDelay:
  enable: true               # 延迟加载好友/群信息
  time: 3s

message-marker:
  disable: false             # 禁用自动已读

qq-logs:
  enable: false              # 命令行展示 QQ 聊天内容

debug:
  group:
    - 0
  uin:
    - 0

logLevel: info               # trace / debug / info / warn / error
```

---

## 字段分类速查

按功能模块分类，点击进入详细说明：

| 模块 | 关键字段 | 说明 |
|------|---------|------|
| [bot 与连接](bot.md) | `bot.*`、`websocket.*`、`reloadDelay`、`debug`、`message-marker`、`qq-logs` | 账号、命令前缀、掉线策略、OneBot 连接、调试 |
| [B 站](bilibili.md) | `bilibili.*` | cookie/扫码登录、检测间隔、图片合并、关注策略、专栏解析 |
| [推特](twitter.md) | `twitter.*` | nitter 镜像、UA、cf_clearance（WSa 新增） |
| [抖音](douyin.md) | `douyin.*` | cookie、UA、人机验证（WSa 新增，测试） |
| [其它订阅源](other-sources.md) | `twitcasting.*` | TwitCasting clientId/secret |
| [代理与图片池](proxy-imagepool.md) | `proxy.*`、`localProxyPool.*`、`pyProxyPool.*`、`imagePool.*`、`localPool.*`、`loliconPool.*` | 代理池、图库 |
| [推送与模板开关](push-template.md) | `concern.*`、`dispatch.*`、`notify.*`、`template.*`、`autoreply.*`、`cronjob.*`、`customCommandPrefix` | 推送调度、模板、自定义命令、定时消息 |

---

## 最小可用配置

如果只想快速跑起来测试，下面这份就够：

```yaml
bot:
  onJoinGroup:
    rename: "【bot】"

bilibili:
  qrlogin: true
  interval: 25s

concern:
  emitInterval: 5s

template:
  enable: true

websocket:
  mode: ws-server
  ws-server: 0.0.0.0:15630

logLevel: info
```

不配置 B 站账号时，订阅数**不要超过 5 个**，否则推送延迟上升。

---

## 配置热重载

DDBOT 用 viper 的 `WatchConfig` 监听配置文件变化，以下配置**修改后无需重启**：

- `customCommandPrefix`（自定义命令前缀）
- `cronjob`（定时消息）

其余配置项修改后需要重启 DDBOT 才能生效。

---

## 下一步

- 按需打开左侧子页面配置各模块
- [命令手册](../commands/index.md) -- 配好后学习命令
- [常见问题](../faq.md) -- 配置出错先看这里
