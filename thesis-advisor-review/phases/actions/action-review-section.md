# Action: Review Section

Review the current section (`state.current_section_id`) and produce advisor-style comments and a minimal revision checklist.

## Refer to

- [specs/output-style.md](../../specs/output-style.md)
- [specs/thesis-review-requirements.md](../../specs/thesis-review-requirements.md)
- [templates/section-review-template.md](../../templates/section-review-template.md)

## Steps

```javascript
const statePath = `${workDir}/state.json`;
const state = JSON.parse(Read(statePath));

const sectionId = state.current_section_id;
if (!sectionId) {
  return { status: 'failed', output_file: statePath, summary: 'No current_section_id set.' };
}

const sectionPath = `${workDir}/sections/${sectionId}.md`;
const sectionText = Read(sectionPath);

// Increment iteration counter for this section
const toc = state.toc.map(ch => ({
  ...ch,
  sections: (ch.sections || []).map(s =>
    s.id === sectionId ? { ...s, iteration: (s.iteration || 0) + 1 } : s
  )
}));

const updatedAt = new Date().toISOString();
Write(statePath, JSON.stringify({ ...state, toc, status: 'waiting_for_revision', updated_at: updatedAt }, null, 2));

// The actual review content is produced by the assistant in chat, but we persist a copy too.
const reviewId = `${sectionId}-iter${(toc.flatMap(c => c.sections).find(s => s.id === sectionId)?.iteration) || 1}`;
const reviewPath = `${workDir}/reviews/${reviewId}.md`;

// Placeholder: the operator (assistant) should replace this with real content when running interactively.
const reviewDoc = `# Review: ${sectionId}\n\n(Write advisor-style review here following templates/section-review-template.md)\n\n---\n\n## Section Text\n\n${sectionText}`;
Write(reviewPath, reviewDoc);

return {
  status: 'completed',
  output_file: reviewPath,
  summary: `Prepared review file scaffold for ${sectionId}`
};
```
