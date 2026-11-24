---
name: glossary-locked-aligned-translator
description: 把真实科研论文（或明确段落区间）译成目标语言，输出段落级双语对照、术语表锁定译名、关系保真核验与逐段译法依据，确保术语一致、段落不漏不重、指代/否定/限定/数字单位/公式/文献编号与原文一致。当用户要求"翻译这篇论文""术语要统一""长文分段翻译别漏段""给双语对照表""核对译文有没有译错数字/否定/引用""译不准的标出来"时使用。输出含术语锁表、段落对齐表、关系核验表、译文核验清单与自检结果。
version: 1.0.0
license: MIT
---

# 文献翻译（Glossary-Locked Aligned Translation）

> 面对真实科研论文，把"翻译"从"给出通顺的译文"变成**可逐段核验的双语工程**。核心不是译文多漂亮，而是让每一段都能回答："对应原文哪一段？用了哪个锁定的术语？指代、否定、限定、数字、公式、引用有没有走样？拿不准的地方标在哪？"

## 1. 用途与适用边界

**解决什么**：基础 Agent 翻译科研文献的典型失败——同一术语在标题、摘要、正文、图注、结论中译名不一；长文跨页跨栏漏段或重复段；把"not necessarily"译成"必然"、把限定条件丢掉；数字单位换算写错；公式符号与文献编号错位；译不准的地方含混带过。本 Skill 用"**先建锁表 → 逐段对齐 → 关系保真 → 译法留痕**"四段流水线消除这些失败，译不准的明确标注，不靠语感硬翻。

**触发场景**：全篇或指定段落区间的论文翻译；术语一致性强制统一；长文按语义分段续译（跨页/跨栏/跨章节）；段落级双语对照输出；译文核验（数字、单位、公式、图表编号、引用标记、否定与限定）；译不准部分的人工确认清单。

**不该用**：用户只要"这段大意"的概括性翻译而不要逐段对照与核验时（转用普通翻译）；原文是扫描件且无文字层、无法读取时（应报告材料问题，不得凭空翻译）；需要解读图表/公式科学含义时（本 Skill 只保证符号与编号保真，不做科学解读）。

**输入**：`/input/case.json`（含 `case_id`、任务说明、原文材料或 PDF 路径、源/目标语言、指定段落区间、可用术语表）与 `task.md`。原文以实际文件为准，不依据领域标签臆测内容。**输出**：`/app/output/result.json`（字段见第 4 节）+ 可编辑的 `bilingual_translation.md`。

## 2. 核心方法（GLAT）

本 Skill 的方法名为 **GLAT = Glossary-Locked Aligned Translation（术语锁定的双语对齐翻译）**，四个不可跳过的机制，一一对应题面"术语稳 / 段落全 / 关系准 / 可核验"：

1. **术语稳 —— 术语锁表（Glossary Lock）**：翻译前先从原文抽取领域术语，建立"原词 → 锁定译名 → 适用范围 → 出处"的术语表；全篇（标题、摘要、正文、图注、结论）强制回填同一译名。术语表可来自输入，也可由本 Skill 抽取并交人工确认；**锁表一旦确定，译文用词与之逐字对齐**，不一致即报错回退。
2. **段落全 —— 段落对齐（Paragraph Alignment）**：按语义边界切分长文，每段分配唯一 `para_id`，携带必要上下文与已锁定术语逐段翻译；跨页、跨栏、跨章节衔接用 `continuation_of` 链接；段号与原文一一对应，不漏段、不重复、不错位。
3. **关系准 —— 关系保真（Relation Fidelity）**：逐段核对七类关系——指代对象、否定与限定条件、因果与让步、数字与单位、公式符号、文献引用编号、图表编号——译前后必须一致；每类核验记录原文依据与译文证据，任一不符标 `relation_mismatch` 并回退重译。
4. **可核验 —— 译文核验清单（Verification Ledger）**：每段译文给出 `translation_basis`（译法依据：直译/意译+理由/术语锁表条目/存疑说明），并指回原文页码段落句子；译不准、原文残缺、多义未决的部分标 `uncertain` 并进 `human_review_items`，**不含混处理、不编造原文没有的内容**。

贯穿一条铁律：**翻译与核验分离**——先完成译文草稿，再以"校对者"身份独立执行第 5 节自检，两遍不可合并；每次修正写入 `change_log`。

## 3. 工作流（SOP）

严格按顺序执行，每步都有检查点；检查点不过则回退该步，不得带病推进。

**Step 0 · 输入解析与边界确认**
- 读 `case.json` 与 `task.md`，提取：源/目标语言、翻译范围（全篇或段落区间）、原文材料路径、是否提供术语表、明确禁止事项。
- 确认原文可读（有文字层）；扫描件无文字层则 `status=blocked` 说明，不得凭空翻译。
- 检查点：范围与语言明确？原文可读？缺口登记 `uncertainties`。

**Step 1 · 术语抽取与锁表（术语稳）**
- 从标题、摘要、正文、图注、结论抽取领域术语（含缩写、同义表述），建术语表：`term / translation / scope / source_location / status`。
- 用户提供术语表时以其为准；自抽术语标 `needs_confirmation=true` 进人工确认。
- 检查点：术语表是否覆盖关键概念？同一原词是否只有一个锁定译名？锁表确定后全篇不得改译。

**Step 2 · 语义分段与对齐框架（段落全）**
- 按语义边界切分原文，分配 `para_id`；记录每个 `para_id` 的原文位置（页码/栏/段号/句边界）。
- 跨页、跨栏、跨章节的续段用 `continuation_of` 链接；建立 `alignment_map` 骨架（段号一一对应）。
- 检查点：段落总数是否与原文字逐段清点一致？有无漏段、重复、错位？不确定处标 `alignment_uncertain`。

**Step 3 · 逐段翻译（携带上下文与锁表）**
- 每段翻译时携带：上一段末句上下文、已锁定术语表、该段涉及的公式/引用占位符。
- 公式、文献编号、图表编号按占位符原文保留，不译、不改号。
- 检查点：每段译文是否只译本段内容、未把上下文混进来？术语是否与锁表逐字一致？

**Step 4 · 关系保真与译法留痕（关系准 + 可核验）**
- 逐段走七类关系核验表；不符标 `relation_mismatch` 并回退 Step 3 重译。
- 为每段写 `translation_basis` 与原文回链（页码/段落/句子）；译不准处标 `uncertain` 进人工确认。
- 检查点：七类关系是否逐项核对并留痕？每段是否可回链原文并有译法依据？

**Step 5 · 独立核对（与翻译分离）**
- 以校对者身份逐项走 `references/checklist.md`；重点复核四类关键问题；发现错误回退对应步骤并写 `change_log`。
- 检查点：自检表全部通过，或未通过项均有 `change_log` 与处置说明。

**Step 6 · 输出**
- 写 `bilingual_translation.md`（可编辑双语对照）+ `result.json`（结构化结果，含 `case_id` 与全部输出契约字段）。

## 4. 输出契约（result.json）

```json
{
  "case_id": "<原样带回>",
  "status": "success | partial | blocked",
  "artifact": {
    "source_language": "en",
    "target_language": "zh",
    "scope": "<全篇 / 段落区间说明>",
    "glossary": [{"term": "...", "translation": "...", "scope": "...", "source_location": "...", "status": "locked|needs_confirmation"}],
    "paragraphs": [
      {"para_id": "P001", "source_text": "<原文逐字>", "source_location": {"page": 1, "column": "left", "paragraph": 2, "sentences": "1-3"}, "translated_text": "<译文>", "translation_basis": "<译法依据>", "status": "ok|uncertain"}
    ]
  },
  "alignment_map": [
    {"para_id": "P001", "source_location": {"page": 1, "paragraph": 2}, "target_index": 1, "continuation_of": null, "note": "段号一一对应"}
  ],
  "relation_checks": [
    {"para_id": "P001", "relation_type": "negation|qualifier|coreference|causality|number_unit|formula|citation|figure_table", "source_evidence": "...", "target_evidence": "...", "preserved": true, "note": "..."}
  ],
  "glossary_lock_check": [{"term": "...", "expected": "...", "occurrences": 4, "all_match": true, "mismatch_at": []}],
  "verification_ledger": [
    {"para_id": "P001", "translation_basis": "直译；术语按锁表 term X", "checked_relations": ["negation", "number_unit"], "issues": []}
  ],
  "uncertainties": ["..."],
  "human_review_items": ["..."],
  "change_log": [{"step": 4, "action": "修正 P003 否定关系误译", "reason": "原文 not necessarily 误译为必然"}],
  "self_check": {"terms_locked": true, "paragraphs_complete": true, "relations_preserved": true, "verifiable": true}
}
```

字段原则：`status` 诚实反映完成情况；`partial` 写清 `uncertainties` 与缺口；`blocked` 说明拦截原因（如无文字层）；译不准标 `uncertain` 并进 `human_review_items`；不得编造原文没有的内容、数值、benchmark 分数或软件状态。

## 5. 质量自检清单

执行前完整读取 `references/checklist.md`，逐项打钩。关键硬项：

- [ ] 术语表覆盖关键概念，全篇译名与锁表逐字一致
- [ ] 标题、摘要、正文、图注、结论中同一术语译名统一
- [ ] 段落与原文一一对应，无漏段、重复、错位
- [ ] 跨页跨栏续段用 continuation_of 链接，未拼接
- [ ] 七类关系（指代/否定限定/因果让步/数字单位/公式/引用/图表号）逐项核验
- [ ] 数字、单位、公式符号、文献编号与原文逐字一致
- [ ] 每段有 translation_basis 与原文回链（页码/段落/句子）
- [ ] 译不准/原文残缺处标 uncertain 并进 human_review_items
- [ ] 无编造内容、数值、benchmark 分数或软件状态
- [ ] 独立核对完成，改动均在 change_log

## 6. 异常处理与边界

| 情况 | 处理 |
|---|---|
| 原文无文字层/扫描件 | `status=blocked`，说明无法读取，不凭空翻译 |
| 原文残缺/断行丢字 | 在对应段标 `uncertain`，按可见内容译并注明缺失，进人工确认 |
| 术语多义未决 | 不擅自定译，标 `needs_confirmation`，暂用中性译法并加注 |
| 公式/表格为图片 | 保留编号占位，标 `image_based`，不臆造符号内容 |
| 长文跨页跨栏 | 用 continuation_of 链接，段号一一对应，不漏不重 |
| 语言混杂 | 保留源语言专名，目标语言中说明，不强行统一 |
| 文献引用无法解析 | 保留原引用标记，标 `citation_unresolved`，进人工确认 |
| 数字单位换算存疑 | 保留原文数值单位，标 `number_uncertain`，不擅自换算 |
| 用户要求"只要大意" | 说明本 Skill 产出逐段对齐译文，如仅需大意可转普通翻译 |

## 7. 最小调用示例（示意，非真实案例）

**输入**（`case.json` 片段）：
```json
{"case_id":"demo-001","task":"把这篇论文第2节译成中文，术语要统一并给出双语对照","source_language":"en","target_language":"zh","scope":"第2节全部段落","materials":{"pdf":"/input/materials/paper.pdf"}}
```
**调用**：加载本 Skill 后，用户说"用 glossary-locked-aligned-translator 翻译第2节，术语统一、给双语对照、译不准的标出来"。

**示意输出**（节选）：
```json
{"case_id":"demo-001","status":"success","artifact":{"source_language":"en","target_language":"zh","glossary":[{"term":"representation learning","translation":"表示学习","status":"locked"}],"paragraphs":[{"para_id":"P001","source_text":"...","source_location":{"page":2,"paragraph":1},"translated_text":"……（示意译文）","translation_basis":"直译，术语按锁表","status":"ok"}]},"relation_checks":[{"para_id":"P001","relation_type":"negation","source_evidence":"not necessarily","target_evidence":"未必","preserved":true}],"self_check":{"terms_locked":true,"paragraphs_complete":true,"relations_preserved":true,"verifiable":true}}
```
> 上述为演示接口用的虚构示例，不代表任何真实案例结果。

## 8. 资源清单

- `references/method.md` —— GLAT 四机制细则与反例库
- `references/checklist.md` —— 覆盖四类关键问题的逐项自检清单
- `references/schema.json` —— `result.json` 字段字典（JSON Schema draft-07）
- `references/relation_types.md` —— 七类关系核验规则与术语表格式
- `assets/bilingual_template.md` —— 可编辑双语对照模板
- `examples/` —— 最小调用示例（已标注为示意，非真实案例）
- `README.md` —— 目录结构、使用方式、依赖与评分对应说明
- `GITHUB_URL.txt` —— 提交前替换占位链接
