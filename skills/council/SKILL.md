---
name: council
description: Trigger this if the user says 'council'.
---

# Council

Before an audit, address outstanding PR comments, fix known issues, and run
relevant checks. Give council the updated implementation; do not defer known
fixes until reviews return. If the user wants to discard the implementation
and explore new ideas, skip fixes to code being replaced.

Bring in three delegates to help with the user's request, usually for review
or feedback. If the user's instructions name council members or effort levels,
use those. Otherwise use these defaults across three different providers:

- **Gemini 4 Neon** at **medium** reasoning effort
- **Claude Opus** at **medium** reasoning effort
- **Codex Astra** (GPT Sol) at **high** reasoning effort

In T3 Code, use `orchestrator_capabilities` to resolve the models, then
`delegate_task` to start one delegate per model in parallel. Elsewhere, use
the harness's subagent tools. If you cannot reach three providers, or a named
member is unavailable, report it instead of substituting.

Give each delegate the request and relevant context. For review or feedback,
keep their work read-only. Collect their responses and summarize the useful
findings, disagreements, and recommended next steps.

Do not wait idle. When a result arrives, check its findings against the code
at once and decide what to fix while the other delegates run. Do not edit
files while a delegate still reviews the same checkout. If the user's
instructions mark a delegate as non-blocking, do not wait for it; treat a
late result as feedback and do not record its score.

For a code review, ask each delegate for a score out of 10.
Count each full review pass as one round.
A fresh review after fixes starts a new round.

If the work ends in a pull request, put the results in the PR description.
Put this section before the model and harness line at the end:

```markdown
## Council review

Rounds: 2

| Delegate | Round 1 | Round 2 |
| --- | --- | --- |
| Claude Opus (medium) | 7/10 | 9/10 |
| Codex Astra (high) | 8/10 | 10/10 |
| Gemini 4 Neon (medium) | 7/10 | 9/10 |
```

If a delegate gives no score, write `no score`. Do not invent one.
If the PR is open, edit the description. Do not post a comment for this.
If there is no PR, put the same results in the final reply only.

If you agree with their recommendations, make the changes and request a fresh
review after. Continue this process until each delegate is at least 9/10
satisfied with the work (9/10 or higher).

To avoid burning quota (especially Claude Opus) and prevent scope creep across
follow-up rounds:

- **Incremental diffs on Round 2+:** Send the full diff on Round 1 only. On
  follow-up rounds, send only the incremental diff since that delegate's last
  reviewed commit, the touched files, and a short checklist of how each prior
  finding was resolved. Do not re-send the full cumulative diff unless a
  delegate explicitly requests a specific file.
- **Skip already-passing delegates on small fixes:** If a delegate already
  scored 9/10 or higher, do not re-run them for a small, localized fix made
  for another delegate unless the change touches a high-risk path (such as
  submission, auth/credentials, data loss, or public API contracts) or
  materially alters behavior. Note their carried score in the table (for
  example, `9/10 (carried)`).
- **Blocker-only follow-up fixes:** Starting on Round 2, fix only the specific
  blocking findings cited as keeping a score below 9/10. Push back on new
  nice-to-have edge cases or style suggestions, defer them explicitly, and
  keep scope limited.
- **Round cap:** If any delegate is still below 9/10 after 3 rounds, or if
  scores drop after a fix round, stop and check in with the user with the
  remaining disagreements instead of looping indefinitely.
