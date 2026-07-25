# 升级与维护

本页讲 DDBOT-WSa 的版本升级、数据库维护、保活与排障。

---

## 升级 DDBOT

### 升级步骤

1. **停止当前运行的 DDBOT**
2. 备份 `.lsp.db` 和 `application.yaml`
3. 用新版二进制替换旧版程序文件
4. 重新启动

```bash
# 停止（systemd）
sudo systemctl stop ddbot

# 备份
cp .lsp.db .lsp.db.bak.$(date +%Y%m%d)
cp application.yaml application.yaml.bak

# 替换程序（下载新版后）
mv DDBOT-new ./DDBOT
chmod +x ./DDBOT

# 启动
sudo systemctl start ddbot
```

### 数据库自动迁移

DDBOT 内置数据库版本迁移机制。启动时如果检测到旧版本数据库，会：

1. 自动备份当前数据库到 `<原文件名>-<时间戳>`
2. 日志提示「五秒后将开始更新数据库」
3. 执行迁移

!!! warning "迁移期间不要中断"
    看到迁移提示后**不要按 Ctrl+C 中断**，否则可能导致数据库损坏。迁移通常很快完成。

### 检查更新

DDBOT 启动后每 24 小时检查一次新版本，有更新会私聊通知管理员。收到通知后按上面步骤升级即可。

如果不想接收更新通知，私聊 BOT 发：

```
/退订更新
```

---

## 数据库维护

### `.lsp.db` 是什么

`.lsp.db` 是 [buntdb](https://github.com/tidwall/buntdb) 嵌入式数据库文件，存储：

- 所有订阅关系
- 权限配置（管理员、群管理员、命令权限）
- 群组配置（@全体、过滤、推送开关）
- 消息缓存、禁言状态
- 版本号、好友/群申请记录

### 文件锁

DDBOT 启动时会对 `.lsp.db` 加文件锁（`.lsp.db.lock`），**防止重复启动**导致数据库损坏。

如果启动报错：

```
tryLock数据库失败：您可能重复启动了这个BOT！
如果您确认没有重复启动，请删除.lsp.db.lock文件并重新运行。
```

- 先确认没有另一个 DDBOT 在跑
- 确认后删除 `.lsp.db.lock` 文件再启动

### 数据库损坏修复

启动报错或闪退，可能是数据库损坏。尝试：

1. **备份** `.lsp.db`
2. 用文本编辑器打开 `.lsp.db`，**删掉最后一行**，保存后重试启动
3. 重复上面步骤数十次，仍不行则到交流群求助

详见 [FAQ](../faq.md)。

### 重置出厂

删除 `.lsp.db` 即恢复出厂设置（**所有订阅和权限丢失**）。仅在数据库严重损坏且无法修复时使用。

### 运维工具

[buntdb-cli](https://github.com/Sora233/buntdb-cli) 可作为运维工具查看/修改数据库。

!!! danger "不要在 DDBOT 运行时使用 buntdb-cli"
    buntdb 不支持多写，DDBOT 运行时用 cli 操作会导致数据库损坏。务必先停止 DDBOT。

---

## 进程保活

DDBOT-WSa 进程退出后不会自动重启，建议配合进程管理器实现开机自启与崩溃拉起：

- **Linux**：systemd（`Restart=on-failure`），示例见 [安装 - 后台常驻运行](install.md)
- **Windows**：nssm 注册为服务
- **Docker**：`restart: unless-stopped`

!!! info "关于「掉线重连」"
    纯血 DDBOT 时代有 `bot.onDisconnected`、`session.token` 等 QQ 协议层的重连机制。DDBOT-WSa 把登录交给了 OneBot 实现端，**IM 连接的稳定性由实现端负责**，DDBOT 自身不再处理协议层重连。

    - 如果是 DDBOT 进程崩溃：靠上面的进程管理器拉起即可
    - 如果是 IM 连接掉线：检查 OneBot 实现端的日志与重连配置（各实现端有自己的重连策略）
    - `bot.onDisconnected` 配置项仍保留但作用已弱化，一般无需关心

---

## 离线缓存

WSa 支持离线缓存（v0.4.0+），BOT 离线期间的消息暂存，上线后补发：

```yaml
bot:
  offlineQueue:
    enable: false      # 是否启用离线缓存
    expire: 30m        # 离线消息有效期
```

!!! note "不能重启 DDBOT"
    离线缓存存在内存里，**期间不能重启 DDBOT**，否则缓存丢失。重启 DDBOT 等同于清空缓存。

---

## 发送失败提醒

当消息多次发送失败时，可触发提醒模板通知管理员：

```yaml
bot:
  sendFailureReminder:
    enable: false       # 是否启用
    times: 3            # 失败次数阈值
```

触发时会执行模板 `notify.bot.send_failed.tmpl`（需自行创建），可用变量：`message`、`target_id`、`target_type`、`target_name`、`times`。

---

## 异常订阅清理

群被封、BOT 离线期间被踢等情况，会导致 BOT 仍向不存在的群推送，增加封号风险。

定期检查：

```
/检测异常订阅
```

清理异常订阅：

```
/清除订阅 --abnormal
```

详见 [管理员命令](../commands/admin.md)。

---

## 日志

日志默认写到 `logs/` 目录，按天滚动，保留 7 天。日志等级在 `application.yaml` 配置：

```yaml
logLevel: info   # trace / debug / info / warn / error
```

- 排障时临时调到 `debug` 或 `trace` 看详细信息
- 正常运行用 `info` 即可
- `trace`/`debug` 会输出大量 base64，谨慎使用

管理员可临时查日志：

```
/log 20              # 最近 20 条
/log 20 -k error     # 含 error 的最近 20 条
```

---

## 下一步

- [配置参考](../config/index.md) -- 所有配置项
- [管理员命令](../commands/admin.md) -- 运维相关命令
- [常见问题](../faq.md) -- 排障
