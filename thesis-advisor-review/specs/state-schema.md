# State Schema

定义 `state.json` 的字段及其含义，用于 autonomous orchestrator 的决策。

## When to Use

| Phase | Usage |
|-------|-------|
| Init | 初始化 state |
| Select Next | 选择下一节 |
| Gate & Advance | 更新通过记录与推进指针 |

---

## State JSON (concept)

```json
{
  "status": "pending|running|waiting_for_revision|completed|aborted",
  "doc_path": "<path to markdown>",
  "toc": [
    {
      "chapter": "# ...",
      "sections": [
        {
          "id": "ch1-sec2",
          "title": "## ...",
          "start_line": 120,
          "end_line": 210,
          "gate": "unreviewed|revise|pass|block",
          "iteration": 0
        }
      ]
    }
  ],
  "current_section_id": "ch1-sec2",
  "completed_section_ids": [],
  "error_count": 0,
  "updated_at": "..."
}
```

## Invariants

- `current_section_id` 必须指向一个存在于 `toc.sections[].id` 的小节。
- 若某小节 `gate === "pass"`，则其 id 必须出现在 `completed_section_ids`。
- 任意时刻只审核一个小节。

