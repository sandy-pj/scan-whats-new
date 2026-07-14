---
name: scan-whats-new
description: On-demand scan of the open-source AI landscape. Sweeps GitHub (trending + search API), Hacker News, and Hugging Face for notable open-source AI projects — both *interesting* (novel capability, new category, strong idea) and *fast-growing* (star velocity, sudden traction) — dedupes against previous scans, and delivers an HTML report at scans/<date>-whats-new.html. Trigger when the user says "scan what's new", "/scan-whats-new", "what's new in open-source AI", "scan for trending AI projects", or "run the what's-new scan".
---

# Scan-whats-new — one on-demand sweep

You are running a single pass of an open-source AI landscape scan. The goal is
signal, not volume: surface projects worth the user's attention — either
because they're **interesting** (new capability, new category, clever idea) or
because they're **growing fast** (traction the market is voting for). State
lives in files; read first, act, then write state back.

## 1. Load state
- Read `projects/scans/seen-projects.json` if it exists → prior snapshots:
  `{ "owner/repo": { "first_seen": "YYYY-MM-DD", "last_scan": "YYYY-MM-DD",
  "stars": N, "reported": true|false } }`. If missing, this is the first run —
  create it at the end (and `mkdir` `projects/scans/` / `scans/` as needed).
- Skim `goal.md` and `projects/active/` names so you can flag (lightly, never
  force) when a find is relevant to the business hunt or an active project.

## 2. Scan
Sweep multiple angles — each is blind to what the others catch. Prefer `gh api`
(authenticated, higher rate limits) over raw curl when `gh` is available.

- **Fast movers (new + hot):** GitHub search API for repos created in the last
  ~30–45 days, sorted by stars, across AI topics/queries, e.g.
  `gh api "search/repositories?q=topic:llm+created:>YYYY-MM-DD&sort=stars&order=desc&per_page=20"`
  and again for `topic:ai-agents`, `topic:rag`, `topic:machine-learning`,
  plus a keyword pass (`q=llm OR "ai agent" created:>...`). Compute
  **stars/day since creation** — that's the growth metric for new repos.
- **Accelerating incumbents:** for every repo already in `seen-projects.json`,
  fetch current stars and compute **Δ stars since last scan** (stars/day over
  the gap). A known repo that suddenly doubles its pace is a finding too.
- **Trending pages:** WebFetch `https://github.com/trending?since=weekly` (and
  `/trending/python?since=weekly`) — catches older repos having a moment that
  created-date search misses.
- **Launch chatter:** Hacker News Algolia API —
  `https://hn.algolia.com/api/v1/search_by_date?query=%22Show%20HN%22&tags=story&numericFilters=points>50`
  and a query pass for open-source AI launches this week. HN points are an
  independent traction signal from stars.
- **Models/spaces (optional):** Hugging Face trending
  (`https://huggingface.co/api/models?sort=likes7d&limit=20` or WebFetch the
  trending page) for notable open-weight releases.

Rate-limit note: unauthenticated GitHub search allows ~10 requests/min — space
the calls or use `gh api`. Keep the whole sweep to a reasonable budget
(~15–25 API calls).

## 3. Filter & rank
- Drop: repos already reported in a previous scan **unless** they re-qualify on
  new growth (Δ since last scan is the story); awesome-lists, tutorials,
  paper-only dumps, and obvious star-farming (all-time-high stars but dead
  commit activity); anything not actually AI/ML.
- **Interesting** = does something new or does an old thing in a clearly better
  way; would make a builder say "oh, that's now possible/easy". Judge by README
  + what people say about it, not stars alone.
- **Fast-growing** = top stars/day for its age bracket, or sharp Δ since last
  scan. Sanity-check with recent commit/release activity (is it alive?).
- Keep roughly **5–10 fast movers** and **3–6 interesting finds** (overlap is
  fine — a project can headline both). Quality over volume.

## 4. Render the HTML report
Write the report as a **self-contained, smartly styled HTML** file at
`scans/<YYYY-MM-DD>-whats-new.html` (append `-2`, `-3`… if the file already
exists — never overwrite an earlier same-day report). No fixed template — use
your judgment — but the bar is: clean modern typography (system font stack,
readable measure, inline CSS only), light *and* dark mode via
`prefers-color-scheme`, and **metrics presented as real tables**, not inline
code fragments: full-width tables in a rounded bordered wrapper with
`overflow-x:auto`, uppercase muted column headers, numeric columns
right-aligned with `font-variant-numeric:tabular-nums`, growth/velocity values
highlighted (e.g. a subtle green badge), and row hover. Tables hold the short
enumerable facts (project, what it is, stars, velocity, age); the *why* stays
in prose beneath them. Structure the content as:

- **TL;DR** — 3–5 sentences: the headline projects and any theme of the week
  (e.g. "agent memory is having a moment").
- **Fast movers** — one `<h3>` + short block per project:
  `<a>` to the repo, one-line pitch, metrics line in `<code>`
  (`★ 4,120 · +310/day · created 12d ago` or `Δ +2,400 since last scan`),
  then 1–2 sentences: what it is and why it's moving.
- **Interesting finds** — same shape, but the paragraph explains *why it's
  interesting* (what it makes newly possible), plus, when genuinely apt, one
  italic line: *Angle:* how it could matter for the business hunt or an active
  project. Omit the line rather than stretch.
- **Radar** — `<ul>` of one-liners: worth knowing, not worth a section.
- **Method** — one closing paragraph: sources swept, date window,
  scan count (e.g. "GitHub search + trending, HN, HF · window: last 45 days ·
  92 repos scanned, 11 reported").

Every project mentioned anywhere in the report gets a link. Every claim of
"fast-growing" gets a number.

## 5. Write state back
- Upsert every repo *evaluated* (not just reported) into
  `projects/scans/seen-projects.json` with today's stars, `last_scan`, and
  `reported` flag — this is what makes the next run's Δ computation and dedup
  work.
- Append to `log.md`:
  `## <date> · scan-whats-new · N repos scanned, M reported → scans/<date>-whats-new.html`
  with a one-line list of the headline projects.

## 6. Report
In chat: name the top 3–5 projects with one line each, note any theme, and give
the path to the HTML file. If a find is a plausible business opportunity, say
so and suggest handing it to `/project-scout` for scoping — but the scan itself
proposes nothing; it only informs.
