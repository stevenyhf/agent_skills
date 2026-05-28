---
name: logistic-everyday-news
description: |
  从网页抓取物流资讯并写入飞书多维表格。
version: 0.1.0
author: fangxia 
license: MIT
---
# Skill Overview
## 功能概述
- 每天早上8点，请帮我浏览 http://www.chinawuliu.com.cn/zixun/ 这个网站，然后总结前100条资讯的核心内容，然后帮我写入待创建的物流每日资讯多维表格，这个表格包括以下字段：日期，资讯关联公司名，主要内容，重要性
## 入口
- **入口文件**：`index.js`
- **导出函数**：`run(context)`
## 参数 (可选)
| 参数 | 类型 | 必须 | 说明 |
|------|------|------|------|
| `url` | string | 是 | 要抓取的页面 URL |
| `count` | number | 否 | 抓取条目数量，默认 100 |
## 示例调用
```json
{
  "action": "run",
  "skill": "logistic-everyday-news",
  "params": {
    "url": "https://www.chinawuliu.com.cn/zixun",
    "count": 20
  }
}
