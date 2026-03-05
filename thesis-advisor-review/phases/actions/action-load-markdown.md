# Action: Load Markdown

Parse a single Markdown thesis file, build chapter/section index, and slice each `##` section into `sections/*.md`.

## Refer to

- [specs/thesis-review-requirements.md](../../specs/thesis-review-requirements.md)
- [specs/state-schema.md](../../specs/state-schema.md)

## Inputs

- `workDir/state.json` must exist
- User provides `doc_path` (absolute or relative to workspace)

## Steps

```javascript
const statePath = `${workDir}/state.json`;
const state = JSON.parse(Read(statePath));

// 1) Read markdown
const md = Read(state.doc_path);

// 2) Split lines and detect headings
const lines = md.split(/\r?\n/);

let currentChapter = null;
let chapterIndex = -1;

const toc = [];
const sections = []; // flat list

function slugify(s) {
  return s
    .toLowerCase()
    .replace(/[^a-z0-9\u4e00-\u9fa5]+/g, '-')
    .replace(/^-+|-+$/g, '')
    .slice(0, 60);
}

for (let i = 0; i < lines.length; i++) {
  const line = lines[i];

  if (line.startsWith('# ')) {
    currentChapter = line.trim();
    chapterIndex++;
    toc.push({ chapter: currentChapter, sections: [] });
  }

  if (line.startsWith('## ')) {
    const title = line.trim();

    const id = `ch${Math.max(1, chapterIndex + 1)}-${slugify(title.replace(/^##\s+/, '')) || `sec-${i+1}`}`;

    // find end line: next '## ' or '# ' or EOF
    let end = lines.length - 1;
    for (let j = i + 1; j < lines.length; j++) {
      if (lines[j].startsWith('## ') || lines[j].startsWith('# ')) {
        end = j - 1;
        break;
      }
    }

    const sectionObj = {
      id,
      title,
      start_line: i + 1,
      end_line: end + 1,
      gate: 'unreviewed',
      iteration: 0
    };

    if (toc.length === 0) {
      // No chapter detected yet; create implicit chapter
      toc.push({ chapter: '# (implicit)', sections: [] });
    }

    toc[toc.length - 1].sections.push(sectionObj);
    sections.push(sectionObj);

    // slice section text
    const sectionText = lines.slice(i, end + 1).join('\n');
    Write(`${workDir}/sections/${sectionObj.id}.md`, sectionText);
  }
}

// 3) Update state: toc + current_section_id if null
const currentSectionId = state.current_section_id || (sections[0]?.id ?? null);

const newState = {
  ...state,
  status: currentSectionId ? 'running' : 'aborted',
  toc,
  current_section_id: currentSectionId,
  updated_at: new Date().toISOString()
};

Write(statePath, JSON.stringify(newState, null, 2));

return {
  status: "completed",
  output_file: statePath,
  summary: `Loaded markdown. Sections: ${sections.length}. Current: ${currentSectionId}`
};
```
