# 关系类型核验规则与术语表格式

本文件是 GLAT 机制一、三的字段级细则，供 Agent 执行时查表。

## 1. 七类关系核验规则（relation_type）
| 类型 | 检查对象 | 原文侧证据示例 | 译文侧要求 | 不符标记 |
|---|---|---|---|---|
| coreference | 指代对象 | this / these / it / 前者 / 该 | 指代落到同一实体，不漂移到邻近名词 | relation_mismatch |
| negation | 否定 | not / no / never / neither / 并非 | 否定完整保留，不得反转或漏译 | relation_mismatch |
| qualifier | 限定条件 | only / merely / partially / under X / in most cases | 限定语保留，不省略、不泛化 | relation_mismatch |
| causality | 因果 | because / therefore / thus / 由于 | 因果方向与强度一致 | relation_mismatch |
| concession | 让步 | however / although / despite / 尽管 | 让步关系一致，不改成因果 | relation_mismatch |
| number_unit | 数字单位 | 1.5×10⁵ / 3 nm / 20% / 表2 | 数值、单位、量级、比较关系逐字一致，不换算 | number_uncertain |
| formula | 公式符号 | E = mc² / 式(3) / α, β | 符号编号占位保留，不译不改号 | symbol_uncertain |
| citation | 文献引用 | [3] / (Smith, 2020) | 引用标记原样保留，不重排不改写 | citation_unresolved |
| figure_table | 图表编号 | Fig. 2 / Table 1 | 编号保留，与题注对应 | mismatch |

> 注：causality 与 concession 可合并为 `causality` 一类记录，也可分列；schema 中以枚举值约束，二者合并时用 `causality` 并在 note 注明让步关系已核对。

## 2. 术语表格式（glossary）
| 字段 | 含义 | 填写规则 |
|---|---|---|
| term | 原文术语 | 逐字，含缩写与同义表述（用 `/` 分隔） |
| translation | 锁定译名 | 全篇唯一，确定后不改 |
| scope | 适用范围 | 领域/语境，如"本文指方法中的隐向量" |
| source_location | 原文出处 | 页码/段落/图注，便于回查 |
| status | 状态 | locked（已确认）/ needs_confirmation（待人工确认） |

## 3. 段落对齐约定（alignment_map）
- `para_id`：P001 起连续编号，全文唯一。
- `source_location`：至少含 page 与 paragraph；有栏信息时加 column；有句子边界时加 sentences（如 "1-3"）。
- `target_index`：译文段落序号，与 para_id 一一对应。
- `continuation_of`：续段指向前一段 para_id；首段为 null。
- `note`：对齐存疑写 "alignment_uncertain: <原因>"。

## 4. 译法依据填写指引（translation_basis）
- 直译：原文结构与译文一致，术语按锁表。
- 意译：说明为何直译不达意，给出理由。
- 术语锁表条目：直接引用锁表中该词条目。
- 多义存疑：列出可能译法，标 uncertain 并进人工确认。

## 5. 不得做的事
- 不解读公式与图表的科学含义（只保真编号与符号）。
- 不擅自换算数字单位、不补全原文残缺内容。
- 不在译文中加入原文没有的解释、评价或结论。
- 不编造 benchmark 分数、软件状态或对照结果；增益须由对照字段与 self_check 证明。
