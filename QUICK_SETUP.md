# Quick Setup Guide | 快速设置指南

## 🚀 5-Minute Setup | 5分钟设置

### 1️⃣ Create Two GitBook Spaces | 创建两个空间

```
Space 1: tradingflow-en (English)
Space 2: tradingflow-zh (Chinese)
```

### 2️⃣ Connect to GitHub | 连接 GitHub

**English Space:**
- Root Path: `/7_docs/en`
- Branch: `main`

**Chinese Space:**
- Root Path: `/7_docs/zh`
- Branch: `main`

### 3️⃣ Update URLs in README files | 更新 README 链接

Replace `your-org` with your GitBook organization name:

**`en/README.md`:**
```markdown
- **[中文文档](https://your-org.gitbook.io/tradingflow-zh)**
```

**`zh/README.md`:**
```markdown
- **[English Documentation](https://your-org.gitbook.io/tradingflow-en)**
```

### 4️⃣ Commit and Push | 提交推送

```bash
git add .
git commit -m "docs: setup multi-space documentation"
git push
```

### 5️⃣ Done! | 完成！ ✅

Visit your spaces:
- 🇬🇧 `https://your-org.gitbook.io/tradingflow-en`
- 🇨🇳 `https://your-org.gitbook.io/tradingflow-zh`

---

## 📝 File Structure | 文件结构

```
7_docs/
├── en/           → English GitBook Space
│   ├── README.md
│   └── SUMMARY.md
└── zh/           → Chinese GitBook Space
    ├── README.md
    └── SUMMARY.md
```

---

## 🔗 Important Files | 重要文件

| File | Purpose |
|------|---------|
| `en/README.md` | English home page |
| `en/SUMMARY.md` | English sidebar |
| `zh/README.md` | Chinese home page |
| `zh/SUMMARY.md` | Chinese sidebar |
| `MULTILANG_SETUP.md` | Detailed guide |

---

## 💡 Need Help? | 需要帮助？

See the full guide: [MULTILANG_SETUP.md](MULTILANG_SETUP.md)
