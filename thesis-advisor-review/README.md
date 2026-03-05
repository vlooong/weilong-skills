# Thesis Advisor Review (Usage)

## How to use

Trigger the skill with phrases like:
- “论文审核：请按节审核我的论文”
- “导师审核：从第 1 章第 1 节开始逐节打磨”
- “thesis review: review section by section”

## Input preparation (default)

- Provide a **single Markdown file**.
- Use `#` for chapters and `##` for sections.

## Typical loop

1. You provide the thesis markdown path (or paste a section)
2. I review the current `##` section with advisor-style annotations
3. You revise and send back the updated section
4. I gate: PASS / REVISE / BLOCK
5. If PASS → next section

## Notes

- I will not fabricate results or data.
- If a section is too long, I will ask you to split it.
