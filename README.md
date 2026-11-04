# tstack (Gambit fork)

Gambit's copy of [t3dotgg/tstack](https://github.com/t3dotgg/tstack), the agent
skills Theo uses every day, adapted for a team that runs on Claude only.

**This fork is for our own use. Never open a PR, issue or comment on
t3dotgg/tstack.** We pull from upstream; nothing goes back. The `upstream`
remote has its push URL disabled, `gh` defaults to `GambitPS/tstack`, and our
agent guard hook blocks any write to the upstream repo.

## What changed from upstream

- **One provider.** Opus 5.5 is the primary model for everything. Upstream's
  cross-provider reviews become fresh-context Opus reviews:
  `sol-loop` is now `opus-loop` ("sol loop" still triggers it), and `council`
  seats three Opus delegates with different lenses (correctness, player,
  house rules). That is lens diversity, not provider diversity, and the skills
  say so.
- **Images go to Codex.** New `codex-image` skill: Codex's image tool through
  the `codex` CLI, with the stdin prompt form that `-i` needs.
- **Repo rules win.** The skills point at each repo's `AGENTS.md` /
  `CLAUDE.md` for the Definition of Done, where tests run, how to merge, and
  whether an asked-for change ships to production after the merge
  (`full-send` step 7; `full-stop` never deploys). Nothing Gambit-specific
  lives here, so upstream changes merge cleanly.
- **No review bots.** Our repos have no Greptile or Macroscope, so
  `babysit-pr` waits with `pr-wait --until green` there.

## Skills

| Skill | Say | What it does |
|---|---|---|
| `full-send` | "full send" | Takes a change to a merged PR (and a deploy, where the repo says so) with nobody watching: proves it, reviews it, files it, babysits it, merges it. |
| `full-stop` | "full stop" | Same as `full-send`, but stops when the PR is ready for you to merge. |
| `council` | "council" | Three fresh-context Opus reviewers with different lenses; loops until they agree. |
| `opus-loop` | "opus loop", "sol loop" | Reviews a change with a fresh-context Opus reviewer in a loop until both agree. |
| `file-pr` | "file a PR" | Files a short PR that leads with the problem, not an inventory of changes. |
| `takeover` | "take over" | Finishes someone else's PR, branch, or agent thread, and keeps their credit. |
| `babysit-pr` | "babysit" | Watches a PR through CI and review, and fixes the real findings. |
| `modernize` | "modernize" | Rebases an old PR and decides if it should still exist. |
| `leaving-pr-comments` | (automatic) | Formats comments that an agent posts on your behalf. |
| `codex-image` | (automatic) | Generates images with Codex. |

Some skills call others:

- `full-send` uses `opus-loop` (or `council`), `file-pr`, and `babysit-pr`.
- `full-stop` uses `full-send`.
- `takeover` uses `modernize`, `file-pr`, `babysit-pr`, and `leaving-pr-comments`.
- `file-pr`, `babysit-pr`, and `modernize` use `leaving-pr-comments`.

## Install

Link the skills so a `git pull` updates them everywhere:

```bash
git clone https://github.com/GambitPS/tstack ~/code/tstack
for d in ~/code/tstack/skills/*/; do ln -sfn "$d" ~/.claude/skills/"$(basename "$d")"; done
```

Use the config dir your Claude Code reads (`$CLAUDE_CONFIG_DIR/skills` when
that is set).

## Sync from upstream

```bash
git fetch upstream
git merge upstream/main   # resolve, keep our changes above
git push origin main
```

Read upstream's diff before merging; a new skill may need the same
one-provider adaptation.

## Requirements

- An authenticated [GitHub CLI](https://cli.github.com) (`gh`).
- `python3` for `babysit-pr`'s `pr-wait` script.
- The `codex` CLI, logged in, for `codex-image`.

## License

[MIT](LICENSE), from upstream.
