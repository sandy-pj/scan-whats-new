# scan-whats-new

A [Claude Code](https://claude.com/claude-code) skill that scans the open-source AI landscape on demand and delivers a self-contained HTML report.

One run sweeps several independent angles:

- **GitHub search API** — repos created in the last ~45 days across AI topics, ranked by stars/day (fast movers).
- **Snapshot deltas** — a local `seen-projects.json` records stars per repo per scan, so repeat runs compute real growth velocity (Δ stars since last scan) and never re-report the same find.
- **GitHub trending** (weekly) — catches older repos having a moment that created-date search misses.
- **Hacker News** — "Show HN" launches as a traction signal independent of stars.
- **Hugging Face trending** (optional) — notable open-weight model releases.

The output is a single HTML file (`scans/<date>-whats-new.html` — self-contained, cleanly styled, metrics as tables; no fixed template) with a TL;DR, **Fast movers** (with numbers — stars, stars/day, deltas), **Interesting finds** (why each matters), a one-liner **Radar**, and a methods footer. Every claim of "fast-growing" gets a number; every project gets a link.

## Install

Copy the skill into a project's `.claude/skills/` directory:

```bash
mkdir -p .claude/skills/scan-whats-new
curl -fsSL https://raw.githubusercontent.com/sandy-pj/scan-whats-new/main/SKILL.md \
  -o .claude/skills/scan-whats-new/SKILL.md
```

Then in Claude Code, run:

```
/scan-whats-new
```

An authenticated [`gh` CLI](https://cli.github.com/) is recommended (higher GitHub API rate limits) but not required.

## Layout assumptions (adapt freely)

The skill was extracted from a discovery-loop workspace and reads/writes a few conventional paths — edit `SKILL.md` to match your project, or just let it create them:

| Path | Role |
|---|---|
| `scans/<date>-whats-new.html` | the deliverable report |
| `projects/scans/seen-projects.json` | star snapshots for dedup + growth deltas |
| `goal.md`, `projects/active/`, `log.md` | optional workspace context / run log — harmless to omit |

## License

MIT
