---
name: ai-note-image-rainbow-fun
version: 1.0.0
description: >
  生成「多色手绘知识点笔记图」的规范化流程。当用户希望把某个知识点、概念、公式、算法、总结或学习卡片做成色彩丰富、层次鲜明、带手绘框图与公式框的笔记图片时，使用本 skill。它适合偏好多彩手账感、希望多张图片维持统一视觉系统的场景。
---

# AI Note Image Rainbow Fun

把知识点生成成「多色手写学习笔记」风格图片，并保证单张或多张成组输出时具有统一的视觉风格。

## 适用场景

当用户出现以下需求时使用本 skill：

- 想把某个概念、公式、算法做成手写知识点图
- 想生成彩色学习卡片或手账风知识页
- 想一次生成多张，但希望它们保持同一套视觉规范
- 想要更活泼、更有层次的手绘笔记风，而不是极简干净风格

如果用户更偏好干净、克制、极简的手写笔记风格，应改用更简洁的对应 skill。

## 核心视觉规范

每张图都应套用同一套骨架：

- 纸张：纯白 A4 纸，竖版，轻微纸张纹理
- 正文：以蓝黑墨水为主
- 重点：用红笔标重点
- 分区与装饰：用绿、橙、紫做标题、框线、分区和图示上色
- 页眉：左上角写带编号的手写标题，如 `#1 Self-Attention 自注意力`；右上角写日期，格式 `YYYY.MM.DD`
- 公式：用手写方式呈现，关键公式用手绘框突出，并与正文形成视觉分层
- 图示：采用彩色手绘箭头、框图、曲线或小草图解释概念
- 语言：标题在适合时优先中英双语；公式保持标准数学表达
- 整体气质：rich、colorful、authentic notebook feel

## 生成流程

### 步骤 1：确定内容清单

对每一张图，先明确：

- 页面标题和编号
- 需要呈现的核心知识点
- 必须出现的公式
- 需要配套说明的图示内容

### 步骤 2：编写图像生成 prompt

使用下面的英文 prompt 模板，把占位符替换成具体内容：

```text
A handwritten study-notes page on PLAIN WHITE A4 paper, PORTRAIT orientation,
light paper texture. Natural COLORFUL journal style using MULTIPLE PEN COLORS -
blue and black for body text, RED for key highlights, plus GREEN, ORANGE and
PURPLE for section headers, box outlines and diagram coloring - rich but
harmonious colors. Title at top '{{#N Title 中文名}}', date in top-right corner
'{{YYYY.MM.DD}}'. Hand-drawn boxed key formula(s): {{formulas}}.
A colorful hand-drawn diagram: {{diagram description}}.
Clean neat academic handwriting, authentic colorful notebook feel, high detail.
```

然后调用当前环境中可用的图片生成能力来产出图片。

- 单批次最多建议生成 4 张
- 超过 4 张时，拆成多批生成
- 多张成组时，编号、日期位置、配色体系必须保持一致

### 步骤 3：交付结果

将生成结果按编号或标题顺序展示给用户。

如果用户还需要配套文字说明，应同时输出：

- 准确的公式文本
- 对应知识点的简短总结
- 图示的解释说明

## 注意事项

- 公式准确性优先。图片主要负责观感，但凡涉及精确表达，文字里也要补充标准公式。
- 多张成组输出时，要统一标题编号格式、日期位置和色彩体系。
- 目标是“色彩丰富但协调”，不要堆太多颜色导致版面脏乱。
- 图示应服务于解释概念，不要只做装饰。
