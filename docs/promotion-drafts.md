# Annota 推广文案草稿

---

## 1. awesome-mcp-servers PR（直接提交）

**分类**: Knowledge & Memory  
**条目格式（按字母排序插入）**:

```
- [Annota](https://github.com/dengls24/annota) - AI-powered Zotero PDF annotation server. Reads papers, highlights key findings with semantic color coding, explains formulas, and writes structured reading notes — all saved back to Zotero. Features two-phase workflow for large PDFs (63–80% context savings) and batch annotations.
```

---

## 2. Zotero Forums 帖子（英文）

**板块**: Plugins & Integrations  
**标题**: Annota — Use AI to Read, Annotate, and Summarize Your Papers Automatically

**正文**:

Hi everyone,

I built an open-source tool that connects Zotero with AI (via Claude Code / MCP) to automate paper reading workflows. It's not a traditional Zotero plugin — it's an MCP server that lets AI directly read your PDFs and create annotations in your Zotero library.

**What it can do:**

- **Smart highlighting**: Tell AI "highlight the findings in the abstract in green" and it does exactly that — reading the text, identifying findings, and creating precise highlights
- **Formula explanations**: AI reads formulas and adds plain-language annotations
- **Structured notes**: Auto-generate reading summaries (contributions, methods, results, limitations) saved as Zotero child notes
- **Simulated peer review**: Get structured feedback as if from a conference reviewer (MICRO/ISCA/ASPLOS etc.)

**Key design choices:**
- Two-phase workflow for large PDFs: first extracts text cheaply, then only gets coordinates for target pages. Reduces context usage by 63–80%.
- Auto-skips References section
- Batch annotations (10 highlights in 1 API call)
- Semantic color coding: green=results, blue=methods, red=limitations, purple=contributions

**Requirements**: Python 3.10+, PyMuPDF, Claude Code (or any MCP-compatible client), Zotero 7

**Limitation**: Write operations currently require Zotero to be closed (SQLite exclusive lock). A plugin bridge is on the roadmap.

GitHub: https://github.com/dengls24/annota  
License: MIT

Would love to hear your feedback and feature suggestions!

---

## 3. 知乎文章

**标题**: 用 AI 自动标注论文？我开源了一个 Zotero + Claude Code 的智能阅读助手

**正文**:

作为一个每周要读大量论文的研究生，我一直在想：能不能让 AI 帮我边读论文边做标注？

于是我做了 **Annota** —— 一个连接 Zotero 和 AI 的开源工具。你只需要用自然语言告诉 AI 你想标注什么，它就能精准地在 PDF 上高亮、写笔记。

### 它能做什么？

| 你说… | AI 做… |
|-------|--------|
| "高亮摘要中的发现结果" | 阅读摘要，识别发现，用绿色标出 |
| "解释第3页的公式" | 提取公式，添加中文解释批注 |
| "写一份结构化阅读笔记" | 生成包含贡献、方法、结果、局限性的笔记，保存到 Zotero |
| "以 MICRO 审稿人视角审阅" | 输出带分维度评分的结构化审稿意见 |

### 效果展示

[插入 assets/note-2.png — 摘要绿色高亮]
*AI 自动识别摘要中的研究发现并用绿色标注*

[插入 assets/note-7.png — 结构化笔记]
*AI 生成的结构化阅读笔记，包含研究问题、方法、发现和启示*

[插入 assets/note-5.png — 公式解释]
*AI 解释 DID 模型公式，添加中文变量说明*

### 技术亮点

**两阶段工作流**：处理长论文时，先提取纯文本理解全文（轻量），再只对目标页获取坐标做标注（精准）。实测节省 63–80% 的上下文开销。

**自动跳过参考文献**：一篇 21 页论文只需提取 13 页正文。

**语义化颜色**：绿色=结果、蓝色=方法、红色=局限、紫色=贡献，一眼看出论文结构。

### 三分钟安装

```bash
git clone https://github.com/dengls24/annota.git
cd annota
pip install pymupdf mcp
```

然后在 Claude Code 中配置 MCP Server 即可使用。详见 GitHub README。

### 开源地址

GitHub: https://github.com/dengls24/annota  
协议: MIT  

如果对你的科研有帮助，欢迎 Star ⭐ 和提 Issue！

---

## 4. 小红书图文

**标题**: 研究生必备｜让AI帮你自动读论文+标注重点

**正文**:

姐妹们！！发现一个读论文神器 🔥

我开源了一个工具，可以让 AI 自动帮你：
✅ 高亮论文中的关键发现（绿色）
✅ 标注研究方法（蓝色）
✅ 解释复杂公式
✅ 写结构化阅读笔记
✅ 模拟审稿人给你提修改意见

而且标注直接保存到 Zotero 里！不用手动复制粘贴！

💡 使用超简单：
安装好之后，用自然语言告诉 AI：
"高亮这篇论文摘要中的发现结果"
"解释第3页的公式"
"写一份结构化阅读笔记"
AI 就自动帮你完成了！

📦 安装只需 3 分钟：
搜索 GitHub: dengls24/annota
按 README 步骤操作就行

需要: Zotero 7 + Python + Claude Code
完全免费开源 MIT 协议

#研究生 #论文阅读 #Zotero #AI工具 #科研神器 #学术

**配图建议**: 4张图
1. assets/note-2.png（摘要高亮效果）
2. assets/note-7.png（结构化笔记）
3. assets/note-5.png（公式解释）
4. assets/note-8.png（AI工作流截图）

---

## 5. Twitter/X（英文）

**推文 1（主推）**:

I built an open-source MCP server that lets AI read your Zotero papers and create precise annotations automatically.

🟢 Highlight findings
🔵 Mark methods
📝 Generate structured reading notes
🔍 Simulate peer review

Two-phase workflow saves 63-80% context on large PDFs.

GitHub: https://github.com/dengls24/annota

[附图: assets/note-7.png]

**推文 2（thread）**:

How it works:

Phase 1: AI reads full text (cheap, no coordinates)
→ Understands the paper, identifies what to annotate

Phase 2: Gets precise coordinates for target pages only
→ Creates highlights exactly where they belong

Result: 21-page paper → only 13 pages extracted (auto-skips references)

**推文 3（thread）**:

3 built-in Claude Code skills:

/annota-annotate — smart highlighting with semantic colors
/annota-summarize — structured reading notes saved to Zotero
/annota-review — simulated peer review with scoring

All MIT licensed. PRs welcome!

**Tag**: @AnthropicAI #MCP #Zotero #AcademicTwitter #OpenSource #AI

---

## 6. Reddit

### r/Zotero 帖子

**标题**: [Tool] Annota — AI reads your papers and creates annotations automatically (open-source)

**正文**:

I've been working on an MCP server that connects Zotero with AI to automate paper reading. It's open-source (MIT) and works with Claude Code or any MCP-compatible client.

**What it does:**
- You say "highlight the findings in the abstract in green" → AI reads the abstract, identifies findings, creates precise green highlights
- You say "explain the formulas" → AI adds plain-language annotations to formulas
- You say "write a reading summary" → AI generates structured notes (contributions, methods, results, limitations) and saves them as Zotero child notes

**Technical highlights:**
- Two-phase workflow: text first (cheap), coordinates only for target pages → 63-80% context savings
- Auto-skips References section
- Batch annotations (10 highlights in 1 call)
- Semantic colors: green=results, blue=methods, red=limitations, purple=contributions

**Known limitation:** Write operations require Zotero to be closed (SQLite exclusive lock). Plugin bridge is on the roadmap.

**Requirements:** Python 3.10+, Zotero 7, Claude Code

GitHub: https://github.com/dengls24/annota

Happy to hear feedback and suggestions!

### r/ClaudeAI 帖子

**标题**: Built an MCP server that turns Zotero into an AI-powered paper reading assistant

**正文**: (同上，增加对 MCP/Claude Code 生态的讨论)

---

## 7. MCP Registry 提交

访问 https://registry.modelcontextprotocol.io/ 查看提交流程，或直接去 https://github.com/modelcontextprotocol/registry 按 docs 指引提交。

**Server 信息**:
- Name: annota
- Description: AI-powered Zotero PDF annotation server. Reads papers, highlights findings, explains formulas, and writes structured reading notes.
- Repository: https://github.com/dengls24/annota
- Author: Lishuo Deng
- License: MIT
- Categories: Knowledge & Memory, Research, PDF

---

## 执行清单

| 渠道 | 状态 | 操作 |
|------|------|------|
| GitHub Topics | ✅ 已完成 | 12 个标签已添加 |
| GitHub Discussions | ✅ 已完成 | 已开启 |
| GitHub Release v1.0.0 | ✅ 已完成 | 已创建 |
| awesome-mcp-servers PR | ⬜ 待提交 | Fork → 添加条目 → PR |
| MCP Registry | ⬜ 待提交 | 按官方流程提交 |
| Zotero Forums | ⬜ 待发帖 | 复制第2节内容发帖 |
| 知乎 | ⬜ 待发布 | 复制第3节 + 插入截图 |
| 小红书 | ⬜ 待发布 | 复制第4节 + 4张配图 |
| Twitter/X | ⬜ 待发布 | 复制第5节发 thread |
| Reddit r/Zotero | ⬜ 待发帖 | 复制第6节内容 |
| Reddit r/ClaudeAI | ⬜ 待发帖 | 复制第6节内容 |
