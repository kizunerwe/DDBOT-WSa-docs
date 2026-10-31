# 首次配置

首次运行 DDBOT 后，本页带你完成最小可用配置：设置管理员、连接 OneBot、确认能收发消息。

完整的 `application.yaml` 字段说明见 [配置参考](../config/index.md)，这里只讲最小必要项。

---

## 配置文件在哪

DDBOT 启动时会在**当前目录**读取 `application.yaml`，如果不存在会自动生成一个最小配置。用任意文本编辑器打开它（Windows 记事本 / VSCode / nano / vim 均可）。

!!! warning "YAML 格式注意事项"
    - 填写时**删除井号 `#` 及后面的注释**
    - **冒号后必须有一个英文空格**
    - 所有标点用**英文（半角）**，不要用中文标点
    - 字符串含特殊字符时用英文引号括起来

    格式错误会导致启动闪退，详见 [FAQ](../faq.md)。

---

## 最小配置（扫码登录 + B 站测试）

默认生成的 `application.yaml` 已经能直接用，**不需要改任何东西**就能完成「下载运行 -> 连接 -> 订阅」的全流程。下面是一份等价的**最小可用配置**，方便看清关键项（实际生成的文件字段更多，还含 `twitter`、`douyin`、`autoreply`、`customCommandPrefix`、`reloadDelay`、`sendFailureReminder`、`offlineQueue` 等；`[next-dev]` 生成的文件不含 `reloadDelay` 与 `sendFailureReminder`）。

```yaml
bot:
  onJoinGroup:
    rename: "【bot】"   # 进群自动改名，留空则不改

bilibili:
  qrlogin: true         # 扫码登录 B 站（不填账号密码）
  interval: 25s

concern:
  emitInterval: 5s

template:
  enable: true          # 启用模板（推荐）

websocket:
  mode: ws-server       # 正向 WS，默认监听 15630
  ws-server: 0.0.0.0:15630   # master / next 默认监听全部网卡（[next-dev] 默认 127.0.0.1:15630）

logLevel: info
```

!!! info "不配置 B 站账号也能用"
    不填 B 站账号/cookie 时，BOT 也能订阅 B 站，但**推荐订阅不超过 5 个**，否则推送延迟会上升。要大规模订阅 B 站，请配置账号，见 [B 站配置](../config/bilibili.md)。

---

## 设置管理员

BOT 必须有管理员才能正常使用管理命令。两种方式：

### 方法一：私聊 `/whosyourdaddy`（推荐）

1. 先完成 [连接 OneBot](connect/index.md)，让 BOT 能收发消息
2. 用你自己的 QQ **私聊 BOT 账号**发送：

    ```
    /whosyourdaddy
    ```

3. BOT 回复「成功 - 您已成为bot管理员」即完成

!!! note "此命令仅在 BOT 没有管理员时生效"
    一旦设置了管理员，后续再发此命令无效。注意 `--set-admin` 只是**新增**管理员；要移除旧管理员，可用管理员账号私聊 BOT 发送 `/grant -r Admin -d <QQ>`，或操作数据库。

### 方法二：命令行 `--set-admin`（BOT 未运行时）

```bash
./DDBOT-WSa --set-admin 你的QQ号码
```

执行后立即退出。该命令需要打开 `.lsp.db` 并加文件锁，所以**必须先停掉正在运行的 DDBOT**；否则会因拿不到锁而失败，设置不会生效。

---

## Web 初始化向导（next-dev，可选）

如果你部署的是 `next-dev` 且启用了 [Web 管理面板](webui.md)，第一次打开面板（`admin.token` 为空）会进入 4 步初始化向导：

1. **设置访问密码** —— 即面板登录密码，写入 `admin.token`，设置后无需重启即可登录
2. **填写 OneBot 连接** —— 可选，支持正向 / 反向；写入后需重启 BOT 才生效，也可整步跳过
3. **选择主题** —— 仅保存在浏览器本地，不写入配置文件
4. **确认** —— 完成页给出后续操作指引

!!! note "向导设置的是面板密码，不是 BOT 管理员"
    向导只负责 Web 面板密码和可选的 OneBot 连接；BOT 的 QQ 管理员仍需用上面的方法一 / 方法二设置。

---

## 设置运行模式

BOT 有三种工作模式，控制对「好友申请」和「加群邀请」的处理：

| 模式 | 好友申请 | 加群邀请 | 命令 |
|------|---------|---------|------|
| **公开**（默认） | 自动同意 | 自动同意 | `/mode 公开` |
| **私人** | 自动拒绝 | 自动拒绝 | `/mode 私人` |
| **审核** | 保留待审 | 保留待审 | `/mode 审核` |

私聊 BOT 发送：

```
/mode 私人
```

!!! tip "推荐私人部署用「私人」模式"
    私人部署建议切到 `私人` 模式，避免被陌生人随意拉群。需要时手动登录 BOT QQ 主动加好友/群。

    审核模式下，可用 `/好友申请` 和 `/群邀请` 命令逐条审批。

---

## 减少刷屏输出

如果已经熟悉命令，可对 BOT 私聊发送 `/silence` 开启沉默模式，全局减少输出：

- 不再提示「命令权限不够」「命令被禁用」「命令语法错误」「命令帮助」

沉默模式也可对单个群开启，见 [命令手册](../commands/admin.md)。

!!! warning "新手不要急着开"
    不熟悉命令时开启沉默模式，会让你看不到错误提示，反而更难排查问题。

---

## 关键文件说明

| 文件 | 作用 | 删除后果 |
|------|------|---------|
| `application.yaml` | 配置文件 | 重新生成最小配置 |
| `device.json` | 历史遗留的设备信息文件（纯血 DDBOT 时代用于伪装设备） | 在 DDBOT-WSa 中已退化为占位文件，删除无实质影响，会自动重新生成 |
| `.lsp.db` | 数据库（订阅/权限/配置） | **恢复出厂设置**，所有订阅丢失 |
| `session.token` | 登录会话（历史遗留） | 在 DDBOT-WSa 中已无实际作用 |
| `logs/` | 日志（保留 7 天） | 历史日志丢失 |
| `template/` | 自定义模板 | 模板丢失，回退到内置默认 |

---

## 下一步

- [连接 OneBot](connect/index.md) -- 对接你的实现端，让 BOT 真正能收发 QQ 消息
- [配置参考](../config/index.md) -- 按需开启 B 站账号、推特、抖音、代理等
- [命令手册](../commands/index.md) -- 学习所有命令
