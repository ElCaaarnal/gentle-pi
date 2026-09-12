# Orchestrator — Memory Detail (lazy-loaded)

Bind this to the parent Pi session only, on SDD phase memory reads/writes. Not always-on; loaded on demand from `assets/orchestrator.md`'s `## Memory Contract` pointer.

### SDD phases

Each write-owning SDD phase subagent reads its required inputs directly from the active backend and persists its artifact before returning. `sdd-research` is the exception: it is an output-only evidence collector, receives persisted intent from the parent, and returns a complete envelope. The parent orchestrator validates and persists that envelope through the preflight-selected store.

| Phase          | Reads                                                   | Writes           |
| -------------- | ------------------------------------------------------- | ---------------- |
| `sdd-explore`  | nothing                                                 | `explore`        |
| `sdd-research` | parent-supplied intent (no backend read)                 | none — parent writes `research` + `preproposal` |
| `sdd-proposal` | exploration (optional)                                  | `proposal`       |
| `sdd-spec`     | proposal (required)                                     | `spec`           |
| `sdd-design`   | proposal (required)                                     | `design`         |
| `sdd-tasks`    | spec + design (required)                                | `tasks`          |
| `sdd-apply`    | tasks + spec + design + `apply-progress` (if it exists) | `apply-progress` |
| `sdd-verify`   | spec + tasks + `apply-progress`                         | `verify-report`  |
| `sdd-sync`     | proposal + spec + design + tasks + `verify-report`      | `sync-report`    |
| `sdd-archive`  | all artifacts                                           | `archive-report` |
| `sdd-status`   | change artifacts (read-only)                            | nothing          |

- SDD artifact keys: in memory/hybrid mode, phase artifacts use stable topic keys such as `sdd/<change>/proposal`, `sdd/<change>/spec`, `sdd/<change>/design`, `sdd/<change>/tasks`, `sdd/<change>/apply-progress`, `sdd/<change>/verify-report`, `sdd/<change>/sync-report`, and `sdd/<change>/archive-report`.
- When the optional research lane is selected, the output-only evidence collector returns both records inline. The parent orchestrator validates and persists them under `sdd/<change>/research` and `sdd/<change>/preproposal` (openspec: `openspec/changes/<change>/research.md`).
- If memory tools are unavailable, do not pretend persistence exists; return artifacts inline and/or write OpenSpec files.

Memory lifecycle rule (when Engram exposes lifecycle metadata/tooling):

- At session start or before architecture-sensitive work, call the injected Engram review tool with action `list` for the current project when the tool is available.
- If the injected Engram review tool is unavailable, do not fail the task. Continue with the injected Engram context/search tools, and still apply lifecycle metadata from any returned observations when present.
- `active` memories may be used normally.
- `needs_review` memories are stale context, not trusted facts.
- When a retrieved memory is marked `needs_review`, surface that stale context to the user and verify it against current evidence before relying on it.
- Do NOT call the injected Engram review tool with action `mark_reviewed` automatically. Only call `mark_reviewed` after explicit user confirmation or through a dedicated memory maintenance command.
