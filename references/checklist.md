# 文献翻译自检清单（逐项打钩）

> 以"校对者"身份独立执行，不与翻译阶段合并。每项不通过须回退对应步骤并在 `change_log` 记录。

## 术语稳（Glossary Lock）
- [ ] 术语表覆盖标题/摘要/正文/图注/结论的关键概念
- [ ] 每个原词只有一个锁定译名（缩写/同义已归并）
- [ ] `glossary_lock_check` 统计出现次数，全部匹配锁表
- [ ] 自抽术语标 needs_confirmation 并进人工确认
- [ ] 锁表确定后全篇无改译

## 段落全（Paragraph Alignment）
- [ ] 段落与原文逐段清点一致，无漏段
- [ ] 无重复段、无错位段
- [ ] 每段有唯一 para_id 与原文位置（页/栏/段号/句边界）
- [ ] 跨页跨栏跨章节续段用 continuation_of 链接，未拼接
- [ ] 图注、表头、公式段一并纳入对齐

## 关系准（Relation Fidelity，七类）
- [ ] 指代对象（coreference）落到同一实体
- [ ] 否定（not/no/never）完整保留，未反转
- [ ] 限定条件（only/partially/under condition）保留
- [ ] 因果与让步（because/however/although）关系一致
- [ ] 数字与单位逐字一致，未擅自换算
- [ ] 公式符号与编号占位保留，未译改
- [ ] 文献引用编号与图表编号保留且对应

## 可核验（Verification Ledger）
- [ ] 每段有 translation_basis（译法依据）
- [ ] 每段回链原文页码/段落/句子
- [ ] 每段 checked_relations 与 issues 已记录
- [ ] 译不准/原文残缺/多义未决标 uncertain，进 human_review_items
- [ ] 未生成原文没有的内容、数值或结论

## 结构与边界
- [ ] artifact 含 source_language/target_language/scope/glossary/paragraphs
- [ ] alignment_map、relation_checks、glossary_lock_check、verification_ledger 齐全
- [ ] status 诚实（success/partial/blocked）
- [ ] partial/blocked 写明原因与缺口
- [ ] 无编造内容、数值、benchmark 分数或软件状态
- [ ] 独立核对完成，所有改动均在 change_log
