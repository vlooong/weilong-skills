# Action: Init

Initialize working directory and state for thesis-advisor-review.

## Role

Create a new workDir under `.workflow/.scratchpad/` and write an initial `state.json`.

## Preconditions

- None.

## Steps

```javascript
const timestamp = new Date().toISOString().slice(0,19).replace(/[-:T]/g, '');
const workDir = `.workflow/.scratchpad/thesis-advisor-review-${timestamp}`;

Bash(`mkdir -p "${workDir}"`);
Bash(`mkdir -p "${workDir}/input" "${workDir}/sections" "${workDir}/reviews" "${workDir}/revisions" "${workDir}/final"`);

const state = {
  status: "pending",
  doc_path: null,
  toc: [],
  current_section_id: null,
  completed_section_ids: [],
  error_count: 0,
  updated_at: new Date().toISOString()
};

Write(`${workDir}/state.json`, JSON.stringify(state, null, 2));

return {
  status: "completed",
  workDir,
  output_file: `${workDir}/state.json`,
  summary: "Initialized workDir and state.json"
};
```
