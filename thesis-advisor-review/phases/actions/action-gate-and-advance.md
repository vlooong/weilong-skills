# Action: Gate & Advance

After user provides revised section text, decide PASS / REVISE / BLOCK, update state, and (if PASS) advance pointer.

## Refer to

- [specs/quality-standards.md](../../specs/quality-standards.md)
- [templates/gate-decision-template.md](../../templates/gate-decision-template.md)
- [specs/state-schema.md](../../specs/state-schema.md)

## Inputs

- User supplies revised text for the current section

## Steps (concept)

```javascript
const statePath = `${workDir}/state.json`;
const state = JSON.parse(Read(statePath));
const sectionId = state.current_section_id;

// 1) Persist user revision
// Write(`${workDir}/revisions/${sectionId}-iterN.md`, revisedText)

// 2) In chat: evaluate against quality gates
// Decide gate = pass|revise|block

// 3) Update state.toc[...].sections[...].gate

// 4) If pass: push to completed_section_ids and select next section

return {
  status: 'completed',
  output_file: statePath,
  summary: 'Gate decision applied (pass/revise/block) and state updated.'
};
```

> 注：门控判定依赖对话式审核内容，属于“人机协作”环节；此 action 文档主要固定状态更新的框架。
