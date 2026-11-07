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
fork runs on Claude plus one Codex seat: two fresh-context Claude Opus
delegates and one GPT-6.1 Sol delegate, all at high effort, each with a
different lens:

| Seat | Model | Lens |
| --- | --- | --- |
| Correctness | GPT-6.1 Sol (Codex) | Bugs, broken callers, mismatch with intent, the case that happened. |
| Player | Claude Opus | What the player or user actually sees and does: behaviour, fidelity to the reference the repo follows, regressions in nearby flows. |
| House rules | Claude Opus | The repo's `AGENTS.md`/`CLAUDE.md` rules and Definition of Done, scope creep, and anything the change adds that the goal did not need. |

Two providers, three lenses. Say so when you report. Use GPT-6.1 Sol, not
GPT-6 Astra, unless the user asks for Astra: the Codex seat runs on a ChatGPT
Plus allowance that it shares with image generation, and Astra uses about five
times as much of it.

Start the Opus seats as native `Agent` subagents with `model: "opus"` and
`run_in_background: false`. Start the Sol seat read-only:

- In T3 Code: `delegate_task` on the `codex` provider, model `gpt-6.1-sol`,
  `reasoningEffort: high`.
- Elsewhere: `codex exec -s read-only -m gpt-6.1-sol -c model_reasoning_effort=high -o review.md - < prompt.md`,
  wrapped in `timeout 1800`. Put the prompt on stdin.

Send all three in one message so they run in parallel. If your instructions
cap concurrent subagents, run them in waves under the cap. If Codex is out of
usage or unavailable, say so and run that round with two seats, or ask the
user. Do not quietly swap in an Opus Correctness seat. The same rule applies
to any named member who is unavailable.

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
| Sol · Correctness | 7/10 | 9/10 |
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
