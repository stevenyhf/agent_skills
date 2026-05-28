---
name: topic-article-publisher
description: >
  话题深度分析 → 飞书云文档 → 微信公众号草稿箱 全流程发布工作流。
  当用户提到"写一篇文章"、"深度分析后发布到公众号"、"话题转公众号文章"、"research and publish"、
  "帮我分析XX并发布到公众号"等场景时使用此 skill。
  流程：(1) 深度分析话题 (2) 生成飞书云文档 (3) 从飞书文档发布到微信公众号草稿箱 (4) IM 通知。
  依赖：autoglm-deepresearch、feishu-create-doc(lark-cli)、wechat_native_publish。
---

# Topic → Article → Publish 工作流

将用户提出的话题，经过深度分析、飞书文档沉淀、一键发布到微信公众号草稿箱的端到端工作流。

## 依赖

| 依赖 | 用途 | 触发方式 |
|------|------|----------|
| `autoglm-deepresearch` | 深度研究分析话题 | 读取 SKILL.md，按流程执行搜索和深读 |
| `lark-cli docs +create` | 创建飞书云文档 | 命令行调用（需用户授权 docx:document:create） |
| `scripts/wechat_native_publish.py` | 发布到微信公众号草稿箱 | 传入 AppID + AppSecret + Markdown 文件 |
| `feishu_fetch_doc` 工具 | 读取飞书文档内容（备选） | 直接调用工具获取文档 Markdown |
| `message` 工具 | IM 通知 | 直接调用工具发送消息 |

## 前置配置

使用前需确保：

1. **飞书 OAuth 授权**：`lark-cli auth login --scope "docx:document:create"` 已完成
2. **微信公众号**：AppID + AppSecret 已获取，且当前机器 IP 已加入微信公众号 IP 白名单
   - 白名单配置路径：微信公众平台 → 设置与开发 → 基本配置 → IP白名单

---

## 工作流分两阶段

### 阶段一：话题 → 深度分析 → 飞书文档

**触发**：用户提出话题或事件。

**执行步骤**：

1. **确认话题**：明确用户的分析需求（话题、角度、篇幅偏好等），如有不确定直接问。

2. **深度研究**：
   - 读取 `autoglm-deepresearch` 的 SKILL.md
   - 按其流程执行：拆解子问题 → 1-2 轮搜索 → 1-3 个页面深读 → 综合生成报告
   - 控制调用次数：`web-search.py` ≤ 2 次，`open-link.py` ≤ 3 次

3. **生成飞书文档**：
   - 将深度研究报告整理为适合公众号阅读的标准 Markdown 格式
   - 文档结构建议：
     ```
     # [话题名称]
     ## 核心观点 / 导读
     ## 背景
     ## 深度分析
     ## 典型案例
     ## 趋势展望
     ## 总结
     ```
   - 使用 `lark-cli docs +create` 创建飞书云文档
   - 文档标题格式：`[深度分析] + 话题名称`
   - 同时将 Markdown 保存到本地文件（供阶段二发布使用）

4. **返回结果**：
   - 向用户回复：飞书文档已创建，附上文档链接
   - 询问用户是否需要发布到微信公众号

**阶段一输出**：飞书云文档 URL + 本地 Markdown 文件路径

---

### 阶段二：飞书文档 → 微信公众号草稿箱

**触发**：用户确认发布（或直接要求发布）。

**执行步骤**：

1. **准备内容**：
   - 如果有本地 Markdown 文件，直接使用
   - 否则通过 `feishu_fetch_doc` 工具获取飞书文档的 Markdown 内容
   - 确认文章标题不超过 64 字符（微信公众号限制）

2. **发布到微信草稿箱**：
   - 脚本路径：`scripts/wechat_native_publish.py`
   - 执行命令：
     ```bash
     python scripts/wechat_native_publish.py <AppID> <AppSecret> <markdown_file> [author]
     ```
   - 脚本自动完成：获取 access_token → 生成封面图 → 上传永久素材 → 创建草稿
   - **只发布到草稿箱，绝不自动发布**

3. **IM 通知**：
   - 发布成功后，通过 `message` 工具或直接回复发送通知
   - 通知内容：
     ```
     ✅ 文章已发布到公众号草稿箱

     📄 标题：{article_title}
     📝 作者：{author}
     ⏰ 发布时间：{current_time}

     请登录微信公众平台预览并发布。
     ```
   - 如果发布失败，发送错误通知并附上原因

---

## 脚本说明

### wechat_native_publish.py

微信公众号原生 API 发布脚本，无第三方依赖。

**用法**：
```bash
python wechat_native_publish.py <AppID> <AppSecret> <markdown_file> [author]
```

**参数**：

| 参数 | 必填 | 说明 |
|------|------|------|
| AppID | 是 | 微信公众号 AppID |
| AppSecret | 是 | 微信公众号 AppSecret |
| markdown_file | 是 | Markdown 文件路径 |
| author | 否 | 作者名称 |

**功能**：
1. 通过 AppID + AppSecret 获取 access_token
2. 自动生成蓝紫色封面 PNG 图片
3. 上传为永久素材（草稿接口要求）
4. 将 Markdown 转换为微信公众号兼容的 HTML（内联样式）
5. 创建草稿

**输出**：JSON 格式，包含 `success`、`media_id`、`title`、`digest`

**返回码处理**：

| errcode | 说明 | 解决方案 |
|---------|------|----------|
| 40164 | IP 不在白名单 | 将当前 IP 加入微信公众平台白名单 |
| 40001 | access_token 无效 | 检查 AppID/Secret 是否正确 |
| 40007 | media_id 无效 | 脚本已自动使用永久素材上传 |
| 45009 | 接口调用频率超限 | 稍后重试 |

---

## 注意事项

### 深度研究阶段
- 搜索和深读次数严格控制，避免过多 API 调用
- 优先展示中间发现，让用户参与决策
- 内容要适合公众号阅读风格，避免纯学术腔调

### 文档创建阶段
- 飞书文档需要用户 OAuth 授权 `docx:document:create` 和 `docx:document` scope
- 如果 lark-cli 权限不足，提示用户运行 `lark-cli auth login --scope "docx:document:create docx:document"`
- 始终将 Markdown 保存到本地，作为发布到微信的备份

### 发布阶段
- **只发布到草稿箱，绝不自动发布**
- 需要当前机器 IP 在微信公众号 IP 白名单中
- 标题超长时截断到 64 字符并告知用户
- 自动生成的封面图为蓝紫色渐变，用户可在微信后台替换

### IM 通知
- 在飞书对话中直接回复通知即可
- 通知内容简洁清晰，包含关键信息
