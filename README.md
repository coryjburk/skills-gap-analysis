# Eccles MBA Career OS — Evidence-Backed Skills Engine

**Full-Time MBA Program · David Eccles School of Business · University of Utah**

### ▶ For users — start here!

> 1. **Open the tool:** [**Launch the Skills Engine →**]([https://coryjburk.github.io/skills-gap-analysis/]
> 2. **Rate yourself,** then add your resume (LinkedIn, GitHub, and certifications make the read sharper).
> 3. **Export your report** and bring it to your coach — then re-run every 6 weeks to watch your scores move.

That's the whole experience. Everything below is context and setup detail you don't need in order to use it.

---

**The core idea:** your self-rating is a *claim*. Your resume, LinkedIn, GitHub, and credentials are the *proof*. This tool scores both and shows you exactly where they line up — so you spend your coaching time on real gaps instead of guesswork.

It runs entirely in the browser as a single HTML file, hosted for free on GitHub Pages. AI scoring is powered by Claude through a small proxy you control.

**🔗 Live tool:** [https://coryjburk.github.io/skills-gap-analysis/]

---

## Table of contents

- [What it does](#what-it-does)
- [Who it's for](#who-its-for)
- [How the tool works (student walkthrough)](#how-the-tool-works-student-walkthrough)
- [How the scoring works](#how-the-scoring-works)
- [Setup & deployment](#setup--deployment)
- [Configuration](#configuration)
- [Privacy & data](#privacy--data)
- [Cost](#cost)
- [Troubleshooting](#troubleshooting)
- [File structure](#file-structure)
- [Maintenance notes](#maintenance-notes)
- [Disclaimer](#disclaimer)

---

## What it does

The tool guides a student through five steps and produces three scores plus an exportable report:

- **Self-Reported score** — what the student claims across ~30 skills.
- **Evidence-Backed score** — what their actual documents demonstrate, scored independently by Claude.
- **Coach-Verified score** — evidence-backed, plus any skills a coach has personally signed off on.

The gap between the first two *is* the coaching conversation. The tool surfaces it skill by skill, flags overclaimed and undersold skills, and tracks real outcomes (applications, interviews, offers) over time so the score can eventually be validated against results.

## Who it's for

- **Students** self-assess, add their evidence, and export a report for their coach.
- **Career coaches** review the gaps, co-sign skills they can vouch for, and log outcomes.
- **Program staff** get a consistent, evidence-anchored picture across a cohort.

## How the tool works (student walkthrough)

1. **Setup** — Pick your target career track (Product, Finance, Consulting, etc.). This loads the right functional skills and calibrates the benchmarks. Optionally flag verifiable differentiators (veteran, founder, technical undergrad, and so on).
2. **Self-Assessment** — Rate yourself 1–4 on each skill. Be honest; inflated ratings just get flagged in the next step.
   - **1** — No exposure
   - **2** — Classroom / coursework only
   - **3** — Demonstrated on the job, with a result
   - **4** — Led or taught others
3. **Evidence** — Add your resume (required) plus any of: LinkedIn, GitHub, certifications, portfolio. Claude scores each source *independently, without seeing your self-ratings*, then blends them.
4. **Evidence-Backed Readiness** — See your three scores and a skill-by-skill breakdown: what your evidence shows, where you over- or under-claimed, and which skills your coach can verify.
5. **Outcomes & Export** — Log where you are in your search (applications, interviews, offers), then copy or print a full report to share with your coach.

Students are encouraged to **re-run every 6 weeks** and share each snapshot, building a timeline that pairs scores with real results.

## How the scoring works

**Competency categories and weights** (these sum to 100%):

| Category | Weight |
|---|---|
| Core Business Acumen | 30% |
| Leadership & Influence | 25% |
| Functional Skills (track-specific) | 25% |
| AI, Data & Modern Workforce | 10% |
| Career Readiness Signals | 10% |

**Evidence source trust weights.** Not every source is equally hard to fake, so each carries a different weight when scores are blended:

| Source | Trust weight | Notes |
|---|---|---|
| Resume | 1.00 | Primary, required |
| GitHub | 1.00 | Technical, data & AI skills only |
| Certifications | 1.00 | Strong evidence for the specific skills each credential covers |
| Portfolio | 0.90 | Work samples |
| LinkedIn | 0.85 | Corroboration; third-party recommendations weigh most |

**Differentiators** (veteran, founder, international experience, etc.) are deliberately kept **out of the competency score**. They're real, verifiable, market-valued assets — but being a veteran isn't proof you can build a financial model. They appear as a separate Differentiator Profile.

**Scoring is done by Claude** (the `claude-sonnet-4-6` model) against an MBA-hiring-standard rubric baked into the app. Scores are AI estimates, not guarantees — validation comes from tracking real outcomes over time.

## Setup & deployment

The tool is one file (`index.html`) hosted on GitHub Pages. AI scoring needs an API key, which **must never** be placed in the public HTML — so a small [Cloudflare Worker](worker.js) holds the key and relays requests to Claude.

### 1. Host the tool (GitHub Pages)

1. Put `index.html` in this repository.
2. Go to **Settings → Pages**, choose your branch (usually `main`) and root folder, and **Save**.
3. Your tool goes live at `https://<your-username>.github.io/<repo>/` within a minute or two.

### 2. Deploy the API proxy (Cloudflare Worker)

The scoring step will not work on a bare Pages site without this — Pages is fully public and can't safely hold an API key.

1. Create a free account at **dash.cloudflare.com**.
2. **Workers & Pages → Create → Create Worker → Deploy.**
3. **Edit code**, delete the default, paste in `worker.js`, and **Deploy**.
4. Confirm the `ALLOWED_ORIGINS` line in `worker.js` matches your GitHub Pages address (the `https://<username>.github.io` part only — no path).
5. **Settings → Variables and Secrets → Add**, choose type **Secret**, name it exactly `ANTHROPIC_API_KEY`, paste your key from **console.anthropic.com**, and **Deploy**.
6. Copy your Worker's URL (e.g. `https://eccles-claude-proxy.<subdomain>.workers.dev`).

### 3. Connect the tool to the proxy

In `index.html`, find this line inside the `callClaude` function:

```js
const res = await fetch("https://api.anthropic.com/v1/messages", {
```

Replace the URL with your Worker's URL:

```js
const res = await fetch("https://eccles-claude-proxy.<subdomain>.workers.dev", {
```

Commit, and Pages will republish. Scoring now works on the live site.

## Configuration

Most of what you'd want to adjust lives near the top of `index.html` in plain data structures — no framework knowledge required:

- **Career tracks** — the `FUNC` object; each track is a label plus a list of six skills.
- **Competency categories & weights** — the `CATS` array.
- **Skills** — the `SKILLS` object (categories) and per-track lists in `FUNC`.
- **Differentiators** — the `DIFFERENTIATORS` array.
- **Certification options** — the `CERT_OPTIONS` array.
- **Source trust weights** — `SOURCE_META`.
- **The scoring rubric** — the `RUBRIC` string, if you want to recalibrate how strict Claude is.

In `worker.js`:

- **`ALLOWED_ORIGINS`** — which website(s) may call the proxy. Add a custom domain here if you map one.
- **`MAX_TOKENS_CAP`** and **`ALLOWED_MODELS`** — guardrails that limit cost if the Worker URL ever leaks.

## Privacy & data

Please review this against your program's data-handling and FERPA obligations before rolling out widely, since the tool processes student resumes:

- **What's sent out:** to produce scores, the text/files a student enters (resume, LinkedIn, etc.) are sent to the Anthropic API for analysis, relayed through your Cloudflare Worker.
- **What's stored:** the app does **not** save student inputs or scores anywhere. The only thing persisted between sessions is the coach's name (kept locally in the browser for convenience). The exported report is the sole record — it exists only where the student saves it.
- **Refresh = gone:** reloading the page clears everything. Students should export before closing.

## Cost

Scoring runs on Anthropic **API credits** (from console.anthropic.com), which are billed per token and are **separate** from any Claude.ai subscription. A single evidence-scoring run is typically a fraction of a cent to a few cents. For a full cohort, add a Cloudflare rate-limiting rule in front of the Worker and keep an eye on the usage dashboard.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Scoring button fails on the live site | HTML still points at `api.anthropic.com` | Swap the `fetch` URL to your Worker URL |
| "Origin not allowed" error | `ALLOWED_ORIGINS` doesn't match your Pages address | Update it in `worker.js`, redeploy |
| "Server is missing ANTHROPIC_API_KEY" | Secret not set, or misspelled | Re-add the secret named exactly `ANTHROPIC_API_KEY` |
| Authentication / 401 error | Invalid or revoked API key | Generate a new key, update the secret |
| Scoring fails with a billing error | No credit on the Anthropic account | Add credit in the Anthropic console |
| Scoring works in Claude preview but not on Pages | Expected — preview uses direct access | Only the deployed copy needs the Worker URL |

To see the exact error: right-click the page → **Inspect** → **Console** tab.

## File structure

```
.
├── index.html      # The entire tool (React + Babel via CDN, single file)
├── worker.js       # Cloudflare Worker proxy that holds the API key
├── wrangler.toml   # Optional — only needed if deploying the Worker via CLI
└── README.md       # This file
```

> Note: `worker.js` is deployed to Cloudflare (via dashboard or CLI), not served by GitHub Pages. It's kept in the repo for reference and version history.

## Maintenance notes

- **Updating the tool:** edit `index.html` and commit — Pages redeploys automatically. There's no build step; the browser compiles the JSX on load via Babel.
- **Rotating the API key:** create a new key in the Anthropic console, update the `ANTHROPIC_API_KEY` secret in Cloudflare, and delete the old key. No code change needed.
- **Performance:** in-browser Babel adds a second or two to page load. Fine for an internal tool; if it ever matters, migrate to a Vite build.

## Disclaimer

Evidence scores are AI-generated estimates, not guarantees of hiring outcomes. This tool is a coaching and self-reflection aid, not a credential or verification service. Validation comes from tracking real outcomes over time.

---

*Evidence-Backed Skills Engine · Full-Time MBA Program · David Eccles School of Business · University of Utah*
