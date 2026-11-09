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
usage or unavailable, the Spark seat below takes Correctness for that round;
say so. Do not quietly swap in an Opus Correctness seat. The same rule applies
to any named member who is unavailable.

## Spark seat (Meta Muse Spark 1.3, light use)

Muse Spark 1.3 runs on the Muse Code High Usage subscription. One review
uses about 3-4% of its weekly limit and about 10% of its short window, so it
sits in two places only:

- **Sign-off:** once the three seats above are all satisfied, run Spark
  alone with the Correctness lens on the final diff. That is one review per
  council. Skip it when the diff is only docs, data or map exports. Run it
  after the other seats, never in their parallel wave. If it
  finds a blocker (wrong behaviour, a compile or test failure, data loss),
  fix it and re-run Spark only, once. Note its should-fix and nit findings
  in the report, but do not act on them or start new rounds for them.
- **Sol backup:** when Codex is out of usage or unavailable, Spark takes
  the Correctness seat for every round Sol misses.

Run it read-only:

- In T3 Code: `delegate_task` on the `cliproxy` provider, model
  `muse-spark-1.3`. Each T3 environment needs `muse-spark-1.3` in the CLI
  Proxy provider's custom models; if T3 says the model is not advertised,
  say so and skip the seat.
- Elsewhere: `claude -p --model muse-spark-1.3 --permission-mode plan` with
  `CLAUDE_CONFIG_DIR` pointed at the CLI proxy Claude home, wrapped in
  `timeout 1800`. Put the prompt on stdin.

Keep its brief tight: give it the diff command, the intent, the repo rules
and the few files it may read, and tell it not to search beyond them. Free
exploration is what makes its reviews expensive.

If the proxy refuses Spark (for example, the usage limit was hit), say so and
finish without it. Do not swap in another model. Use Muse Wonder only when
the user asks: it costs about twice as much per review.

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
| Spark · Sign-off | — | 9/10 |
```

Write `—` for a round a delegate did not join. When Spark stands in for Sol,
label its row `Spark · Correctness (Sol unavailable)`. With Spark seated,
say "three providers" rather than two.

If a delegate gives no score, write `no score`. Do not invent one.
If the PR is open, edit the description. Do not post a comment for this.
If there is no PR, put the same results in the final reply only.

If you agree with their recommendations, make the changes and request a fresh
review after. Continue this process until all delegates are satisfied with the
outcome.

Do not let repeated reviews result in scope creep. It is your job to fight to
keep scope limited.
