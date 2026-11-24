# GLAT 方法细则与反例库

## 四机制依赖关系
GLAT 的四机制有严格先后；跳过前置会级联失败：

- **术语稳（Glossary Lock）→ 段落全（Paragraph Alignment）**：锁表确定后逐段翻译才有统一用词基准。
- **段落全 → 关系准（Relation Fidelity）**：段号一一对应，关系核验才知道"这一段对应哪一段原文"。
- **关系准 → 可核验（Verification Ledger）**：关系核验记录构成译法留痕的证据。

## 机制一：术语稳 —— 术语锁表
- 术语来源优先级：用户提供术语表 > 原文抽取并人工确认 > 模型初拟（须标 needs_confirmation）。
- 锁表字段：`term`（原词）、`translation`（锁定译名）、`scope`（适用范围/领域）、`source_location`（原文出处）、`status`（locked / needs_confirmation）。
- 同义表述与缩写归并：同一概念的多种原文形式映射到同一锁定译名，并在 scope 注明。
- 硬规则：锁表确定后，标题/摘要/正文/图注/结论中该词**逐字使用锁定译名**；`glossary_lock_check` 统计出现次数并核对，不一致即回退重译。

## 机制二：段落全 —— 段落对齐
- 切分依据语义边界，不按固定字数硬切；每段唯一 `para_id`。
- 每段记录原文位置：页码、栏、段号、句子边界（如"1-3"）。
- 跨页/跨栏/跨章节续段用 `continuation_of` 链接，各段独立成条，**不拼接、不合并**。
- `alignment_map` 段号与原文一一对应；段落清点以原文逐段为准，不凭译文行数推断。

## 机制三：关系准 —— 关系保真（七类）
逐段核对以下七类，详见 `relation_types.md`：
1. 指代对象（coreference）：this/these/it/前者/该……指代是否落到同一实体。
2. 否定与限定（negation/qualifier）：not/no/never/only/partially/under condition 等是否保留。
3. 因果与让步（causality/concession）：because/therefore/however/although 等逻辑关系是否一致。
4. 数字与单位（number_unit）：数值、单位、量级、比较关系逐字一致，不擅自换算。
5. 公式（formula）：符号、上下标、编号占位保留，不译不改号。
6. 文献引用（citation）：引用编号/作者年标记保留，不重排。
7. 图表编号（figure_table）：图号表号保留，与题注对应。
任一 `preserved=false` 标 `relation_mismatch` 并回退 Step 3 重译。

## 机制四：可核验 —— 译法留痕
- 每段 `translation_basis` 取值：直译 / 意译+理由 / 术语锁表条目 / 多义存疑说明。
- 每段回链原文页码、段落、句子；`verification_ledger` 记录 checked_relations 与 issues。
- 译不准、原文残缺、多义未决标 `uncertain` 并进 `human_review_items`，含混处理是硬错误。

## 翻译与核对分离
翻译阶段禁止自我核对；翻译完成后以"校对者"身份独立执行 checklist，发现错误回退对应步骤并写 `change_log`。合并两遍会漏掉自己引入的术语漂移与否定误译。

## 反例库
| 反例 | 为什么错 | 改法 |
|---|---|---|
| 同一术语标题译 A、正文译 B、结论译 C | 术语漂移，不可核验 | 建锁表，全篇逐字回填 |
| 把 "not necessarily" 译成"必然" | 否定丢失，语义反转 | 逐段核对 negation，标 mismatch 重译 |
| 把 "only under X" 的限定删掉 | 限定条件丢失，结论被夸大 | qualifier 核验，保留限定 |
| 跨页段落译成一段 | 段号错位，无法回链 | continuation_of 链接，段号一一对应 |
| 漏掉图注段落 | 段落不全 | 按原文逐段清点，不凭译文行数 |
| 数字单位擅自换算（如 1e5 写成 10 万） | 与原文不逐字一致 | 保留原文数值单位，存疑标 number_uncertain |
| 文献编号 [3] 译成"第三篇文献" | 引用标记被改写 | 保留原引用编号占位 |
| 译不准处写"大意如此"含混带过 | 不可核验 | 标 uncertain + human_review_items |
| 无对照记录却自称"术语一致率提升" | 自述不能替代证据 | 用 glossary_lock_check/self_check 证明，不编 benchmark |
