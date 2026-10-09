# 云崽 ws-plugin 对接

如果你已经在用 [云崽（Yunzai）](https://github.com/TimeRainStarSky/Yunzai) 生态，可以通过 [ws-plugin](https://github.com/XasYer/ws-plugin) 让云崽作为 OneBot 实现端连接 DDBOT。

---

## 适用场景

- 已有云崽机器人，不想再单独部署 LLOneBot/NapCat
- 想用云崽的 icqq 协议端来收发 QQ 消息

---

## 安装云崽与 ws-plugin

### 1. 安装前置

- [Git](https://git-scm.com/downloads)
- [Node.js](https://nodejs.org/)（推荐 18+）

### 2. 安装云崽（TRSS-Yunzai）

在合适目录右键打开 Git Bash，运行：

```bash
git clone --depth 1 https://github.com/TimeRainStarSky/Yunzai ./Yunzai/TRSS-Yunzai
cd Yunzai/TRSS-Yunzai
npm --registry=https://registry.npmmirror.com install pnpm -g
pnpm config set registry https://registry.npmmirror.com
```

### 3. 安装 ws-plugin

```bash
git clone --depth=1 https://github.com/XasYer/ws-plugin.git ./plugins/ws-plugin/
pnpm i
```

### 4. 启动 Redis 数据库

云崽依赖 Redis，按 TRSS-Yunzai 的说明启动 redis。

### 5. 启动云崽

```bash
node app
```

按提示登录 BOT 的 QQ 账号。

---

## 配置 ws-plugin 连接 DDBOT

启动云崽后，ws-plugin 会生成配置文件。在 ws-plugin 的连接配置里新增一个**正向 WebSocket**（连向 DDBOT 的 ws-server）：

| 配置项 | 值 |
|--------|----|
| 连接类型 | 正向 WebSocket（WebSocket Client） |
| 地址 | `ws://127.0.0.1:15630/ws` |
| Token | 通常留空；`master` / `next` 的 ws-server 不校验 token，`[next-dev]` 配置了 `websocket.token` 时需填一致的值 |

!!! tip "DDBOT 侧配置"
    DDBOT 保持默认正向模式：

    ```yaml
    websocket:
      mode: ws-server
      ws-server: 0.0.0.0:15630   # master / next 默认监听全部网卡
    ```

    `master` / `next` 默认监听全部网卡，ws-plugin（云崽）用 `ws://127.0.0.1:15630/ws` 即可连入；这两个分支的 ws-server **不校验 `token`**，ws-plugin 侧留空即可。

    `[next-dev]` 默认收紧为 `127.0.0.1:15630`：未配置 `token` 时只允许监听回环地址；若 ws-plugin（云崽）在**其它机器或容器**里、需把 `ws-server` 改成它可达的地址，则**必须配置 `token`**，否则拒绝启动。

配置保存后 ws-plugin 会自动重连。

---

## 验证连接

1. DDBOT 日志出现 `有新的ws连接了!!`（`[next-dev]` 为 `WebSocket client connected (ws-server mode)`）
2. 私聊 BOT 发 `/ping`，收到 `pong` 即正常

---

## 注意事项

- **不要关闭云崽和 DDBOT 本体**，两者需同时运行
- ws-plugin 已适配 DDBOT 的 at 操作和错误提示（见 0.4.0 更新日志）
- 如需更新 icqq，按云崽文档操作，注意更新后可能需要重新登录

---

## 参考

- ws-plugin：https://github.com/XasYer/ws-plugin
- TRSS-Yunzai：https://github.com/TimeRainStarSky/Yunzai
