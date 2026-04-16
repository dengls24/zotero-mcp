# Annota 智能批注工作流开发指引 (For Claude Code)

## 1. 项目概述
本项目旨在开发一个基于 MCP (Model Context Protocol) 的 Zotero 本地服务。目标是允许大语言模型（如 Claude）通过自然语言对话，直接读取本地 Zotero 库中的 PDF 文件，理解语义，并**精准回写带有物理坐标的高亮批注和笔记**。

## 2. 系统架构
- **大脑/控制器**：Claude Code (通过 MCP 协议发送 Tool Calls)
- **桥梁服务**：Annota (建议使用 Python 或 Node.js 开发)
- **底层数据源**：本地 Zotero 7 客户端 (PDF 文件存储 + SQLite 数据库 / Local API)

## 3. 核心技术挑战：语义到坐标的映射
LLM 仅理解文本语义，而 Zotero 的 PDF 批注依赖于具体的页面几何坐标。因此，系统必须具备**文本解析与坐标提取能力**。工作流必须是：`LLM 意图 -> 检索带坐标的文本 -> 匹配目标段落 -> 提取该段落坐标 -> 写入 Zotero 批注`。

---

## 4. MCP Tools (Skill) 接口定义规范

在 MCP Server 中，需要实现以下三个核心 Tool：

### Tool 1: `get_pdf_layout_text` (带坐标的文本提取)
- **功能描述**：解析指定 PDF 的某一页，返回包含文本内容及其在页面上物理坐标的结构化数据。
- **推荐技术栈**：Python 环境下强烈建议使用 `PyMuPDF` (`fitz`)，Node.js 下可使用 `pdf.js`。
- **输入参数 (Input Schema)**：
  - `item_id` (string): Zotero 条目 ID 或 PDF 文件绝对路径。
  - `page_number` (integer): 需要解析的页码（从 0 开始）。
- **返回结果 (Output Schema)**：
  - 返回一个 JSON 数组，每个元素包含 `text` (文本块内容) 和 `rect` (坐标 `[x0, y0, x1, y1]`)。

### Tool 2: `create_pdf_annotation` (写入 PDF 批注/高亮)
- **功能描述**：在 Zotero 中为指定 PDF 的指定位置创建高亮批注。
- **技术路径**：通过 Zotero Local API 或直接操作 Zotero 的本地 SQLite 数据库（注意并发锁和数据备份）。
- **输入参数 (Input Schema)**：
  - `item_id` (string): 目标 PDF 的 Zotero ID。
  - `page_index` (integer): 页码索引。
  - `rects` (array): 坐标数组 `[[x0, y0, x1, y1], ...]`。
  - `color` (string): 批注颜色，支持 hex 代码（如 `#00FF00` 绿色, `#0000FF` 蓝色, `#FFFF00` 黄色）。
  - `comment` (string, optional): 附加在批注上的文字评论。
  - `type` (string): 批注类型，默认为 `highlight`，可选 `ink` (矩形框)。

### Tool 3: `add_child_note` (创建独立笔记)
- **功能描述**：为某个文献条目创建一个富文本笔记（Child Note）。
- **输入参数 (Input Schema)**：
  - `parent_item_id` (string): 父文献 ID。
  - `note_content` (string): Markdown 或 HTML 格式的笔记内容。

---

## 5. 具体业务场景工作流 (Claude Code 执行逻辑)

当用户输入自然语言指令时，Claude Code 应按照以下预设的业务逻辑调用上述 Tools：

### 场景 A：标出摘要中的发现结果
1. **指令**：“把这篇论文摘要中的发现结果标出来。”
2. **执行链**：
   - 调用 `get_pdf_layout_text` 获取第一页内容。
   - LLM 分析文本，识别出代表“发现/结果”的句子（如 "We found that...", "The results show..."）。
   - 提取这些句子的 `rect` 坐标。
   - 调用 `create_pdf_annotation`，传入坐标，设置 `color: "#28CA42"` (绿色)。

### 场景 B：标出每篇文章的假设框架
1. **指令**：“把文章的假设框架标出来。”
2. **执行链**：
   - 多次调用 `get_pdf_layout_text` 扫描前几页，搜索关键词（Hypothesis, Framework, Model 及其上下文）。
   - LLM 锁定阐述假设的核心段落，提取坐标。
   - 调用 `create_pdf_annotation`，传入坐标，设置 `color: "#2EA8E5"` (蓝色)，并设置 `comment: "本文核心假设/理论框架"`。

### 场景 C：画出公式并解释含义
1. **指令**：“把公式的含义解释写出来并画出来。”
2. **执行链**：
   - 调用 `get_pdf_layout_text` 扫描全文，定位数学公式区域（可通过特定的排版特征或标识符识别）。
   - 提取公式所在区域的边界坐标（Bounding Box）。
   - 调用 `create_pdf_annotation`，传入坐标，设置 `type: "ink"` (画框) 或背景高亮，`color: "#FFD400"` (黄色)。
   - LLM 基于上下文理解该公式中各个变量的含义。
   - 调用 `add_child_note`，将详细的公式变量解释以 Markdown 格式生成为该条目的子笔记。

---

## 6. 开发注意事项与要求
1. **坐标系转换**：`PyMuPDF` 等解析库获取的坐标可能与 Zotero 内部渲染的坐标系比例不同（如 72 DPI 与真实物理尺寸），在实现 `create_pdf_annotation` 时必须进行坐标系缩放校准。
2. **数据安全**：如果选择直接操作 SQLite，必须在执行 `UPDATE/INSERT` 前备份数据库，或确保 Zotero 处于非独占锁定状态。推荐优先探索 Zotero 7 提供的新版 API 接口。
3. **日志输出**：MCP Server 在执行坐标提取和写入时，需在 console 输出清晰的日志，以便调试坐标映射的准确性。