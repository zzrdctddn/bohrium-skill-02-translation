# 示例说明（示意，非真实案例）

本目录提供一组最小调用示例，仅用于说明输入输出接口与完整使用流程，不构成任何真实案例结果或正式答案。

- `example_input.json`：虚构输入，含 `case_id` 与 `note`（注明"演示接口用，非官方案例"）。
- `example_output.json`：虚构输出，含 `case_id`、`status`、主产物 `artifact`（术语表+段落双语）、`alignment_map`、`relation_checks`、`glossary_lock_check`、`verification_ledger`、`uncertainties`、`human_review_items`、`change_log`、`self_check`。译文、原文位置与关系证据均为占位示意。

## 调用流程
1. 加载 `glossary-locked-aligned-translator` Skill。
2. 按 `example_input.json` 的形态准备 `/input/case.json` 与 `task.md`（真实运行时原文为实际文件）。
3. 自然语言下指令，如"翻译第2节，术语统一、给双语对照、译不准的标出来"。
4. 按 SKILL.md 的 SOP 执行，产出 `result.json` 与 `bilingual_translation.md`。

## 复用提示
- 示例中的 `source_language`、`target_language`、`scope` 可替换为你自己的任务需求；术语表格式与关系类型见 `references/relation_types.md`。
- 不要把示例中的虚构译文、位置或结论当作真实翻译结果；正式结果必须来自对真实原文的实际翻译与核对。
