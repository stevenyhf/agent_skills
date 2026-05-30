---
name: douban-movie
version: 1.0.0
description: >
  豆瓣电影查询与影评分析技能。搜索电影、获取评分/演职员/短评/影评等信息，
  生成综合分析报告。支持通过片名搜索或直接传入豆瓣链接。
  触发条件：提及豆瓣电影、电影查询、影评分析、电影评分、看电影。
tags: [douban, movie, film-review, rating, 豆瓣, 电影, 影评]
---

# 豆瓣电影查询与影评分析

## 功能概述

通过豆瓣电影页面和第三方API获取电影信息，生成包含评分、演职员、剧情、
短评摘要、影评分析的综合报告。

## 数据源（按优先级）

### 1. web_extract 直接抓取豆瓣页面（首选）
- 豆瓣电影详情页可通过 web_extract 直接访问，无需登录
- URL格式：`https://movie.douban.com/subject/{id}/`
- 可获取：片名、评分、评分分布、演职员、奖项、热门短评标题和票数
- 局限：剧情简介可能被"展开"按钮截断；评论正文需要JS渲染

### 2. api.wmdb.tv 第三方API（结构化补充）
- 免费、无需认证
- 搜索：`https://api.wmdb.tv/api/v1/movie/search?q={keyword}&limit=5`
- 详情：`https://api.wmdb.tv/movie/api?id={douban_id}`
- 返回：IMDB评分、烂番茄评分、演员表、剧情简介等

### 3. web_search 搜索豆瓣链接
- 当用户只提供片名（无URL）时，用搜索找到豆瓣页面
- 查询格式：`"{片名}" site:movie.douban.com`

## 执行工作流

### 步骤1：获取电影URL
- 用户提供URL → 直接使用
- 用户提供片名 → web_search `"片名" site:movie.douban.com` 找到URL
- 提取 douban subject ID（URL中的数字）

### 步骤2：并行获取数据
同时调用：
1. `web_extract` 抓取豆瓣电影详情页
2. `web_extract(urls=[wmdb_url])` 从wmdb.tv获取补充数据（如有douban_id）

### 步骤3：并行获取扩展数据（新增）
同时调用以下数据源，用于新增的四大分析板块：

1. **时代背景**：`web_search` 搜索 `"{片名}" 导演 拍摄背景 创作动机 幕后`
2. **社会影响**：`web_search` 搜索 `"{片名}" 社会影响 文化现象 讨论`
3. **多平台评论**：`web_search` 搜索 `"{片名}" site:rottentomatoes.com OR site:metacritic.com OR site:imdb.com`
4. **精选豆瓣评论**：通过豆瓣影评列表页获取最热影评，
   抓取前3篇影评详情页（`/review/{id}/`），提取完整影评全文

### 步骤4：解析与整合
从豆瓣页面提取：
- 片名（中英文）
- 年份、导演、编剧、主演
- 评分、评分分布
- 类型、制片国家/地区、语言、片长
- 奖项信息
- 热门短评（标题+有用数）
- 热门影评（标题+链接ID）

从wmdb.tv补充：
- 剧情简介（豆瓣页面可能截断）
- IMDB评分、烂番茄评分（如有）

从web_search补充：
- 导演创作背景、拍摄动机
- 社会文化影响报道
- 多平台评分与评论摘要

### 步骤5：生成分析报告
按以下结构输出（中文）：

```
标题：《{片名}》深度解析

一、基本信息
  片名、年份、导演、编剧、主演、类型、国家、语言、片长

二、评分分析
  豆瓣评分及分布、IMDB评分（如有）、评分解读

三、剧情概述
  来自wmdb或页面提取的剧情简介

四、主创团队分析
  导演风格、主要演员、创作背景

五、荣誉与影响
  获奖情况、行业影响

六、拍摄背景与时代文化因素
  （新增）该片拍摄时的时代背景、社会语境、文化潮流，
  导演的创作动机是什么——什么事件、情绪或思潮促使导演拍这部电影？
  引用导演访谈、幕后花絮、新闻报道中的原始素材。

七、当代社会与文化影响
  （新增）这部电影对当下社会、文化产生了哪些影响？
  是否引发了公共讨论、改变了流行文化、催生了亚文化现象？
  影片的主题在今天是否仍有现实意义？

八、多平台评论汇总
  （新增）来自IMDB、烂番茄（Rotten Tomatoes）、Metacritic等
  国际影评平台的评分和核心评价观点。
  各平台之间的评分差异及其可能的原因分析。

九、豆瓣精选用户评论深度分析
  （新增）选取豆瓣用户评论中最优秀的3篇（按"有用"数排序），
  对每篇进行深度总结：
  - 评论者核心观点
  - 评论者的分析角度和论证逻辑
  - 最有价值的洞察
  - 三篇评论之间的共识与分歧
  最后综合提炼用户评论中的集体智慧。

十、观众反馈
  热门短评摘要、观众关注焦点

十一、综合评价
  基于以上全部信息的独立分析与推荐
```

## 注意事项

1. **反爬机制（实测2026-05-30）**：
   - 豆瓣详情页有时返回"载入中..."（JS动态渲染未完成），不可靠
   - 豆瓣影评详情页（/review/{id}/）可稳定抓取，含完整影评全文
   - wmdb.tv API始终可用，是结构化数据的可靠来源
   - 降级策略：详情页失败时，用wmdb.tv API（含剧情/评分/演员）+
     web_search结果中的描述片段 + 影评页面组合生成报告

2. **数据完整性**：web_extract返回的是页面摘要（约5000字符），
   超长页面会被截断。优先使用关键信息字段。

3. **用户偏好**：用户偏好简洁中文回复，输出到本地文件时使用
   `~/` 目录，临时文件用 `/tmp/`。

4. **影评风格**：如需写影评，参考 haofeng-movie-skill 的
   写作风格和六维评价体系。

5. **批量查询**：多次查询时注意间隔（建议>2秒），避免触发
   豆瓣频率限制。

## 可用脚本

### 搜索并获取电影信息（execute_code示例）
```python
from hermes_tools import web_search, web_extract

# Step 1: 搜索
results = web_search(query='"肖申克的救赎" site:movie.douban.com', limit=3)
url = results['data']['web'][0]['url']  # 取第一个结果

# Step 2: 提取详情
page = web_extract(urls=[url])
print(page)
```

### wmdb.tv API 补充数据
```python
from hermes_tools import web_extract

# 搜索（按关键词）
search_url = "https://api.wmdb.tv/api/v1/movie/search?q=肖申克的救赎&limit=3"
# 详情（按douban ID）
detail_url = "https://api.wmdb.tv/movie/api?id=1292052"
result = web_extract(urls=[detail_url])
```
