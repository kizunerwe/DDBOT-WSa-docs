# 常见问题

提问前请先看本页，仍未解决再咨询交流群 `980848391` 或到 [Issues](https://github.com/cnxysoft/DDBOT-WSa/issues) 反馈。

---

## 部署与连接

### Q：下载的程序怎么运行？

- Windows 程序应该有 `.exe` 后缀，双击运行
- 根据系统选 `windows`/`linux`/`darwin`，根据架构选 `386`/`amd64`/`arm`/`arm64`
- Windows 11/10/7/Server 64 位选 `windows-amd64`

详见 [安装](deploy/install.md)。

### Q：启动后闪退？

通常是 `application.yaml` 格式错误：

- 删除 `#` 及后面的注释
- 所有引号、冒号用**英文半角**
- 冒号后必须有一个英文空格

详见 [首次配置](deploy/first-config.md)。

### Q：提示「tryLock数据库失败：您可能重复启动了这个BOT」

DDBOT 对 `.lsp.db` 加了文件锁防重复启动：

1. 先确认没有另一个 DDBOT 在跑
2. 确认后删除 `.lsp.db.lock` 文件再启动

### Q：WebSocket 连不上？

- 确认 DDBOT 已启动，日志显示 `WebSocket server started`
- 确认端口 15630 没被占用
- 确认 `websocket.mode` 两边对应：DDBOT 用 `ws-server` 时，实现端必须用「反向 WS」连过来
- 确认 token 一致

详见 [连接 OneBot](deploy/connect/index.md)。

### Q：连上了但收不到消息？

- 确认 BOT 已被拉进群或已是好友
- 确认 `reloadDelay` 延迟加载已完成（看日志）
- 确认 BOT 未被禁言

### Q：macOS 提示「无法验证开发者」

系统设置 -> 隐私与安全性 -> 仍要打开。或终端运行：

```bash
xattr -d com.apple.quarantine ./DDBOT
```

---

## B 站相关

### Q：为什么设置了 @全体成员却没效果？

**官方 bot**：@全体次数在所有群共享，用尽即失效，用 @特定用户替代。

**私人部署**：

1. 检查是否把 BOT 设为 QQ 群管理员
2. 检查 `/config at_all` 是否设置为 `on`
3. 同一主播每 2 小时只会 @全体一次

### Q：为什么无法订阅 B 站？

- 检查 `application.yaml` 的 `bilibili` 配置是否完整
- 检查账号/密码或 cookie 格式
- 检查 `minFollowerCap` 粉丝数门槛
- 新号可能提示「账户异常」（B 站侧限制）

cookie 格式：

```yaml
bilibili:
  SESSDATA: "xxxxxxxxxxxx"
  bili_jct: "xxxxxxxxxxxx"
```

!!! warning "cookie 注意"
    - 不要点 B 站「退出登录」，否则 cookie 失效
    - 用浏览器「清除历史记录」或隐私窗口登录获取
    - cookie 可能较长，确认复制完整

详见 [B 站配置](config/bilibili.md)。

### Q：订阅了动态/直播没推送？

- 检查 `/config filter` 是否设了过滤
- 检查日志是否有 `notify` 记录和发送失败
- 检查 BOT 是否被禁言

### Q：B 站 cookie 过期了怎么办？

- 改用扫码登录：`qrlogin: true`，清空 `SESSDATA`/`bili_jct` 重启
- 或重新获取 cookie 填入

---

## 风控与封号

### Q：为什么群消息发送失败？

新 BOT 大概率是 QQ 账号被风控：

- 用 BOT 挂机 3-7 天（期间**不要使用 BOT 功能**，否则可能封号）解除风控
- DDBOT-WSa 不再依赖 `device.json` 登录，此文件已退化为占位，删除它通常无效。重点检查 OneBot 实现端是否正常连接、账号是否被风控

### Q：为什么后台有反应但 QQ 收不到？

- 大概率风控，按上面方法挂机
- 检查 OneBot 实现端日志，看消息是否被实现端拒收或上报失败
- 实在不行可尝试在实现端侧重新登录、清理登录态

### Q：BOT 掉线无法重连？

DDBOT-WSa 把 IM 登录交给 OneBot 实现端，**连接稳定性由实现端负责**：

- DDBOT 进程崩溃：用进程管理器（systemd / nssm / Docker `restart`）拉起，见 [升级与维护](deploy/upgrade.md#进程保活)
- IM 连接掉线：检查 OneBot 实现端的日志与重连配置，各实现端有自己的重连策略
- 纯血 DDBOT 时代的 `session.token`、`bot.onDisconnected` 等协议层重连机制在 WSa 中已弱化，一般无需关心

---

## 数据库

### Q：数据库文件损坏怎么办？

**操作前备份 `.lsp.db`**，然后：

1. 用文本编辑器打开 `.lsp.db`
2. 删掉最后一行
3. 保存后重试启动
4. 重复数十次仍不行则到交流群求助

### Q：如何恢复出厂设置？

删除 `.lsp.db` 即恢复出厂（所有订阅和权限丢失）。

### Q：能用 buntdb-cli 操作数据库吗？

可以，但**不要在 DDBOT 运行时操作**（buntdb 不支持多写）。务必先停止 DDBOT。

---

## 推特 / 抖音

### Q：推特抓不到推文？

- 镜像不可用：换一个或多配几个 `baseUrl`
- UA 被拒：填入真实浏览器 `userAgent`
- CF 防护：配 `cfclearance`

详见 [推特配置](config/twitter.md)。

### Q：推特视频推不出来？

确认已安装 FFmpeg 且在 PATH 或程序目录。见 [媒体与 FFmpeg](deploy/connect/media.md)。

### Q：抖音订阅没启动？

- 检查 `acSignature` 和 `acNonce` 是否都填了
- 看日志是否有 `Stop` 提示
- 一直触发人机验证：调大 `interval`，确认 UA 与 cookie 一致

详见 [抖音配置](config/douyin.md)。

---

## 模板

### Q：改了模板不生效？

- 确认 `template.enable: true`
- 确认模板文件在 `template/` 目录下，后缀 `.tmpl`
- 确认文件名与内置模板完全一致（区分大小写）
- DDBOT 会自动热重载，无需重启

### Q：模板输出多了空行？

用 `{{-` 和 `-}}` 控制空白：

```text
{{- reply .msg -}}
```

详见 [语法与变量 - 空白控制](template/syntax.md)。

---

## 迁移

### Q：从纯血 DDBOT 迁移要注意什么？

- 备份 `.lsp.db`
- 用文本编辑器把 `.lsp.db` 里的 `ae` 替换为 `ex`
- 配置 `websocket` 段
- @全体成员可能失效，建议重新配置

详见 [从纯血 DDBOT 迁移](deploy/connect/migrate.md)。

---

## 交流与反馈

- **GitHub Issues**：[cnxysoft/DDBOT-WSa/issues](https://github.com/cnxysoft/DDBOT-WSa/issues)
- **交流群**：980848391（755612788 已满）
- **B 站专栏**：[cv10602230](https://www.bilibili.com/read/cv10602230)
