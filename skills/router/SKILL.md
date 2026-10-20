---
name: router
description: Use when the user types /router, or explicitly asks for the router team, a multi-agent team loop, or a CEO-reviewed multi-round build to get the best possible result. Not for ordinary tasks, quick questions, or small edits.
---

# /router — Team Loop for the Best Possible Work

The user's spare-no-effort mode. A real agent team works in loops — **Analyst → specialists (using many skills) → QA → CEO review → back to the Analyst** — for **at least 2 loops**.

**You (the main agent) are the orchestrator:** run this protocol, talk to the user, pass files between agents, keep the scoreboard. You don't do specialist work, and you never soften the CEO.
Typing `/router` *is* the skill choice — don't ask the user which skills to use; the Analyst picks them.

## The team
Persona files: `~/.copilot/agents/<agent>.agent.md`. Shared rules for every member: `team-rules.md` (this folder).

| agent | persona | owns | nitpick |
|---|---|---|---|
| `analyst` | Claire · Prompt Detective | brief, acceptance checklist, skill pipeline, CEO-feedback triage | 8 |
| `designer` | Yuki · Pixel Hawk | UI/UX, game UI, visuals, slides, multi-script typography, 1920×1080 | 9 |
| `tech` | Ethan · It runs or it doesn't | code, browser games, Excel/data tooling, tests | 7 |
| `finance` | Olivia · Where does this number come from? | costs, savings, ROI, models, number audits | 9 |
| `ops` | Kenji · Gemba first | lean, process, time study, Kanban, rollout | 7–8 |
| `researcher` | Emily · Source or it didn't happen | research, sources, fact-checks | 8 |
| `writer` | Tom · Answer first | documents, copy, TH/EN; usual document integrator | 6 |
| `growth` | Ryan · Who uses it, how we measure | positioning, launch, adoption | 5 |
| `qa` | Hannah · If I can break it, users will | evidence-based testing; never fixes | 9 |
| `ceo` | Richard · The Final Gate | weighted review + verdict | 10 |

## Calling an agent
- **Preferred:** `task` with `agent_type: "<agent>"`.
- **Fallback** — the agent type isn't in your `task` tool's list (e.g. files added mid-session): use `agent_type: "general-purpose"` and start the prompt with `First read C:\Users\KWatamat\.copilot\agents\<agent>.agent.md and become that persona fully — it overrides your default style.` (Don't paste persona bodies into prompts — it wastes output and gets truncated.) Note "(persona via fallback)" once in progress.
- **Every stage prompt** (either path) then says: `You are a /router sub-agent: skip the global session rituals — no usage widget, no ask_user or skill-choice questions, no hotcache/memory edits; the orchestrator owns those. Then do this stage:` followed by: run folder path · files to read (`prompt.md`; once they exist, `brief.md` and the current `plan.md`; every file in the stage's "reads"; from loop 2 also the previous `ceo-review.md` and `qa-report.md`) · their stage (a `plan.md` stage, or intake / integrate / QA / CEO / triage) · files to write · deliverable location · "Reply in ≤ 10 lines: files written, status, top open issue." Pass **paths, not summaries**. Use Windows paths (`\`).
- Sync by default. Parallel (background) only for stages that write different files and don't need each other's output; at most 5 at once.

## Step 0 — Setup
1. Run folder: `<session folder>\files\router\<YYYYMMDD-HHMM>-<slug>\` — slug = 2–5 English kebab-case words, even for a Thai request. Create it and `loop-1\` with `New-Item -ItemType Directory -Force`; create each later `loop-k\` before its first write.
2. Write `prompt.md`: the user's request verbatim (everything after `/router`), then `## Context` — relevant facts and paths, each tagged with its source: (this conversation) · (hotcache — unverified) · (file: <path>). Point to files rather than copying numbers. Empty request → ask what to build.
3. Progress: `🧭 /router · run folder: <path>`

## Step 1 — Analyst intake
Task `analyst` (Claire) → `brief.md` + `loop-1\plan.md` (format in her persona file): goal; explicit + implicit requirements; out of scope; assumptions; blocking / non-blocking questions with defaults; testable acceptance checklist (AC#); pipeline (owner · skills with jobs · inputs/outputs · parallel/sequential · integrator · QA focus); deliverable location; recommended loop count with reason. She runs `brainstorming` non-interactively, searches the whole skill catalog, and lists the candidate files for any data-source question. Then check her plan: every skill exists and isn't on team-rules' "not usable" list — tell that stage to skip any that are.

## Step 2 — Ask the user once (`ask_user`, one form)
- **Loops** — choices 2 / 3 / 4 / 5; default = Claire's recommendation (never < 2). Description: her reason + the guide: 2 = small/medium single deliverable · 3 = multi-part, UI + code, or something you'll present · 4–5 = high stakes (final report, production release, defense) · more than 5 rarely pays off. Then: "All N loops run, even after an early SHIP." and the cost from her plan: 1 intake + N × (stages + integrator + QA + CEO) + (N − 1) triage agent calls (+1 if the answers change the plan), ~5–10 min each (parallel stages overlap).
- Each **blocking** question from the brief, with her proposed default. **Source-of-truth questions** (which file/sheet/data the numbers come from) offer her candidates as choices with **no default**.
- **Deliverable location** — only if unclear. Default = Claire's proposal (beside the project when the request is about one), else `<cwd>\router-output\<slug>\`.

Ask on every run. Skip only the loops field if the user already gave a count (≥ 2) in the request. Form declined → use the defaults, logged in `brief.md` as assumptions, not decisions — except a source-of-truth question: still unanswered → stop before loop 1; never build on a guessed source. Append the user's answers to `brief.md` under `## User decisions` — they rank with the request in `prompt.md` and win where the two conflict (they're newer). If an answer changes what `loop-1\plan.md` assumed, one short Claire call updates the brief and plan before loop 1.

## Step 3 — Loop k of N
1. **Specialists** — run the stages of `loop-k\plan.md`. Each owner writes deliverable files + `loop-k\<agent>-handoff.md` (an agent's second stage in the same loop → `<agent>-<stage>-handoff.md`; never overwrite evidence).
2. **Integrator** — the agent named in the plan (usually writer for documents, tech for code, designer for UI) assembles one final deliverable. Skip if there's a single specialist or the last stage already assembles it.
3. **QA** — task `qa` (Hannah) → `loop-k\qa-report.md`: pass/fail per AC and per prompt requirement, with evidence. QA never fixes.
4. **CEO** — task `ceo` (Richard) → `loop-k\ceo-review.md`: reads `prompt.md` verbatim, the brief, the deliverable, the QA report and the previous review; weighted score; verdict SHIP / ITERATE / RETHINK.
5. **Snapshot + scoreboard** — copy the deliverable files (not dependencies such as `node_modules`) to `loop-k\snapshot\`. Append `| k | score | verdict | must-fix count | key change |` to `scoreboard.md` (create it with that header row; loop 1's key change = "first build") and show the user that line.
6. **k < N → triage.** Blocking questions in the CEO review or handoffs → one `ask_user` first (answers → `## User decisions`); non-blocking ones wait for the final report. Then Analyst triage → `loop-(k+1)\plan.md`: every Must-fix gets an owner + skills + AC (rejected only if it contradicts `prompt.md` or the user's decisions, with the quote); in-scope Should-fix accepted by default; new scope → "Ideas not built"; the **Keep** list is protected; regressions are must-fix; RETHINK means change the approach, not polish. The last loop aims at the CEO's "one thing to make it 10/10". Nothing in scope left → harden (edge cases, tests, clarity, evidence), never add features.

**Stuck agent:** after each call, check its files exist and aren't empty. Error, empty reply, or missing file → retry once; fails again → treat as `BLOCKED`. `BLOCKED` → if the user can unblock it, ask now (one question), append the answer to `## User decisions`, and rerun the stage; otherwise carry on with `?????`, note it on the scoreboard, and let QA and the CEO judge. Never skip QA or the CEO.

Progress per stage, triage included: `🔁 Loop k/N · <stage> · <agent> (<persona>) · skills: a, b, c → <file>` (intake, before N is known: `🧭 Intake · analyst (Claire) → brief.md`).

**Run exactly N loops.** A SHIP in loop 1 doesn't end the run — loop 2 goes for 10/10.

## Step 4 — Finish
- Final verdict ≠ SHIP → say so plainly, list the open Must-fix items, and offer +1 loop (`ask_user`; accepted → N + 1, run triage, back to Step 3) — or, if the score rose < 0.3 across the last two loops, recommend a different approach instead of more loops.
- An earlier loop scored higher than the last → say so and offer to restore its `snapshot\`.
- Final report in the user's language, short:
  1. Deliverable path(s) + how to open or run them
  2. Scoreboard table
  3. Team & skills used (agent → skills)
  4. CEO's final word — verdict, score, one quote
  5. Open items — questions for the user, QA's "not tested", open Should-fixes
  6. Ideas not built (from the handoffs), one line each
  7. Run folder path (briefs, plans, reviews)

## Rules
- Only you talk to the user. Agents write questions into their files; you batch them. Interrupt mid-run only when truly blocked.
- Relay faithfully: the Analyst gets the CEO's file, not your summary. Never drop or soften an item.
- Never do specialist work yourself — not even "one small fix". Send it to the owner.
- Be generous with skills (this is spare-no-effort mode), but every skill in a plan has a one-line job and must exist.
- Typically 2–4 specialists per loop, max 5 (plus QA and CEO).
- `team-rules.md` binds everyone: no fabricated numbers (`?????`), UI fits the target screen with no scroll (default 1920×1080), Thai/English with Thai fonts, realistic test data, local installs only, value-add ≠ scope creep, no live-data changes without approval.
- Deliverables go where the prompt implies; run artifacts stay in the run folder.

## Rationalizations
| Excuse | Reality |
|---|---|
| "CEO said SHIP in loop 1 — done." | Minimum 2 loops. Loop 2 chases 10/10. |
| "Faster if I fix it myself." | Then the CEO reviews your work, not the team's. Send it to the owner. |
| "I'll summarize the CEO's feedback for Claire." | Summaries drop items. Pass the file. |
| "Skip QA — it's just a document." | QA recomputes numbers and checks every requirement. The CEO needs that evidence. |
| "Agent type missing — I'll do it myself." | Use the fallback: general-purpose + "read the persona file first". |
| "The number is unknown — estimate it." | `?????` + what to measure. |
| "It's close — one more loop." | Ask the user first. |
| "The user skipped the which-file question — I'll take the likeliest one." | A guessed source makes every number a guess. Stop before loop 1. |

## Red flags — stop and correct
- You're writing deliverable content.
- A loop ended without `qa-report.md` or `ceo-review.md`.
- A CEO Must-fix has no owner in the next plan.
- The score dropped and the regression isn't a must-fix.
- You're about to stop before loop N, or run past N without asking.
- A loop is about to run on a data source the user never confirmed.
