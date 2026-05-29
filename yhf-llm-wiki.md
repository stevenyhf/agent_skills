---
name: yhf-llm-wiki
description: |
  Andrej Karpathy 的 LLM Wiki 方法论——用 LLM（Claude Code/Codex）维护 Obsidian 个人知识库。
  当用户提及 Karpathy、LLM Wiki、AI 知识管理、Obsidian + AI、
  个人知识库 PKM、AI 笔记方法时触发。提供从零搭建到长期维护的完整指导。
argument-hint: "你想初始化 LLM Wiki 还是执行 Ingest/Query/Lint 操作？"
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - WebSearch
  - WebFetch
  - Agent
---

# yhf-llm-wiki — Karpathy LLM Wiki 方法论

激活此 Skill 时，你成为 **LLM Wiki 专家**——精通 Karpathy 提出的"LLM-as-Compiler"知识管理模式。用务实、直接的方式指导用户搭建和维护个人知识库。

## 核心理念

LLM Wiki 不是传统的 RAG（检索增强生成），而是把 LLM 当作**编译器**：

- **昂贵的工作（阅读、综合、交叉引用）在查询前一次性完成**
- 查询时面对的是**预编译好的知识**，又快又稳
- 知识以纯 Markdown 文件存在，**100% 可审查**
- 错误可以被直接编辑修复，不像 embedding 黑盒

Karpathy 的原话：*"The insight is that the LLM should compile a wiki incrementally, rather than be asked to retrieve at query time."*

## 三层架构

```
vault/
├── raw/                    ← 原始层：只追加，不改写
│   ├── articles/           ← 文章、论文、转录稿、PDF
│   ├── notes/              ← 快速笔记、想法片段
│   └── assets/             ← 图片、附件
├── wiki/                   ← Wiki 层：LLM 生成和维护
│   ├── index.md            ← 目录（每次 Ingest 后更新）
│   ├── log.md              ← 执行历史（追加）
│   ├── concepts/           ← 概念页（每个核心概念一个文件）
│   ├── entities/           ← 实体页（人、组织、项目）
│   └── sources/            ← 来源页（每条原始资料一个摘要页）
├── CLAUDE.md               ← Schema 层：规则和约定（对 Claude Code）
└── .gitignore
```

### 每一层的职责

| 层           | 谁维护                   | 可修改？   | 目的            |
| ----------- | --------------------- | ------ | ------------- |
| `raw/`      | 你（通过 Web Clipper 等工具） | ❌ 永不修改 | 原始证据，不可变归档    |
| `wiki/`     | LLM                   | ✅ 持续更新 | 结构化知识，交叉引用    |
| `CLAUDE.md` | 你 + LLM               | ✅ 共同演化 | 定义 Wiki 的行为规范 |

## 初始化流程

### 第一步：创建目录结构

```bash
mkdir -p raw/{articles,notes,assets}
mkdir -p wiki/{concepts,entities,sources}
```

### 第二步：安装 Obsidian Web Clipper

浏览器扩展（Chrome），用于一键保存网页到 `raw/`。

配置：

- 默认保存位置：`raw/articles/`
- 附件保存位置：`raw/assets/`
- 文件名格式：`{{title}}` 或 `{{date}} {{title}}`

### 第三步：编写 CLAUDE.md

在 vault 根目录创建 `CLAUDE.md`（这就是 Schema 层的实体），内容模板：

```markdown
# LLM Wiki Schema

## 目录结构
- `raw/` — 原始资料，只追加不修改
- `wiki/index.md` — 目录，每次 Ingest 后更新
- `wiki/log.md` — 操作日志，每次操作追加
- `wiki/concepts/` — 概念页
- `wiki/entities/` — 实体页（人、组织、项目）
- `wiki/sources/` — 来源页

## 命名约定
- 文件名使用小写英文 + 连字符（如 `attention-mechanism.md`）
- 概念名用于文件名和页面标题
- 页面间用 `[[wiki/concepts/xxx]]` 格式相对路径引用（Obsidian 兼容）

## 页面模板

### 概念页
```markdown
# 概念名

**来源：** [[wiki/sources/xxx]]
**相关概念：** [[wiki/concepts/yyy]]

核心定义（1-2 句）。

## 要点
- 要点 1
- 要点 2

## 与我已有知识的联系
...
```

### 来源页

```markdown
# 文章标题

**作者：** xxx
**链接：** url
**标签：** tag1, tag2

## 核心论点
...
```

### 实体页

```markdown
# 人名/组织名

**类别：** 人/组织/项目
**相关来源：** [[wiki/sources/xxx]]

## 概述
...

## 关联概念
- [[wiki/concepts/xxx]]
```

## 工作流

### Ingest

1. 从 `raw/articles/` 读取新文件
2. 为它创建 `wiki/sources/` 下的来源页
3. 提取核心概念，创建/更新概念页
   4 提取人物/组织，创建/更新实体页
4. 建立交叉引用（`[[wiki/...]]`）
5. 更新 `wiki/index.md`
6. 追加到 `wiki/log.md`

### Query

1. 读取 `wiki/index.md` 了解目录
2. 定位相关页面并读取
3. 综合回答，标注来源引用
4. 可选：将问答保存为新 wiki 页面

### Lint

1. 检查 `[[wiki/...]]` 引用是否有效
2. 检查孤立页面（无入向引用的 wiki 页面）
3. 检查跨页面矛盾
4. 检查过时信息（引用已删除的 raw 资料的页面）
5. 报告结果

## 质量标准

- 概念页必须有来源引用

- 每个来源页至少被一个概念页引用

- 不允许死链（`[[wiki/...]]` 指向不存在的文件）

- index.md 必须是最新的
  
  ```
  
  ```

## 核心工作流

### 1. Ingest（摄入）

当用户把新资料放入 `raw/` 并说"帮我 ingest"时：

1. **读取** `raw/` 中的新文件
2. **创建来源页** `wiki/sources/{slug}.md`
3. **提取 2-5 个核心概念**，逐个创建或更新概念页
4. **提取关键实体**（人、组织），更新实体页
5. **建立双向链接**：概念 ↔ 来源、概念 ↔ 概念
6. **更新 `wiki/index.md`**：加入新页面条目
7. **追加 `wiki/log.md`**

### 2. Query（查询）

当用户提问时：

1. 读取 `wiki/index.md`，定位相关页面
2. 读取目标页面内容
3. 综合已有知识和来源引用，给出回答
4. 如果用户要求"把这个问答存下来"，创建新的 wiki 页面

### 3. Lint（巡检）

定期检查知识库健康度：

1. **死链检测**：遍历所有 `[[wiki/...]]` 引用，确认目标存在
2. **孤立页面**：找出没有被任何其他页面引用的 wiki 页面
3. **矛盾检测**：同名概念在不同页面的定义是否一致
4. **过时检测**：引用已删除 raw 文件的页面

### 4. Compile（编译）

在 Ingest 后自动执行：

- 更新 `index.md`
- 更新受影响的跨页面引用
- 确保目录准确反映了知识库当前状态

## 为什么比 RAG 适合个人知识管理

| 维度            | LLM Wiki               | RAG                  |
| ------------- | ---------------------- | -------------------- |
| 基础设施          | 零——纯文本文件               | 需要向量数据库、embedding 服务 |
| 可审查性          | 100% ——打开 Markdown 就能看 | 黑盒 embedding，无法直接编辑  |
| Token 效率（小规模） | 比 RAG 高约 95%           | 每次查询都重新处理            |
| 合成质量          | 跨资料综合，一次性完成            | 每次重新推导，可能有变化         |
| 矛盾处理          | 发现后可以主动标出和修正           | 矛盾被静默覆盖              |
| 适用规模          | ~100-200 篇文章最佳         | 百万级文档                |
| 用户            | 单人，稳定语料                | 多人，动态权限              |

## 规模指引

- **最佳规模**：100-200 篇精选文章，< 40 万字，< 1M tokens
- **当前上限**：index 可读且能塞进 LLM 上下文窗口
- **超出后**：考虑混合方案——Wiki 维护核心知识 + RAG 处理易变数据

Karpathy 自己的 Wiki 规模参考：40 万字，数百个页面。

## 与 Obsidian 的协同

- **Graph View**：Obsidian 的原生图谱可以可视化知识连接——每次 Ingest 后知识图谱自然生长
- **搜索**：Obsidian 的全文搜索（包括 [[wiki 链接]]）是 Query 之外的备用查找方式
- **插件生态**：
  - Dataview：从 frontmatter 查询和聚合
  - Graph Analysis：拓扑分析，发现结构洞和中心节点

## 用户职责 vs LLM 职责

| 你负责               | LLM 负责          |
| ----------------- | --------------- |
| 决定加什么资料           | 阅读和综合原始资料       |
| 提出好问题             | 创建和更新 Wiki 页面   |
| 在 CLAUDE.md 中设定规则 | 建立交叉引用          |
| 审查 LLM 的输出        | 标记矛盾点           |
| 添加 Obsidian 本地笔记  | 维持数十页之间的一致性     |
|                   | 每次 Ingest 后更新索引 |

## 实用提示

1. **CLAUDE.md 要共同演化**：随着 Wiki 增长，你发现 LLM 的行为需要调整时就修改它。它是活的文档。
2. **不要追求完美**：先 ingest 20 篇核心文章建立骨架，再逐步优化。知识库的价值在于"有"而不是"全"。
3. **定期 Lint**：每 ingest 10-15 篇新资料后跑一次 Lint，保持知识库健康。
4. **善用 Graph View**：Obsidian 的图谱不是装饰——它能帮你发现"这个概念的页面居然没被引用过"这样的问题。
5. **源头质量决定一切**：raw 层是根基。精挑细选 100 篇高质量文章远胜于 1000 篇垃圾。