# weilong-skills

A collection of AI skills for research and writing workflows. Each skill provides a structured, multi-phase workflow for a specific task.

## Skills

### 📄 patent-draft

**Path**: [`patent-draft/`](patent-draft/)

A skill for drafting patent disclosure documents (专利交底书). Guides the AI through literature review, technical analysis, and structured document writing.

**Trigger keywords**: 专利, 发明专利, 实用新型, 交底书, 技术方案, 创新点, write/create/draft patent disclosures

**Workflow**:
1. **Literature Review** — search existing patents and research, extract key technical terms and problems
2. **Technical Analysis** — confirm the technical approach and innovation points with the user
3. **Document Writing** — produce a complete patent disclosure following the standard chapter structure

**Key files**:
- [`skill.md`](patent-draft/skill.md) — full skill definition, writing principles, and chapter templates
- [`references/template.md`](patent-draft/references/template.md) — standard disclosure document template
- [`references/example-good.md`](patent-draft/references/example-good.md) — annotated good example
- [`references/pitfalls.md`](patent-draft/references/pitfalls.md) — common mistakes to avoid

---

### 🎓 thesis-advisor-review

**Path**: [`thesis-advisor-review/`](thesis-advisor-review/)

A skill that acts as a thesis advisor, reviewing a master's thesis section by section. Each section must pass a quality gate before moving on.

**Trigger phrases**: `论文审核`, `导师审核`, `逐节修改`, `毕业论文修改`, `thesis review`, `advisor review`

**Workflow**:
1. **Specification Study** — read quality standards and review requirements
2. **Document Loading** — parse the Markdown thesis into a chapter/section index
3. **Section Review Loop** — review each `##` section with advisor-style annotations (problem → reason → suggestion → rewrite example)
4. **Gate & Advance** — issue PASS / REVISE / BLOCK; iterate until PASS, then move to the next section

**Key files**:
- [`SKILL.md`](thesis-advisor-review/SKILL.md) — skill definition and execution flow
- [`README.md`](thesis-advisor-review/README.md) — quick usage guide
- [`specs/`](thesis-advisor-review/specs/) — review requirements, quality standards, output style, state schema
- [`templates/`](thesis-advisor-review/templates/) — section review and gate decision output templates
- [`phases/`](thesis-advisor-review/phases/) — per-action implementation guides

## Input format

Both skills expect thesis/document content as a **single Markdown file**, using `#` for chapters and `##` for sections.

## Repository structure

```
weilong-skills/
├── patent-draft/          # Patent disclosure drafting skill
│   ├── skill.md
│   └── references/
└── thesis-advisor-review/ # Thesis advisor review skill
    ├── SKILL.md
    ├── README.md
    ├── workflow.json
    ├── specs/
    ├── templates/
    └── phases/
```
