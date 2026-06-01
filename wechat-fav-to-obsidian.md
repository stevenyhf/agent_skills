# WeChat Favorites to Obsidian Sync

将微信收藏内容安全同步到 Obsidian 的技能。

## 核心原则

> **微信从未开放个人"收藏夹"导出 API。任何绕过官方渠道的方案（逆向协议、解密数据库、Frida Hook）都有极高封号风险。**

本技能仅使用微信官方认可的通道：**小程序** 和 **公众号**，确保零封号风险。

## 触发条件

- "微信收藏同步到Obsidian"
- "WeChat favorites to Obsidian"
- "微信收藏导出 markdown"
- "导出微信收藏"
- "微信笔记同步方案"
- "wechat to obsidian"

## 同步架构（安全路径）

### 方案 A：小程序桥接（推荐）

**原理**：用户在收藏内容上点击"分享"转发到小程序，小程序存储数据后，Obsidian 插件同步拉取。

**安全性**：零风险，完全使用微信官方 OpenAPI

**工作流**：
```
微信收藏 → 用户转发 → 小程序 → CloudBase → Obsidian插件拉取 → .md文件
```

**实施步骤**（见 `references/solution-a-miniapp.md`）：
1. 部署 wechat-inbox-sync 小程序到微信 CloudBase
2. 在 Obsidian 安装 wechat-inbox-sync 社区插件
3. 配置 API Key 绑定
4. 日常使用：长按收藏 → 分享 → 发给小程序

### 方案 B：公众号桥接（自建）

**原理**：收藏内容转发到公众号 → 公众号通过官方消息API接收 → 服务端写入 Markdown 文件 → Obsidian 同步

**安全性**：零风险，使用官方消息接收 API

**工作流**：
```
微信收藏 → 用户转发 → 公众号 → 服务端脚本 → .md文件 → Obsidian同步
```

**实施步骤**（见 `references/solution-b-official-account.md`）：
1. 注册微信公众号/服务号
2. 开启开发者模式，配置消息接收 URL
3. 部署服务端脚本（见 `scripts/wechat_receiver.py`）
4. Obsidian 通过 git/Remotely Save 同步

### 方案 C：macOS 辅助功能（仅 macOS）

**原理**：AppleScript 控制微信 UI 界面，逐个点击收藏内容并导出网页

**安全性**：模拟人工操作，目前未被封号，但非官方

**实施步骤**（见 `references/solution-c-macos.md`）

## 安全红线

### 绝对不要做的事

| 做法 | 风险 | 后果 |
|------|------|------|
| 逆向 WeChat 协议直读收藏数据 | 极高 | 2025年起大规模封号，甚至永久封禁 |
| 解密本地 EnMicroMsg.db | 极高 | 微信持续升级数据库加密，检测可识别 |
| Frida Hook 微信进程 | 极高 | 进程完整性检测可识别 |
| 使用非官方 Web WeChat API | 高 | Web 微信已被限制 |
| 自动化批量模拟登录 | 极高 | 触发验证码加账号限制 |

### 安全的做法

| 做法 | 安全性 | 原理 |
|------|--------|------|
| 转发给小程序 | 绿色 | 使用微信官方小程序 OpenAPI |
| 转发给公众号 | 绿色 | 使用微信官方消息接收 API |
| 手动导出网页 | 绿色 | 收藏中的网页链接可直接打开后保存 |

## 当用户使用此 Skill 时

1. 询问用户的技术背景：是否有开发经验、是否是 macOS 用户、需求是历史批量导出还是长期增量同步
2. 根据需求推荐方案：长期同步选 A，自建选 B，大批量历史选 C
3. 提供具体的实施指导，引导用户完成配置

## 输出格式规范

同步到 Obsidian 的笔记统一使用以下 Markdown 模板：

```markdown
---
title: "{标题}"
source: wechat-favorite
type: "{type/text/link/image/file/audio}"
wechat_id: "{消息ID}"
created_at: "{创建时间}"
tags:
  - wechat
  - "{自动生成的标签}"
---

# {标题}

来源: 微信收藏
保存时间: {YYYY-MM-DD HH:mm:ss}
类型: {content_type}

---

{正文内容}

{附带链接/图片/附件描述}
```

## 相关文件

- 方案 A 详解: `references/solution-a-miniapp.md`
- 方案 B 详解: `references/solution-b-official-account.md`
- 方案 C 详解: `references/solution-c-macos.md`
- 公众号服务端脚本: `scripts/wechat_receiver.py`
- 安全指南: `references/safety-guide.md`
- Obsidian 配置: `references/obsidian-setup.md`
- FAQ: `references/faq.md`
