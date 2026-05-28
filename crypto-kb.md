---
name: crypto-kb
description: "加密货币知识库：基于比特币白皮书和以太坊白皮书回答加密货币、加密经济、加密技术、数字货币、区块链相关问题。"
---

# 加密货币知识库 (Crypto Knowledge Base)

基于两份核心白皮书回答加密货币相关问题：
1. **比特币白皮书** - 中本聪《比特币：一种点对点的电子现金系统》(2008)
2. **以太坊白皮书** - Vitalik Buterin《以太坊：下一代智能合约和去中心化应用平台》(2014)

## 触发场景

当用户询问以下主题时激活此skill：
- 加密货币（cryptocurrency）
- 加密经济（crypto economy / tokenomics）
- 加密技术（cryptography in blockchain）
- 数字货币（digital currency）
- 区块链基础（blockchain fundamentals）
- 共识机制（PoW/PoS/consensus）
- 智能合约（smart contracts）
- 去中心化（decentralization）
- 比特币或以太坊的技术原理

## 参考资料

详细知识要点存放在 `references/` 目录：
- `references/bitcoin-whitepaper.md` — 比特币白皮书核心知识
- `references/ethereum-whitepaper.md` — 以太坊白皮书核心知识
- `references/cross-reference.md` — 两份白皮书的对比索引与FAQ定位指南

## 回答规范

1. **引用来源**：回答时明确标注引用自"比特币白皮书"还是"以太坊白皮书"
2. **精确性**：使用白皮书中的原始概念和术语，不臆造未提及的内容
3. **对比分析**：当问题涉及两者时，对比两份白皮书的观点差异
4. **局限性**：白皮书未涵盖的内容（如2015年后的发展）应明确标注"超出白皮书范围"
5. **语言**：使用中文回答，技术术语保留英文原文并解释

## 知识分类

### 加密技术基础
- 数字签名（ECDSA）
- 哈希函数（SHA-256）
- 公钥密码学
- 零知识证明（ZKP）概念
- 默克尔树（Merkle Tree）

### 经济模型
- 发行机制与货币政策
- 交易手续费市场
- 激励相容设计
- 双重支付问题与解决方案
- 51%攻击与安全性

### 共识机制
- 工作量证明（PoW）
- 权益证明（PoS）
- 最长链规则
- 难度调整算法

### 协议与架构
- P2P网络拓扑
- 区块结构与传播
- 交易验证流程
- 状态转换模型（以太坊）
- EVM（以太坊虚拟机）

### 应用场景
- 电子现金与价值转移
- 智能合约与DApp
- 去中心化金融（DeFi）概念基础
- 数字身份与签名
