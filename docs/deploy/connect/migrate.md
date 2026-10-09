# 从纯血 DDBOT 迁移

如果你之前在用纯血 DDBOT（Sora233/DDBOT），想迁移到 DDBOT-WSa，本页给出注意事项。

---

## 迁移思路

DDBOT-WSa 复用了 DDBOT 的数据库结构（`.lsp.db`），但底层从「直接登录 QQ」改成了「OneBot WebSocket」。迁移的核心是：

1. 备份原 `.lsp.db`
2. 替换程序文件
3. 启动 WSa，让内置的数据库版本迁移自动执行
4. 配置 WebSocket 连接一个 OneBot 实现端

---

## 步骤

### 1. 停止旧版 DDBOT

确保旧版程序已完全退出，且 `.lsp.db` 没有被占用。

### 2. 备份

```bash
cp .lsp.db .lsp.db.bak
cp application.yaml application.yaml.bak
```

!!! danger "务必先备份"
    数据库迁移有风险，操作前一定要备份。万一出错可以用备份恢复。

### 3. 替换程序

把旧的 DDBOT 程序文件替换为 DDBOT-WSa 的新版本（下载见 [安装](../install.md)）。

### 4. 让程序自动迁移数据库

旧库不需要手工改动。WSa 启动时会检查数据库版本，检测到旧版本会：

1. 先把当前数据库备份为 `.lsp.db-<Unix 时间戳>`
2. 日志提示「五秒后将开始更新数据库」
3. 自动执行迁移（把旧的订阅类型位标记转换成 `live` / `news` 类型串）

!!! warning "迁移期间不要中断"
    看到迁移提示后不要按 Ctrl+C，等它执行完。

!!! note "@全体成员可能失效"
    迁移后 `@全体成员` 配置可能失效，建议重新配置一遍 `/config at_all`。

### 5. 配置 WebSocket

编辑 `application.yaml`，加上 WebSocket 配置（直接以 WSa 首次运行生成的新配置为准更省事）：

```yaml
websocket:
  mode: ws-server
  ws-server: 0.0.0.0:15630   # master / next 默认监听全部网卡
```

`master` / `next` 默认监听 `0.0.0.0:15630`，且 ws-server **不校验 `token`**。`[next-dev]` 默认收紧为 `127.0.0.1:15630`：未配置 `token` 时只允许监听回环地址；若实现端在**其它机器或容器**里、需把 `ws-server` 改成它可达的地址，则**必须配置 `token`**，否则拒绝启动。

然后按 [连接 OneBot](index.md) 对接一个实现端。

### 6. 启动并验证

```bash
./DDBOT-WSa
```

检查日志：

- 数据库版本迁移是否成功
- WebSocket 是否启动
- OneBot 实现端是否连上

私聊 BOT 发 `/sysinfo` 查看好友/群组/订阅数，与迁移前对比。

---

## 配置文件变化

纯血 DDBOT 的部分配置项在 WSa 里有调整，建议**用 WSa 生成的新配置为准**，把旧配置里有用的值填进去：

- 新增 `websocket` 段（必须）
- 新增 `twitter`、`douyin` 段（WSa 新增订阅源，可选）
- `bilibili` 段字段基本兼容，新增了 `qrlogin`、`autoParsePosts` 等
- 旧版的 `sign-server` 不再需要

完整字段见 [配置参考](../../config/index.md)。

---

## 订阅数据保留

迁移后原有的订阅（B 站、斗鱼、虎牙等）会保留，无需重新订阅。但建议检查：

- `/list` 查看订阅列表是否完整
- `/检测异常订阅` 检查是否有订阅但 BOT 不在群的情况（迁移过程中群关系可能变化）

---

## 不建议回退

WSa 的数据库迁移是单向的，迁移后不建议再回退到纯血 DDBOT。如必须回退，用第 2 步的备份恢复 `.lsp.db` 和程序。

---

## 下一步

- [连接 OneBot](index.md) -- 对接实现端
- [配置参考](../../config/index.md) -- 了解新配置项
- [常见问题](../../faq.md) -- 迁移遇到问题先看这里
