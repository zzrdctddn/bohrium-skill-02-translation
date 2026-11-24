# 文献翻译 Skill（glossary-locked-aligned-translator）

一个把科研论文转成"可逐段核验双语对照"的 Agent Skill。核心：术语锁表 → 段落对齐 → 关系保真 → 译法留痕，术语稳、段落全、关系准、可核验。

## 目录结构
- `SKILL.md` —— Skill 本体（YAML frontmatter、GLAT 方法、SOP、输出契约）
- `references/` —— 方法细则与反例库、自检清单、字段字典、关系核验规则
- `assets/` —— 可编辑双语对照模板
- `examples/` —— 最小调用示例（已标注为示意，非真实案例）
- `GITHUB_URL.txt` —— 提交前填 GitHub 公开链接（当前为 REPLACE_ME 占位）

## 使用方式
1. 将本目录作为 Skill 安装到 Agent（平台约定路径 `/skills/weekly-skill`）。
2. 输入 `/input/case.json`（含 `case_id`、任务、源/目标语言、翻译范围、原文路径、可选术语表）与 `task.md`。
3. 自然语言下指令，如"翻译第2节，术语统一、给双语对照、译不准的标出来"。
4. 按 SKILL.md 的 SOP 执行，输出到 `/app/output/result.json` 与 `bilingual_translation.md`。

## 依赖
- 仅需文本处理与模型翻译能力；无外部运行时依赖。术语表格式、段号对齐约定、关系类型表内置于 SKILL.md 与 `references/`。
- 注：Runtime contract 规定预算为 64 次模型请求 / 900 秒 / 每次 16384 token；资源≤50 个 UTF-8 文本文件、单文件≤500KB、总计≤2MB、无软链。原文以 `/input` 实际文件为准，不依据领域标签臆测内容。

## 适用与限制
- 适用：全篇或指定段落区间的科研论文翻译、术语一致性强制统一、长文分段续译、段落级双语对照、译文核验与人工确认清单。
- 限制：扫描件无文字层标 `blocked`；译不准/原文残缺/多义未决标 `uncertain` 并进人工确认；公式与图表按编号占位保真，不解读科学含义。材料缺失、无法判断、运行失败分别说明，不编造事实与 benchmark 分数。

## 评分对应
实际效果(40)、使用增益(25)、创新增益(15)、可靠体验(10)、生态贡献(10) 的支撑材料分别见 SKILL.md 第 3/2/2/6/8 节与 `references/`：
- 实际效果：SOP Step 1~4（术语稳/段落全/关系准/可核验）+ `relation_checks`/`verification_ledger`。
- 使用增益：GLAT 四机制带来的额外结构（锁表、对齐表、关系核验、译法留痕）+ `self_check` 对照字段。
- 创新增益：术语强制回填、跨页段落接续、段落级对齐、关系保真核验为基线难稳定实现的能力，见 `references/method.md`。
- 可靠体验：Step 0 边界确认 + 第 6 节异常处理表。
- 生态贡献：术语表格式、关系类型表、双语模板与示例见 `references/`、`assets/`、`examples/`。
