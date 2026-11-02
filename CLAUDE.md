# CLAUDE.md — Project Memory

Natural-language video search: index long videos (CCTV, lectures, dashcam) so a text query returns timestamped segments. Combines visual embeddings, VLM captions, ASR transcripts, and keyword search, fused with RRF and optionally re-ranked. An eval harness backs every design choice with numbers.

## Current status
- **Milestone:** M0 — Setup (no model code)
- **Phase:** docs created; pre-implementation quiz pending; then decisions D-0a, D-0b, D-0c, D-12
- **Git:** `main` pushed to `origin` (`git@github.com:omrode1/video-search.git`). Repo-local config: author `omrode1 <omrode.34@gmail.com>`, `core.sshCommand` uses `~/.ssh/id_ed25519_personal`. `.claude/` is gitignored. Working branch: `m0-setup`.

## Working rules (condensed)
1. **Roles:** the user is the architect and decides; Claude is the engineer and teacher. Understanding matters more than speed.
2. **Decision gates:** any meaningful choice (architecture, data models, interfaces, libraries/models, storage, APIs, folder layout, eval method, perf tradeoffs, anything hard to reverse) → present: question (1 sentence), 2–4 options with pros/cons/cost (complexity, compute, learning value), recommendation + why, cost of choosing wrong + reversibility. **Wait for the answer.** No dependent code before then. Trivial choices: just mention them in the summary. If unsure, ask.
3. **Quizzes:**
   - *Pre-implementation:* short explanation → 3–5 questions (conceptual / design / prediction), no answers in the same message → grade each (correct/partly/incorrect), teach the gaps → if more than half are missed, offer an explanation or tiny experiment.
   - *Pre-merge:* walk through the diff → 2–3 questions on the actual code → merge only after the user says **"approve merge"**.
   - Log every quiz in `docs/LEARNING.md`.
4. **Git:** `main` stays clean. One branch per milestone/task (`m1-mvp-search`, ...). Small, descriptive commits. Never merge to `main` without "approve merge".
5. **Docs:** keep the files below accurate. If code and docs disagree, flag it and ask which is right.
6. **Session start:** read this file + `docs/ROADMAP.md`, then tell the user in 3–4 lines where we are and what's next.
7. **Task end:** summarize what was done, what was decided, what's open, what's next.
8. Explain new concepts the first time they appear (user knows Python + basic ML; learning retrieval, embeddings, VLMs).

## Maintained files
| File | Purpose |
|---|---|
| `CLAUDE.md` | This file: rules, current milestone, conventions, how to run |
| `docs/DECISIONS.md` | ADR-style decision log |
| `docs/HLD.md` | High-level design |
| `docs/LLD.md` | Low-level design |
| `docs/ROADMAP.md` | Milestones + status + open questions |
| `docs/LEARNING.md` | Quiz log, concepts learned / to revisit, reading links |
| `eval/RESULTS.md` | Eval runs: date, config, metrics, notes (created in M5 or earlier) |

## Known environment (observed 2026-11-02, not yet a decision)
- GPU: NVIDIA RTX 3050 Laptop, 6 GB VRAM (driver 590.48.01)
- CPU: 20 threads; RAM: 15 GB
- Python 3.12.3 system; `uv` 0.12.7 installed

## Accepted decisions (summary; details in docs/DECISIONS.md)
- D-0a: CCTV primary; 1–2 lecture videos added in M3
- D-0b: laptop GPU only for now; device configurable
- D-0c: uv + Python 3.12
- D-12: ~40 GB disk, ≤5 GB VRAM one model at a time, ≤6 GB RAM, indexing ≤1 h per hour of CCTV

## Conventions
- Package manager: **uv**, Python **3.12** (D-0c). Layout, lint/format, test runner: TBD in M0.

## How to run
- Nothing to run yet.
