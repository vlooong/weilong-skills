# Action: Select Next Section

Select the next `##` section to review based on state.

## Refer to

- [specs/state-schema.md](../../specs/state-schema.md)

## Steps

```javascript
const statePath = `${workDir}/state.json`;
const state = JSON.parse(Read(statePath));

if (!state.toc?.length) {
  return {
    status: "failed",
    output_file: statePath,
    summary: "No TOC loaded. Run action-load-markdown first."
  };
}

// Flatten sections
const sections = state.toc.flatMap(ch => ch.sections || []);

// Find first section not passed
const next = sections.find(s => s.gate !== 'pass');

if (!next) {
  const newState = { ...state, status: 'completed', current_section_id: null, updated_at: new Date().toISOString() };
  Write(statePath, JSON.stringify(newState, null, 2));
  return { status: 'completed', output_file: statePath, summary: 'All sections passed. Thesis review completed.' };
}

const newState = { ...state, current_section_id: next.id, updated_at: new Date().toISOString() };
Write(statePath, JSON.stringify(newState, null, 2));

return {
  status: 'completed',
  output_file: statePath,
  summary: `Selected section ${next.id}`
};
```
