# Orchestrator

Autonomous orchestrator for `thesis-advisor-review`.

## Role

Drive a section-by-section advisor review loop until all sections pass.

## Refer to (Required)

- [specs/state-schema.md](../specs/state-schema.md)
- [specs/quality-standards.md](../specs/quality-standards.md)

## State Management

### Read State

```javascript
const state = JSON.parse(Read(`${workDir}/state.json`));
```

### Update State

```javascript
function updateState(workDir, updates) {
  const statePath = `${workDir}/state.json`;
  const state = JSON.parse(Read(statePath));
  const newState = { ...state, ...updates, updated_at: new Date().toISOString() };
  Write(statePath, JSON.stringify(newState, null, 2));
  return newState;
}
```

## Decision Logic (concept)

```javascript
function selectNextAction(state) {
  if (state.error_count >= 3) return 'action-abort';
  if (state.status === 'pending') return 'action-init';
  if (!state.doc_path) return 'action-wait-doc-path';
  if (!state.toc?.length) return 'action-load-markdown';

  // If we are waiting for user revision, we must gate first
  if (state.status === 'waiting_for_revision') return 'action-gate-and-advance';

  // Otherwise select section and review
  if (!state.current_section_id) return 'action-select-next-section';
  return 'action-review-section';
}
```

## Actions

| Action | Purpose | Preconditions |
|--------|---------|---------------|
| [action-init](actions/action-init.md) | Initialize workDir and state | none |
| action-wait-doc-path | Ask user for markdown path | state.doc_path is null |
| [action-load-markdown](actions/action-load-markdown.md) | Parse markdown TOC & slice sections | state.doc_path set |
| [action-select-next-section](actions/action-select-next-section.md) | Pick next unpassed section | toc loaded |
| [action-review-section](actions/action-review-section.md) | Review current section | current_section_id set |
| [action-gate-and-advance](actions/action-gate-and-advance.md) | Gate decision and advance | waiting_for_revision |

## Termination Conditions

- All sections have `gate === "pass"` → set `state.status = "completed"`
- Error count >= 3 → abort and preserve state

## Error Recovery

| Error | Strategy |
|------|----------|
| Cannot parse headings | Ask user to normalize markdown headings (# / ##) |
| Section too long | Ask user to split the section into smaller subsections |
| User disagrees with gate | Explain which must-fix remains and what evidence would resolve it |
