# Multi-Language Setup Guide | 多语言设置指南

## 🆓 Current Setup (Free Solution) | 当前方案（免费）

This documentation uses **GitBook's free single-space multi-language approach**.

本文档使用 **GitBook 的免费单空间多语言方案**。

### How It Works | 工作原理

All language versions are in one GitBook Space, organized by chapters in the sidebar:
- 🇬🇧 **English** section
- 🇨🇳 **中文** section

所有语言版本都在一个 GitBook Space 中，通过侧边栏章节组织：
- 🇬🇧 **English** 部分
- 🇨🇳 **中文** 部分

### Advantages | 优势

✅ **Completely Free** | **完全免费** - No paid plan required
✅ **Single Maintenance** | **单一维护** - All content in one place
✅ **Easy Navigation** | **便捷导航** - Switch languages via sidebar
✅ **Simple Structure** | **简单结构** - No complex configuration

### File Structure | 文件结构

```
7_docs/
├── README.md              # Home page (bilingual intro)
├── SUMMARY.md             # Table of contents (all languages)
├── .gitbook.yaml          # GitBook configuration
├── en/                    # English documentation
│   ├── getting-started/
│   ├── core-concepts/
│   ├── node-details/
│   ├── engineering-docs/
│   └── resources/
└── zh/                    # Chinese documentation
    ├── getting-started/
    ├── core-concepts/
    ├── node-details/
    ├── engineering-docs/
    └── resources/
```

---

## 💎 Alternative: Paid Variants (Optional) | 备选方案：付费变体（可选）

If you upgrade to GitBook **Business** or **Enterprise** plan, you can use native language variants.

如果升级到 GitBook **Business** 或 **Enterprise** 计划，可以使用原生语言变体功能。

### Setup for Variants | 变体设置

1. Create `.gitbook/variants.yaml`:

```yaml
variants:
  - name: en
    title: "TradingFlow Documentation"
    label: "English"
    file: ./en/README.md
    summary: ./en/SUMMARY.md
    locales: ['en-US', 'en-GB']

  - name: zh
    title: "TradingFlow 文档"
    label: "中文"
    file: ./zh/README.md
    summary: ./zh/SUMMARY.md
    locales: ['zh-CN', 'zh-TW']
```

2. Update `SUMMARY.md` to remove language sections
3. GitBook will show a language switcher in the UI

### Variants Advantages | 变体优势

- 🎯 Better SEO with separate URLs per language
- 🌐 Automatic language detection based on browser
- 🎨 Cleaner UI with language switcher

---

## 🔄 Alternative Free Solutions | 其他免费方案

### Option 1: Multiple Free Spaces | 选项1：多个免费空间

Create separate GitBook spaces:
- `tradingflow-docs-en` (English)
- `tradingflow-docs-zh` (中文)

Add cross-links in each space's README.

### Option 2: GitHub Pages Landing | 选项2：GitHub Pages 落地页

Create a simple HTML landing page to choose languages:

```html
<!DOCTYPE html>
<html>
<head>
    <title>TradingFlow Docs - Choose Language</title>
</head>
<body>
    <h1>🌍 Choose Your Language</h1>
    <ul>
        <li><a href="https://tradingflow.gitbook.io/docs-en">🇬🇧 English</a></li>
        <li><a href="https://tradingflow.gitbook.io/docs-zh">🇨🇳 中文</a></li>
    </ul>
</body>
</html>
```

Host on GitHub Pages and use as the main entry point.

---

## 📝 Maintenance Tips | 维护建议

### Content Synchronization | 内容同步

When updating documentation:
1. Update the original language (e.g., English)
2. Translate and update the other language (e.g., Chinese)
3. Keep file structure identical across languages
4. Use consistent naming conventions

### Link Management | 链接管理

- Use relative paths: `../core-concepts/vaults.md`
- Keep internal links within the same language
- Add cross-language links in README when needed

### Version Control | 版本控制

```bash
# Commit both languages together
git add en/ zh/ SUMMARY.md README.md
git commit -m "docs: update multi-language documentation"
git push
```

---

## 🆘 Need Help? | 需要帮助？

If you need to switch between different multi-language approaches, refer to this guide and choose the solution that fits your needs and budget.

如果需要在不同的多语言方案之间切换，请参考本指南并选择适合你需求和预算的解决方案。

**Current Recommendation: Free single-space approach** ✅

**当前推荐：免费单空间方案** ✅
