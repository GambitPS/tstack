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
or feedback. If the user's instructions name council members, use those.

Upstream tstack seats one flagship model from each of three providers. This
fork runs on Claude only, so the default council is three fresh-context
Claude Opus delegates at high effort, each with a different lens:

| Seat | Lens |
| --- | --- |
| Correctness | Bugs, broken callers, mismatch with intent, the case that happened. |
| Player | What the player or user actually sees and does: behaviour, fidelity to the reference the repo follows, regressions in nearby flows. |
| House rules | The repo's `AGENTS.md`/`CLAUDE.md` rules and Definition of Done, scope creep, and anything the change adds that the goal did not need. |

Same model, three contexts: that is lens diversity, not provider diversity.
Say so when you report, and never present it as a cross-provider review.

In Claude Code, start each seat as a native `Agent` subagent with
`model: "opus"` and `run_in_background: false`. Send them in one message so
they run in parallel. If your instructions cap concurrent subagents, run them
in waves under the cap. In T3 Code you may use `delegate_task` instead when a
seat must outlive your turn. If a named member is unavailable, report it
instead of substituting.

Give each delegate the request and relevant context. For review or feedback,
keep their work read-only. Collect their responses and summarize the useful
findings, disagreements, and recommended next steps.

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
| Opus · Correctness | 7/10 | 9/10 |
| Opus · Player | 8/10 | 10/10 |
| Opus · House rules | 7/10 | 9/10 |
```

If a delegate gives no score, write `no score`. Do not invent one.
If the PR is open, edit the description. Do not post a comment for this.
If there is no PR, put the same results in the final reply only.

If you agree with their recommendations, make the changes and request a fresh
review after. Continue this process until all delegates are satisfied with the
outcome.

Do not let repeated reviews result in scope creep. It is your job to fight to
keep scope limited.
