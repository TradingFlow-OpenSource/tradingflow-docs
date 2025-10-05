# Multi-Language Setup Guide | 多语言设置指南

## 🎯 Current Setup (Multiple Spaces - Recommended) | 当前方案（多空间 - 推荐）

This documentation uses **multiple independent GitBook Spaces** - one for each language.

本文档使用**多个独立的 GitBook Space** - 每种语言一个。

### How It Works | 工作原理

Each language has its own dedicated GitBook Space:
- 🇬🇧 **English Space**: `tradingflow-en` 
- 🇨🇳 **Chinese Space**: `tradingflow-zh`

每种语言都有自己专用的 GitBook Space：
- 🇬🇧 **英语空间**: `tradingflow-en`
- 🇨🇳 **中文空间**: `tradingflow-zh`

### Advantages | 优势

✅ **Completely Free** | **完全免费** - No paid plan required
✅ **Clean Navigation** | **清晰导航** - Each space has its own clean sidebar
✅ **Better SEO** | **更好的SEO** - Separate URLs for each language
✅ **Independent Updates** | **独立更新** - Update languages separately
✅ **Professional Look** | **专业外观** - No language mixing in sidebar

### File Structure | 文件结构

```
7_docs/
├── README.md                   # Language selection page (主选择页)
├── SUMMARY.md                  # Points to language spaces (指向语言空间)
├── .gitbook.yaml               # GitBook root configuration
├── MULTILANG_SETUP.md          # This guide (本指南)
├── en/                         # English Space content
│   ├── README.md               # English home with cross-links
│   ├── SUMMARY.md              # English sidebar navigation
│   ├── getting-started/
│   ├── core-concepts/
│   ├── node-details/
│   ├── engineering-docs/
│   └── resources/
└── zh/                         # Chinese Space content
    ├── README.md               # Chinese home with cross-links
    ├── SUMMARY.md              # Chinese sidebar navigation
    ├── getting-started/
    ├── core-concepts/
    ├── node-details/
    ├── engineering-docs/
    └── resources/
```

---

## 📋 Setup Instructions | 设置步骤

### Step 1: Create GitBook Spaces | 步骤1：创建 GitBook 空间

1. Go to your GitBook organization
2. Click "New Space" button
3. Create **two separate spaces**:
   - `tradingflow-en` (English)
   - `tradingflow-zh` (Chinese)

前往你的 GitBook 组织并创建两个独立空间。

### Step 2: Connect to GitHub | 步骤2：连接到 GitHub

For each space:

1. **English Space** (`tradingflow-en`):
   - Go to Space Settings → Integrations → GitHub
   - Connect to your repository
   - Set **root path**: `/7_docs/en`
   - Set **primary branch**: `main`

2. **Chinese Space** (`tradingflow-zh`):
   - Go to Space Settings → Integrations → GitHub
   - Connect to your repository
   - Set **root path**: `/7_docs/zh`
   - Set **primary branch**: `main`

为每个空间连接 GitHub 仓库并设置对应的根路径。

### Step 3: Configure GitBook Files | 步骤3：配置 GitBook 文件

Each language folder has its own:
- `README.md` - Home page with cross-language links
- `SUMMARY.md` - Sidebar navigation

每个语言文件夹都有自己的首页和导航配置。

### Step 4: Update Cross-Language URLs | 步骤4：更新跨语言链接

After creating the spaces, update the URLs in:

**English README** (`en/README.md`):
```markdown
## 🌍 Other Languages
- **[中文文档](https://your-org.gitbook.io/tradingflow-zh)** - Chinese documentation
```

**Chinese README** (`zh/README.md`):
```markdown
## 🌍 其他语言
- **[English Documentation](https://your-org.gitbook.io/tradingflow-en)** - 英文文档
```

Replace `your-org` with your actual GitBook organization name.

用你实际的 GitBook 组织名称替换 `your-org`。

### Step 5: Test and Verify | 步骤5：测试验证

1. Commit and push to GitHub
2. Wait for GitBook to sync (usually < 1 minute)
3. Visit both spaces:
   - `https://your-org.gitbook.io/tradingflow-en`
   - `https://your-org.gitbook.io/tradingflow-zh`
4. Test cross-language navigation links

提交到 GitHub 后等待 GitBook 同步，然后测试所有链接。

---

## 🎨 URL Structure | URL 结构

### English Space
- Home: `https://your-org.gitbook.io/tradingflow-en`
- Getting Started: `https://your-org.gitbook.io/tradingflow-en/getting-started/what-is-tradingflow`
- Core Concepts: `https://your-org.gitbook.io/tradingflow-en/core-concepts/on-chain-vaults`

### Chinese Space
- Home: `https://your-org.gitbook.io/tradingflow-zh`
- 快速开始: `https://your-org.gitbook.io/tradingflow-zh/getting-started/what-is-tradingflow`
- 核心概念: `https://your-org.gitbook.io/tradingflow-zh/core-concepts/on-chain-vaults`

---

## 🔄 Maintenance Workflow | 维护工作流

### Adding New Content | 添加新内容

1. **Create content in both language folders**:
   ```bash
   # Add to English
   touch en/new-section/new-article.md
   
   # Add to Chinese
   touch zh/new-section/new-article.md
   ```

2. **Update SUMMARY.md in both languages**:
   - `en/SUMMARY.md` - Add English navigation
   - `zh/SUMMARY.md` - Add Chinese navigation

3. **Commit and push**:
   ```bash
   git add en/ zh/
   git commit -m "docs: add new section to both languages"
   git push
   ```

### Updating Existing Content | 更新现有内容

- Each language can be updated independently
- Changes sync automatically to the respective GitBook Space
- No need to rebuild or redeploy manually

---

## ✅ Advantages of Multiple Spaces | 多空间方案优势

### vs. Single Space with Chapters | 对比单空间章节方案

| Feature | Multiple Spaces | Single Space |
|---------|----------------|---------------|
| **Navigation Clarity** | ⭐⭐⭐⭐⭐ Clean | ⭐⭐⭐ Mixed |
| **SEO** | ⭐⭐⭐⭐⭐ Separate URLs | ⭐⭐⭐ Shared URL |
| **Maintenance** | ⭐⭐⭐⭐ Independent | ⭐⭐⭐⭐⭐ Unified |
| **Professional Look** | ⭐⭐⭐⭐⭐ Very clean | ⭐⭐⭐ Acceptable |
| **Cost** | 🆓 Free | 🆓 Free |
| **Setup Complexity** | ⭐⭐⭐ Moderate | ⭐⭐⭐⭐⭐ Simple |

### vs. Paid Variants | 对比付费变体方案

| Feature | Multiple Spaces (Free) | Variants (Paid) |
|---------|----------------------|------------------|
| **Cost** | 🆓 **Free** | 💰 Business Plan Required |
| **Auto Language Detection** | ❌ Manual | ✅ Automatic |
| **Language Switcher UI** | Manual links | Built-in switcher |
| **SEO** | ✅ Good | ✅ Excellent |
| **Professional Look** | ✅ Very good | ✅ Perfect |
    └── resources/
```

---

## 💡 Tips and Best Practices | 技巧和最佳实践

### Cross-Language Consistency | 跨语言一致性

1. **Keep folder structure identical**:
   - `en/getting-started/` ↔ `zh/getting-started/`
   - Same file names for parallel content

2. **Use consistent navigation order**:
   - English SUMMARY.md and Chinese SUMMARY.md should have matching structures

3. **Update both languages together**:
   - When adding new features, update both EN and ZH
   - Prevents documentation drift

### Cross-Links Management | 跨链接管理

**Add language switcher to every README**:
- Makes it easy for users to switch languages
- Increases discoverability
- Provides better user experience

### Version Control | 版本控制

```bash
# Good commit message format
git commit -m "docs(en): add deployment guide"
git commit -m "docs(zh): 添加部署指南"
git commit -m "docs(both): update core concepts"
```

---

## 🆘 Troubleshooting | 故障排除

### Space Not Syncing | 空间未同步

1. Check GitHub integration in GitBook settings
2. Verify the root path is correct (`/7_docs/en` or `/7_docs/zh`)
3. Make sure files are committed to the correct branch
4. Trigger manual sync in GitBook UI

### Cross-Links Not Working | 跨链接失效

1. Verify both spaces are created and published
2. Update URLs with actual GitBook space names
3. Test links in incognito mode to bypass cache

### Content Not Updating | 内容未更新

1. Wait 1-2 minutes for GitBook to sync
2. Hard refresh browser (Ctrl+Shift+R or Cmd+Shift+R)
3. Check git commit reached the correct branch

---

## 🔮 Future: Upgrade to Paid Variants | 未来：升级到付费变体

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
