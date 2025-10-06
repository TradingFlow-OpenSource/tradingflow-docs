# Writing Style Guide | 书写风格指南

This document defines the writing style and formatting standards for TradingFlow documentation.

本文档定义了 TradingFlow 文档的书写风格和格式标准。

---

## Core Principles | 核心原则

### Narrative Over Lists | 段落叙述为主

Documentation should be **primarily narrative** with bullet points used judiciously for emphasis. The recommended balance is:

文档应该**以段落叙述为主**，适度使用 bullet points 进行强调。推荐的比例是：

- **Paragraph Content**: 50-70% of content
- **Bullet Points**: 30-50% of content

**段落内容**: 50-70%
**Bullet Points**: 30-50%

### When to Use Bullet Points | 何时使用列表

Bullet points are appropriate for:

适合使用 bullet points 的场景：

- Listing concrete features or capabilities
- Showing advantages or benefits
- Enumerating supported options
- Highlighting key specifications

Avoid using bullet points for:

避免在以下情况使用 bullet points：

- Explaining complex concepts
- Describing workflows or processes
- Providing context and background
- Narrative descriptions

---

## Structure Examples | 结构示例

### ✅ Good Example | 好的示例

```markdown
# On-Chain Vaults

TradingFlow follows the principles of decentralized applications and does not custody users' keys, wallets, or assets in any form. Therefore, when users choose to deploy workflows on a blockchain network, they need to create corresponding on-chain "vaults" in advance.

These vaults belong solely to the users themselves, with only the users having the rights to deposit and withdraw. When a user's configured workflow determines that a transaction is needed, TradingFlow sends a signal to the on-chain vault.

## How It Works

On-chain vaults adopt a non-custodial architecture design. The system sends an execution signal to the user's on-chain vault rather than directly operating user assets. After receiving the signal, the vault smart contract completes trading operations through decentralized exchanges.

## Supported Networks

Currently, TradingFlow supports:

- **Aptos** - High-performance Move language blockchain
- **Flow EVM** - Ethereum-compatible Flow ecosystem
```

**Analysis | 分析:**
- Opening paragraphs provide context (段落提供上下文)
- Technical details in narrative form (技术细节采用叙述形式)
- Bullet points only for concrete list of supported networks (仅在列举具体网络时使用列表)
- Ratio: ~70% paragraphs, ~30% bullet points (比例: ~70% 段落, ~30% 列表)

### ❌ Poor Example | 不好的示例

```markdown
# On-Chain Vaults

## What are On-Chain Vaults?

- TradingFlow doesn't custody keys
- TradingFlow doesn't custody wallets  
- TradingFlow doesn't custody assets
- Users need to create vaults
- Vaults are on blockchain networks

## Features

- Fully decentralized
- User controlled
- Independent access rights
- No third party trust needed

## How It Works

1. Workflow determines transaction needed
2. TradingFlow sends signal
3. Transaction executes on DEX
4. Process is transparent
```

**Problems | 问题:**
- Excessive fragmentation (过度碎片化)
- Lacks narrative flow (缺乏叙事流畅性)
- Too many bullet points (bullet points 过多)
- No contextual depth (缺少深度说明)
- Ratio: ~10% paragraphs, ~90% bullet points (比例: ~10% 段落, ~90% 列表)

---

## Formatting Guidelines | 格式指南

### Headings | 标题

- **H1 (#)**: Document title only (仅文档标题)
- **H2 (##)**: Main sections (主要章节)
- **H3 (###)**: Subsections (子章节) - use sparingly (谨慎使用)

Avoid overuse of H3 headings. If you find yourself needing H3, consider if the content could be reorganized.

避免过度使用 H3 标题。如果需要 H3，考虑内容是否可以重新组织。

### Paragraphs | 段落

- Keep paragraphs focused on one main idea
- Use 2-4 sentences per paragraph typically
- Connect paragraphs with logical flow
- Avoid single-sentence paragraphs

保持段落聚焦于一个主要观点
通常每段 2-4 个句子
用逻辑流连接段落
避免单句段落

### Lists | 列表

When using bullet points:

使用 bullet points 时：

- Make each item concise but complete
- Maintain parallel structure
- Use **bold** for item labels when appropriate
- Add brief descriptions after the label

每个条目简洁但完整
保持平行结构
适当时使用**粗体**标注条目标签
在标签后添加简短描述

---

## Tone and Voice | 语气和风格

### Professional but Accessible | 专业但易懂

Strike a balance between technical accuracy and readability:

在技术准确性和可读性之间取得平衡：

- Use precise technical terms where needed
- Explain concepts in plain language
- Avoid unnecessary jargon
- Assume moderate technical knowledge

需要时使用精确的技术术语
用通俗语言解释概念
避免不必要的行话
假设读者具有中等技术知识

### Active Voice | 主动语态

Prefer active voice over passive voice:

优先使用主动语态：

- ✅ "TradingFlow sends a signal to the vault"
- ❌ "A signal is sent to the vault by TradingFlow"

- ✅ "用户可以在 Windmill 页面查看余额"
- ❌ "余额可以被用户在 Windmill 页面查看"

---

## Language-Specific Notes | 语言特定注意事项

### English

- Use present tense for describing functionality
- Be direct and concise
- Use "you" to address the reader
- Avoid overly complex sentence structures

### 中文

- 使用简洁明了的表达
- 避免过于书面化的用语
- 保持专业性的同时注重可读性
- 适当使用标点符号断句

---

## References | 参考资料

Good examples of technical writing style:

技术写作风格的良好参考：

- [Stripe API Documentation](https://stripe.com/docs)
- [Ethereum Documentation](https://ethereum.org/en/developers/docs/)
- [Rust Book](https://doc.rust-lang.org/book/)
- CyberDAO Governance Documentation (see examples in original request)
- Sanctum Infinity Documentation (see examples in original request)

---

## Review Checklist | 审核检查清单

Before publishing documentation, verify:

发布文档前请验证：

- [ ] Paragraph content comprises 50-70% of the document
- [ ] Bullet points are used strategically, not excessively
- [ ] Each section flows naturally into the next
- [ ] Technical concepts are explained, not just listed
- [ ] Examples provide context and understanding
- [ ] Language is professional but accessible
- [ ] Both English and Chinese versions follow the same structure

---

**Last Updated**: 2025-10-06

This style guide should be referenced when creating or updating any TradingFlow documentation.

本风格指南应在创建或更新任何 TradingFlow 文档时参考。
