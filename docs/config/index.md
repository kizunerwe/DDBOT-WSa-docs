# 配置总览

DDBOT-WSa 的配置文件是 `application.yaml`，首次运行会在**当前工作目录**自动生成一份示例配置（源码里的 `exampleConfig`）。

!!! note "生成文件包含哪些段"
    不同分支生成的文件不一样：

    - `master`：`bot`、`bilibili`、`twitter`、`douyin`、`concern`、`template`、`autoreply`、`customCommandPrefix`、`logLevel`、`websocket`、`reloadDelay`。
    - `[next]` 追加：`acfun`、`weibo`、`youtube`、`twitcasting`、`extDb`、`telegram`。
    - `[next-dev]` 追加：`proxy`、`localProxyPool`、`pyProxyPool`、`imagePool`、`loliconPool`、`localPool`、`douyu`、`huya`、`xhs`、`twitch`、`heybox`、`notify`、`dispatch`、`cronjob`、`debug`、`message-marker`、`qq-logs`、`adapter`、`admin`；同时移除 `reloadDelay`。

    下面这份 YAML 是**字段速查 / 合并示例**，不是生成文件的全集。生成文件里没有的键，需要时手动添加；它们是否被读取见各子页面。

---

## 配置文件位置

- 路径：DDBOT **当前工作目录**下的 `application.yaml`
- 程序启动时只检查当前目录的 `application.yaml`；不存在就先在**当前目录**写一份，然后才依次 `AddConfigPath(".")`、`AddConfigPath("./config")` 读取。当前目录总有一份且排在前面，所以放在 `./config/application.yaml` 的配置不会被读到
- 修改后部分配置支持热重载（如 `customCommandPrefix`、`cronjob`；`[next-dev]` 的 `platform.<site>.enable`），其余需重启

!!! warning "YAML 格式"
    - 删除 `#` 及后面的注释
    - 冒号后必须有一个英文空格
    - 所有标点用英文半角
    - 格式错误会导致启动闪退

---

## 字段速查 / 合并示例

```yaml
bot:
  commandPrefix: "/"         # 命令前缀，默认 /
  onDisconnected: ""         # 写 "exit" 直接退出；留空时 master / next 尝试重连（失败也退出），[next-dev] 仅记录日志
  onJoinGroup:
    rename: "【bot】"         # 进群自动改名，留空不改
  sendFailureReminder:       # 发送失败提醒（master / next 有效；[next-dev] 不会触发）
    enable: false
    times: 3
  offlineQueue:              # 离线缓存
    enable: false
    expire: 30m              # 生成值；留空时 master / next 解析为 0 并丢弃全部缓存，[next-dev] 回退 30m

bilibili:
  SESSDATA:
  bili_jct:
  account:                   # 账号密码登录（未配置 cookie 时使用）
  password:
  qrlogin: true
  interval: 25s              # 生成配置 25s；代码未配置时默认 20s
  imageMergeMode: "auto"
  hiddenSub: false
  unsub: false
  minFollowerCap: 0          # 0 要求粉丝 > 0；-1 不限制；正整数 N 表示粉丝 <= N 拒绝
  disableSub: false
  onlyOnlineNotify: false
  autoParsePosts: false
  secAnalysis: false         # [next] 动态二次解析

acfun:                       # [next]
  account:
  password:
  authKey:                   # [next]
  acPassToken:               # [next]
  unsub: false
  interval: 25s              # 生成配置 25s
  onlyOnlineNotify: false
  minFollowerCap: 0          # [next-dev] 生成该键，但代码无读取点，设置无效

twitter:
  mode: mirror               # [next] 起生成但 master / next 不读取（固定 mirror）；仅 [next-dev] 生效，mirror / api（缺省回退 api）
  baseUrl:
    - "https://nitter.net/"
  interval: 30s              # master 生成 300s；next / next-dev 生成 30s；[next-dev] 未配置时代码默认 120s，显式配置低于 120s 会告警；master / next 未配置默认 30s 且不告警
  userAgent:
  unsub: false               # [next-dev] 取消订阅时自动取关（仅 api 模式且 apiFetchMode 非 per_user）
  retweetFullText: false     # [next-dev]
  translate:                 # [next-dev]
    enabled: false
    minInterval: 5s
    targetLang: zh
    batchWait: 20s
  queryIdRefreshInterval: 168h  # [next-dev]
  # api 模式（mode: api 时生效，[next-dev]）
  auth_token:
  ct0:
  bearerToken:
  queryId:
  screenName:

douyin:                      # WSa 新增（测试）；master 仅 live
  acSignature:
  acNonce:
  userAgent:
  interval: 30s
  sessionId:                 # [next]
  onlyOnlineNotify: false    # [next]

weibo:                       # [next]
  onlyOnlineNotify: true     # 生成配置 true；键缺失时代码默认 false
  mode: guest                # guest / login（[next]）/ api（[next-dev]）；非法值会告警并回退 guest
  interval: 30s
  sub:
  qrlogin: true
  autorefresh: false         # [next-dev]
  apiModeBaseURL: "http://127.0.0.1:5000"  # [next-dev] 生成值
  snapcastURL: ""            # [next-dev]
  disableCookieAlert: true   # [next-dev] true 关闭 Cookie/SUB 失效告警；键缺失时告警开启
  cookieRefreshAPI:          # [next-dev] 读取但不生成该键；autorefresh 依赖它

youtube:                     # [next]
  onlyOnlineNotify: true     # 生成配置 true；键缺失时代码默认 false

douyu:                       # [next-dev]
  interval: 60s              # 生成该键，但代码未读取；轮询间隔走 concern.emitInterval

huya:                        # [next-dev]
  interval: 60s              # 生成该键，但代码未读取；轮询间隔走 concern.emitInterval

xhs:                         # [next-dev]
  cookies:
    a1:
    web_session:
    webId:
  interval: 15s
  onlyOnlineNotify: false

twitch:                      # [next-dev]
  clientId:
  clientSecret:
  interval: 30s
  onlyOnlineNotify: false

heybox:                      # [next-dev]（站点名 xhh）
  x_xhh_tokenid:             # 可选，未配置时自动生成
  interval: 30s
  onlyOnlineNotify: false

twitcasting:                 # 该段由 [next] 起生成；键本身在 master 也被读取
  clientId:
  clientSecret:
  nameStrategy: name
  broadcaster:
    title: false
    created: false
    image: false

concern:
  emitInterval: 5s

dispatch:
  largeNotifyLimit: 50

notify:
  parallel: 1

template:
  enable: true               # 生成配置 true；键缺失时为 false

autoreply:
  group:
    command: []
  private:
    command: []

cronjob:
  - cron: "* * * * *"
    templateName: "定时1"
    target:
      private: [123]
      group: []

customCommandPrefix:
  签到: ""

imagePool:                   # 该段由 [next-dev] 生成
  type: "off"                # localPool / loliconPool / off；生成文件写的是 lolicon（代码不认，会记录"未知的图片池"）

localPool:
  imageDir:

loliconPool:
  apikey:                    # 已不需要
  cacheMin: 20
  cacheMax: 50
  proxy:

proxy:                       # 该段由 [next-dev] 生成；master / next 需手写
  type: "off"                # systemProxy（[next-dev] 未配置时的默认）/ localProxyPool / pyProxyPool / off；master / next 未配置时记录 "unknown proxy type"，不启用代理

localProxyPool:
  oversea:
    - 127.0.0.1:8888
  mainland:
    - 127.0.0.1:8888

pyProxyPool:
  host: "http://127.0.0.1:8000"   # 生成值；代码未配置时为空

adapter:                     # [next-dev]
  mode: onebot-v11           # onebot-v11（默认）/ satori

websocket:
  mode: ws-server
  token:
  ws-server: 0.0.0.0:15630   # master / next 默认；[next-dev] 默认 127.0.0.1:15630，且监听非回环地址时必须配 token
  ws-reverse: ws://localhost:3001

admin:                       # [next-dev]
  enable: false
  addr: "127.0.0.1:15631"
  token: ""

platform:                    # [next-dev]，热重载
  weibo:
    enable: true

message-marker:
  disable: false

qq-logs:
  enable: false              # [next-dev] 生成的配置写 true

extDb:                       # [next]
  enable: false
  path: ".ext.db"

telegram:                    # [next]
  enable: false
  token: ""
  endpoint: ""
  chats: []

debug:
  group:
    - 0
  uin:
    - 0

logLevel: info
```

!!! warning "已失效 / 需注意的字段"
    - `bot.account` / `bot.password`：不在自动生成的配置里，QQ 连接由 OneBot 实现端负责。`[next-dev]` 起代码完全不读取；`master` / `next` 只有内置 MiraiGo 初始化会读到（`bot/bot/bot.go`），而登录调用已停用（根 `bot.go` 里 `bot.Login()` 被注释），填了不起作用。
    - `sendFailureReminder.enable` / `.times`：`master` / `next` 有效，发送累计失败达到 `times` 次触发 `notify.bot.send_failed.tmpl`；`[next-dev]` 只订阅事件、没有发布点，提醒不会触发。
    - `reloadDelay`：`master` / `next` 生成并生效（连上实现端后延迟加载好友/群/群员信息）；`[next-dev]` 起已无此键。
    - `twitter.cfclearance`：三个分支都没有这个配置键，也没有读取点；代码只在抓取时识别 Cloudflare 拦截并报错，无法通过配置注入。
    - `weibo.alertGroupId` / `weibo.alertQQList`：`[next-dev]` 生成的键，getter 均已无调用者（`alertGroupId` 的 getter 另标了 `// Deprecated:`），设置无效；告警目标改由 `/config weibo_alert` 控制。
    - `acfun.minFollowerCap`：`[next-dev]` 生成该键，但代码没有任何读取点，设置无效。
    - `douyu.interval` / `huya.interval`：`[next-dev]` 生成该键，但代码没有读取点；斗鱼/虎牙轮询间隔走 `concern.emitInterval`。

---

## 字段分类速查

按功能模块分类，点击进入详细说明：

| 模块 | 关键字段 | 说明 |
|------|---------|------|
| [bot 与连接](bot.md) | `bot.*`、`websocket.*`、`debug`、`message-marker`、`qq-logs`、`adapter.*`（`[next-dev]`） | 账号、命令前缀、掉线策略、OneBot/适配器连接、调试 |
| [B 站](bilibili.md) | `bilibili.*` | cookie/扫码登录、检测间隔、图片合并、关注策略、专栏解析 |
| [推特](twitter.md) | `twitter.*` | nitter 镜像、UA（master）；api 模式、自动翻译（[next-dev]） |
| [抖音](douyin.md) | `douyin.*` | cookie/UA、人机验证、live（WSa 新增，测试）；news 与 sessionId（[next]） |
| [其它订阅源](other-sources.md) | `acfun.*`、`weibo.*`、`youtube.*`、`douyu.*`、`huya.*`、`twitcasting.*` | A 站/微博/YouTube/斗鱼/虎牙/TwitCasting；小红书/Twitch/小黑盒为 `[next-dev]` |
| [代理与图片池](proxy-imagepool.md) | `proxy.*`、`localProxyPool.*`、`pyProxyPool.*`、`imagePool.*`、`localPool.*`、`loliconPool.*` | 代理池、图库 |
| [推送与模板开关](push-template.md) | `concern.*`、`dispatch.*`、`notify.*`、`template.*`、`autoreply.*`、`cronjob.*`、`customCommandPrefix` | 推送调度、模板、自定义命令、定时消息 |
| [Web 管理面板](../deploy/webui.md) | `admin.*`、`platform.<site>.enable` | 面板启停与地址（[next-dev]）；按平台停用轮询 / 推送 |

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

不配置 B 站账号时，订阅数**不要超过 5 个**，否则推送延迟上升；配置 B 站账号后代码日志称最高可支持 2000 订阅。

---

## Web 管理面板（`admin`，[next-dev]）

`[next-dev]` 内置 Web 管理面板，**默认关闭**：

```yaml
admin:
  enable: false              # 默认 false；不开则完全不启动管理服务
  addr: "127.0.0.1:15631"    # 监听地址，默认仅本机可访问
  token: ""                  # 访问密码；留空时首次打开进入初始化向导
```

| 字段 | 默认 | 说明 |
|------|------|------|
| `admin.enable` | `false` | 是否启动 Web 管理面板。关闭时不会监听任何端口 |
| `admin.addr` | `127.0.0.1:15631` | 面板监听地址。默认仅本机可访问 |
| `admin.token` | 空 | 面板登录密码。留空时所有数据接口返回 401，只能进入初始化向导 |

`admin` 配置修改后需重启；只有初始化向导写入的 `admin.token` 会在运行期热生效。详见 [Web 管理面板](../deploy/webui.md)。

!!! warning "面板默认只监听本机"
    不要把面板直接暴露到公网。确需远程访问时用反向代理加 HTTPS，并设置足够强的密码。

---

## 平台级启停（`platform`，[next-dev]）

`[next-dev]` 可以按平台单独停用轮询与推送：

```yaml
platform:
  weibo:
    enable: false            # 停用微博；缺省或 true 视为启用
```

| 取值 | 行为 |
|------|------|
| 缺省 / `true` | 平台正常轮询与推送 |
| `false` | 停止该平台的 API 请求与推送；**已有订阅数据保留**，改回 `true` 即继续 |

`site` 用订阅源标识（如 `bilibili`、`weibo`、`twitter`、`douyin`、`acfun`、`twitch`）。该开关走配置热重载，**无需重启**。

---

## 配置热重载

DDBOT 用 viper 的 `WatchConfig` 监听配置文件变化，以下配置**修改后无需重启**：

- `customCommandPrefix`（自定义命令前缀）
- `cronjob`（定时消息）
- `platform.<site>.enable`（平台级启停，`[next-dev]`）

`bot.commandPrefix`、`debug.*` 等每次读取配置的键也会即时生效；其余多数配置项修改后需要重启 DDBOT。

---

## 下一步

- 按需打开左侧子页面配置各模块
- [命令手册](../commands/index.md) -- 配好后学习命令
- [常见问题](../faq.md) -- 配置出错先看这里
