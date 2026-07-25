# 从纯血 DDBOT 迁移

如果你之前在用纯血 DDBOT（Sora233/DDBOT），想迁移到 DDBOT-WSa，本页给出注意事项。

---

## 迁移思路

DDBOT-WSa 复用了 DDBOT 的数据库结构（`.lsp.db`），但底层从「直接登录 QQ」改成了「OneBot WebSocket」。迁移的核心是：

1. 备份原 `.lsp.db`
2. 替换程序文件
3. **修改数据库里的字段前缀**（`ae` -> `ex`），让 WSa 能正确初始化
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

### 4. 修改数据库字段

这是迁移的关键一步。用文本编辑器（VSCode / Notepad++，**不要用记事本**）打开 `.lsp.db`：

1. 搜索 `ae` 字段
2. **全部替换为 `ex` 字段**
3. 保存

只需把 `ae` 改成 `ex`，DDBOT-WSa 就能正确初始化数据库。

!!! warning "为什么是 ae -> ex"
    纯血 DDBOT 和 WSa 在数据库里用了不同的 key 前缀来标记某些内部状态。直接用旧库启动 WSa 会因为找不到预期的前缀而初始化失败。改前缀是最简单的兼容方式。

!!! note "@全体成员可能失效"
    迁移后 `@全体成员` 配置可能失效，建议重新配置一遍 `/config at_all`。

### 5. 配置 WebSocket

编辑 `application.yaml`，加上 WebSocket 配置（纯血 DDBOT 没有这一段）：

```yaml
websocket:
  mode: ws-server
  ws-server: 0.0.0.0:15630
```

然后按 [连接 OneBot](index.md) 对接一个实现端。

### 6. 启动并验证

```bash
./DDBOT
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
