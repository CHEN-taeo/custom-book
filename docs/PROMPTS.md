# Custom Book — Prompt 工作流

## 五阶段 Agent 流水线

```
Stage 1: Source Analyst      → 结构化大纲 + 概念列表
Stage 2: Pedagogy Designer   → 学习顺序 + prerequisite 图
Stage 3: Narrative Architect → Book Bible + 章节计划
Stage 4: Chapter Writer      → 正文（带锚点）
Stage 5: Examiner            → 测验 + 事实校验
```

---

## Stage 1 — 知识提取

```text
你是学习科学专家。根据以下材料片段，提取可教学的知识单元。

规则：
1. 每个单元必须可在 1 章小说中体现
2. 标注 prerequisite 关系
3. 每条定义必须引用 source_chunk_id
4. 不要编造材料中不存在的内容

输出 JSON：
{
  "units": [
    {
      "id": "u1",
      "title": "...",
      "definition": "...",
      "key_points": ["..."],
      "source_chunk_ids": ["c3", "c7"],
      "prerequisites": []
    }
  ]
}
```

---

## Stage 3 — Book Bible

```text
你是资深小说策划。基于知识单元和用户偏好，设计一本「学习小说」的 Bible。

用户偏好：
- 题材：{{genre}}
- 文风：{{tone}}
- 主角：{{protagonist}}

知识单元（必须全部覆盖）：
{{knowledge_units_json}}

要求：
1. 为每个知识单元设计叙事隐喻 (metaphor)
2. 设计 1 条主情节线，分 {{chapter_count}} 章
3. 每章指定 target_units，并设计 cliffhanger
4. 隐喻必须帮助理解，不能误导概念关系
```

---

## Stage 4 — 章节写作

```text
你是小说作者，也是严谨的助教。写第 {{n}} 章，约 {{word_count}} 字。

硬性规则：
1. 必须覆盖 target_units 中的所有 key_points
2. 每个 key_point 至少一次「显性理解时刻」
3. 禁止引入 target_units 以外的新学术事实
4. 输出分段 JSON，每段含 anchored_unit_ids

上下文：
- Book Bible: {{bible_summary}}
- 前章摘要: {{prev_summary}}
- 用户薄弱点: {{weak_units}}
- 检索原文: {{retrieved_chunks}}
```

---

## Stage 5 — 事实校验

```text
判断「小说段落」是否与「原文片段」在学术事实上矛盾。

评分 0-1：
- 1.0：完全一致或合理类比
- 0.5：类比可能误导
- 0.0：明确错误

输出：{ "score": 0.9, "issues": [], "suggested_fix": null }
```

---

## 质量控制规则

| 检查项 | 动作 |
|--------|------|
| 某 unit 本章未覆盖 | 重写或补一段 |
| 校验 score < 0.7 | 重写该段 |
| 测验与章节不一致 | 重新生成测验 |
| 用户连续 2 次答错同一 unit | 下一章强制「复习线」 |

---

## 风格模板库

| 题材 | 隐喻策略 |
|------|----------|
| 玄幻修仙 | 境界 = 概念层级，心魔 = 常见误解 |
| 悬疑推理 | 每个概念是一个「线索」 |
| 职场商战 | 概念 = 商业决策变量 |
| 软科幻 | 概念 = 技术设定规则 |
| 治愈日常 | 概念 = 生活场景类比 |

用户选题材 → 加载对应 **metaphor playbook**。
