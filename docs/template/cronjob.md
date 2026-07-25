# 定时消息

DDBOT 支持通过模板发送定时消息，按 cron 表达式定时触发。

---

## 配置

在 `application.yaml` 的 `cronjob` 段定义定时消息：

```yaml
cronjob:
  - cron: "* * * * *"
    templateName: "定时1"
    target:
      private: [123]
      group: []
  - cron: "0 * * * *"
    templateName: "定时2"
    target:
      private: []
      group: [456]
```

| 字段 | 说明 |
|------|------|
| `cron` | 5 字段 cron 表达式（分 时 日 月 周），最小粒度 1 分钟 |
| `templateName` | 模板名，对应 `custom.cronjob.<名字>.tmpl` |
| `target.private` | 私聊发送的 QQ 号列表 |
| `target.group` | 群聊发送的群号列表 |

---

## 创建模板文件

每条定时消息对应一个模板：

- `定时1` -> `template/custom.cronjob.定时1.tmpl`
- `定时2` -> `template/custom.cronjob.定时2.tmpl`

---

## cron 表达式

DDBOT 使用 5 字段的 Linux cron 表达式：

| 字段 | 含义 | 范围 |
|------|------|------|
| 第 1 位 | 分钟 | 0-59 |
| 第 2 位 | 小时 | 0-23 |
| 第 3 位 | 日 | 1-31 |
| 第 4 位 | 月 | 1-12 |
| 第 5 位 | 周 | 0-6（0=周日） |

### 示例

| 表达式 | 含义 |
|--------|------|
| `* * * * *` | 每分钟 |
| `0 * * * *` | 每小时整点 |
| `0 9 * * *` | 每天 9:00 |
| `*/30 * * * *` | 每 30 分钟 |
| `0 9 * * 1-5` | 工作日 9:00 |
| `0 0 1 * *` | 每月 1 号 0:00 |

!!! tip "cron 工具"
    在 [tool.lu/crontab](https://tool.lu/crontab/)（选「类型：Linux」）编辑和测试 cron 表达式。

---

## 配置示例

### 每分钟私聊

```yaml
cronjob:
  - cron: "* * * * *"
    templateName: "每分钟提醒"
    target:
      private: [123456]
      group: []
```

`定时1` 每分钟触发，私聊 QQ 123456 发送 `custom.cronjob.每分钟提醒.tmpl` 内容。

### 每小时群聊

```yaml
cronjob:
  - cron: "0 * * * *"
    templateName: "整点报时"
    target:
      private: []
      group: [456789]
```

每小时整点在群 456789 发送报时模板。

### 每日早安

```yaml
cronjob:
  - cron: "0 8 * * *"
    templateName: "早安"
    target:
      private: []
      group: [456789]
```

模板内容：

```text
早上好！现在是 {{ hour }} 点 {{ minute }} 分
今天是 {{ year }} 年 {{ month }} 月 {{ day }} 日
{{ choose "今天也要加油喵" "新的一天开始啦" "元气满满" }}
```

---

## 可用变量

定时消息模板可使用时间函数：

```text
现在是 {{ year }}-{{ month }}-{{ day }} {{ hour }}:{{ minute }}:{{ second }}
今天是本周第 {{ weekday }} 天
今天是今年第 {{ yearday }} 天
```

也可使用 `cooldown` 避免重复触发，`choose`/`roll` 增加随机性等，详见 [模板函数](funcs.md)。

---

## 热重载

`cronjob` 配置修改后**自动重载**，无需重启 DDBOT。

---

## 下一步

- [自定义命令](custom-command.md) -- 创建交互式命令
- [模板函数](funcs.md) -- 时间函数与其它
- [推送与模板开关 - 定时消息](../config/push-template.md) -- 配置项说明
