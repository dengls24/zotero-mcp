---
name: annota-annotate
description: >
  论文智能标注。使用两阶段工作流：先提取全文理解内容，再对目标句子精确标注。
  支持按语义分类标注（结果=绿色、方法=蓝色、不足=红色、贡献=紫色）。
  无具体指令时自动执行四色全覆盖标注（每类至少3-5处）。
  触发词：annotate、highlight、标注、高亮、mark this paper、标出论文中的
when_to_use: >
  用户要求标注论文中的特定内容（发现、方法、贡献、不足等），
  或要求对 PDF 做高亮标注，或给出 PDF 路径/论文标题要求标注。
argument-hint: "[PDF路径或论文标题] [标注内容描述(可选)]"
user-invocable: true
disable-model-invocation: false
---

# 论文智能标注

你将对一篇学术论文 PDF 执行智能标注。严格遵循两阶段工作流。

## 输入解析

用户输入：`$ARGUMENTS`

从输入中提取：
1. **目标 PDF**：文件路径、论文标题、或 item key
2. **标注内容**（可选）：要标注什么（如"发现结果"、"方法"、"limitation"等）
3. **标注颜色**（可选）：如果用户指定了颜色则使用，否则按语义自动选择

**如果用户没有指定标注内容**，默认执行"四色全覆盖标注"（见下方默认模式）。

## 颜色语义映射

| 内容类型 | 颜色 | 代码 |
|----------|------|------|
| 核心贡献、创新点、novelty | 紫色 | #a28ae5 |
| 方法、算法、模型、架构设计 | 蓝色 | #2EA8E5 |
| 实验结果、发现、数据、性能 | 绿色 | #28CA42 |
| 问题、不足、limitation、future work | 红色 | #ff6666 |
| 通用/用户未指定单一类型 | 黄色 | #ffd400 |

---

## Phase 1 — 理解论文内容

### Step 1.1：定位 PDF

如果用户提供了文件路径，直接使用。否则：
- 调用 `search_zotero_items(query)` 搜索论文
- 获取 `pdf_attachment_id`

### Step 1.2：提取全文

调用 `get_pdf_text_bulk(item_id, skip_refs=True)` 批量提取纯文本。

### Step 1.3：分析内容

**如果用户指定了标注目标**，只识别该类内容。

**如果用户没有指定（默认四色全覆盖模式）**，识别以下四类，每类至少找 3-5 处最有代表性的句子：

| 类型 | 标准 | 最少数量 |
|------|------|---------|
| 贡献/创新点 | 论文声称的 novelty、"we propose"、"our key insight" | 3-5 处 |
| 方法/架构 | 核心技术设计、关键机制、算法描述 | 3-5 处 |
| 实验结果 | 具体数字、性能对比、improvement 数据 | 4-6 处 |
| 局限/不足 | limitation、weakness、future work | 2-3 处 |

输出标注清单（在对话中展示）：
```
📋 将创建以下标注（共 N 条）：

[紫·贡献] Page 1: "We propose DCC, a novel..."
[紫·贡献] Page 2: "Our key insight is..."
[蓝·方法]  Page 3: "The architecture consists of..."
[蓝·方法]  Page 4: "We design a two-stage pipeline..."
[绿·结果]  Page 6: "achieves 2.3× speedup over baseline"
[绿·结果]  Page 6: "reduces energy by 41%"
[红·不足]  Page 8: "Our approach is limited to..."
...

⚠️ 写入需要关闭 Zotero。请确认已关闭后回复"确认"，或直接说"开始"。
```

### Step 1.4：检查已有标注

调用 `list_annotations(item_id)` 查看是否已有标注，避免重复。
如果已有大量标注，告知用户并询问是否继续（追加）。

---

## Phase 2 — 精确标注

### Step 2.1：获取目标页坐标

只对包含目标句子的页面调用 `get_pdf_layout_text(item_id, page_number)`。

**绝对不要**对每一页都调用这个工具——只调用目标页。
多个目标句子在同一页时，只调用一次该页。

### Step 2.2：匹配坐标

在返回的 blocks 中找到目标句子对应的 rect 坐标。
一个句子可能跨多行，需要收集所有匹配行的 rect。
匹配时用子串匹配（句子的前 30 个字符即可定位），不要精确匹配。

### Step 2.3：写入标注

用户确认后，调用 `batch_annotate` 一次性写入所有标注：

```json
{
  "item_id": "...",
  "annotations": [
    {"page_index": 0, "rects": [...], "color": "#a28ae5", "text": "贡献描述..."},
    {"page_index": 2, "rects": [...], "color": "#2EA8E5", "text": "方法描述..."},
    {"page_index": 5, "rects": [...], "color": "#28CA42", "text": "结果数据..."},
    {"page_index": 7, "rects": [...], "color": "#ff6666", "text": "局限说明..."}
  ]
}
```

### Step 2.4：完成汇报

```
✅ 标注完成！共创建 {N} 条标注：
  🟣 贡献/创新点：{N1} 条
  🔵 方法/架构：{N2} 条
  🟢 实验结果：{N3} 条
  🔴 局限/不足：{N4} 条

请在 Zotero 中按 Ctrl+Shift+R 刷新查看。

还需要什么？
- 生成结构化阅读笔记 → /annota-summarize
- 写审稿意见 → /annota-review
- 针对某个具体内容追加标注 → 直接告诉我
```

---

## 错误处理

- `database_locked`：提醒用户关闭 Zotero 后重试
- 找不到目标句子坐标：报告哪些句子未能匹配，给出可能原因（OCR 误差、特殊字符、扫描版 PDF 等）
- 论文是扫描版（无文本层）：告知用户无法提取坐标，建议使用 OCR 工具预处理
