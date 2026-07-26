---
title: DDBOT-WSa 文档
description: 基于 OneBot 协议的消息推送框架
hide:
  - navigation
  - toc
---

# DDBOT-WSa

<p class="hero-subtitle">基于 OneBot 协议的消息推送框架</p>

只要对接任意一个 OneBot 11 实现端，就能把 B 站、斗鱼、虎牙、ACFun、YouTube、微博、推特、抖音等平台的直播/动态更新，推送到 OneBot 协议所连接的 IM 平台。

<span class="hero-actions">
  <a class="primary-button" href="quickstart/">快速开始</a>
  <a class="secondary-button" href="deploy/intro/">了解 DDBOT</a>
</span>

---

## 你能用它做什么

- **直播 / 动态推送** —— B 站、斗鱼、虎牙、ACFun、YouTube、微博、推特、抖音、TwitCasting，订阅后开播 / 发动态自动推送。
- **精细控制推送** —— 按关键字、动态类型过滤；@全体成员或指定人；下播提醒、标题变更提醒；防刷屏去重。
- **模板与自定义命令** —— 用 `text/template` 语法自定义所有推送 / 命令 / 事件格式；自定义命令回复；定时消息（cron）。
- **插件扩展** —— 实现 `Concern` 接口即可接入任意订阅源，框架负责轮询、去重、限流、持久化。

---

## 文档导航

- **[快速开始](quickstart/)** —— 3 步跑通：下载运行 → 连接 OneBot → 完成第一次订阅
- **[部署与连接](deploy/intro/)** —— 安装、首次配置、对接 OneBot 实现端、媒体、迁移、升级
- **[版本与分支](deploy/branches/)** —— master / next / next-dev 分支差异与选择建议
- **[配置参考](config/)** —— `application.yaml` 全字段表，按模块逐项说明
- **[命令手册](commands/)** —— 速查表 + 每条命令的详细用法与示例
- **[订阅源](sources/)** —— master 分支 9 个内置订阅源，含推特 / 抖音等 WSa 新增
- **[模板系统](template/)** —— 语法、变量、全部模板函数、命令 / 推送 / 事件模板
- **[插件开发](plugin/)** —— 从脚手架到完整订阅源，附 Twitter / 抖音范本走查
- **[常见问题](faq/)** —— 部署排障、风控、订阅不推送等高频问题

---

## 关于 DDBOT-WSa

DDBOT 由 [Sora233](https://github.com/Sora233/DDBOT) 开发，最初基于 MiraiGo 直接走 QQ 协议。后续演进为多条分支：

| 分支 | 维护者 | 定位 |
|------|--------|------|
| **DDBOT**（纯血） | Sora233 | 原版，直接走 QQ 协议（MiraiGo 失效，已不可用） |
| DDBOT-ws | Hoshinonyaruko | 二次修改，改为接入 OneBot 协议 |
| **DDBOT-WSa** | cnxysoft | 三次修改，恢复 DDBOT 原有功能 + 接入 OneBot 11 |

**DDBOT-WSa** 不再直接对接 QQ 协议，而是通过 **OneBot 11 WebSocket** 与实现端通信。OneBot 是一套通用 IM 协议规范，理论上任何支持该协议的平台都能接入（QQ 是目前最主流的实现目标，但不局限于此）。

> DDBOT **不是聊天机器人**。它只在「订阅对象有更新」和「答复命令」时主动发言，刻意把交互设计到最小程度，正常聊天永远不会误触它。

!!! info "本文档基于 master 分支"
    本站内容以 **master 分支**为准。`next` / `next-dev` 分支有更多实验性功能（小红书、Twitch、小黑盒订阅源等），差异说明见 [版本与分支](deploy/branches/)。

---

## 关于本文档

本站是 DDBOT-WSa 的**文档中心**，由 [kizunerwe](https://github.com/kizunerwe) 基于官方资料与源码整理编写，旨在提供一份清晰、全面、跟得上版本演进的文档。

**参考与致谢**

本站在编写过程中参考了以下前人文档与资料，在此致谢：

- **官方文档与源码**：[cnxysoft/DDBOT-WSa](https://github.com/cnxysoft/DDBOT-WSa)（仓库 README、INSTALL/EXAMPLE/TEMPLATE/FAQ/UPDATE/FRAMEWORK 等 md，以及源码注释）
- **晴风《DDBOT 安装教程》**：[ddbot.songlist.icu](https://ddbot.songlist.icu/)（VuePress 搭建的早期社区文档，本站的信息架构与 OneBot 对接章节参考了它的组织方式）
- **Sora233 的 B 站专栏**：[cv10602230](https://www.bilibili.com/read/cv10602230)（DDBOT 原版介绍）

如发现文档有错误或过时内容，欢迎到 [Issues](https://github.com/cnxysoft/DDBOT-WSa/issues) 反馈，或加交流群 `980848391`。

---

## 交流与反馈

- **源码仓库**：[cnxysoft/DDBOT-WSa](https://github.com/cnxysoft/DDBOT-WSa)
- **文档问题反馈**：[Issues](https://github.com/cnxysoft/DDBOT-WSa/issues)
- **交流群**：980848391（755612788 已满）
- **B 站专栏介绍**：[cv10602230](https://www.bilibili.com/read/cv10602230)
