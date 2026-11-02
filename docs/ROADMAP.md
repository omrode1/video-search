# Roadmap

Each milestone: pre-implementation quiz → decision gates → build on a branch → pre-merge quiz → "approve merge" → update docs.

| Milestone | Scope | Branch | Status |
|---|---|---|---|
| **M0 — Setup** | Docs + `CLAUDE.md`, git init, decide D-0a/D-0b/D-0c/D-12, agree repo layout, environment + tooling. No model code. | `m0-setup` | **in progress** |
| M1 — MVP search | Fixed 1 FPS sampling, one image embedder, vector index (D-7), CLI search over one video, minimal UI that seeks to the timestamp. | `m1-mvp-search` | todo |
| M2 — Segmentation & motion gating | Segmenters, motion-gated sampling, segment pooling, caching layer. | `m2-segmentation` | todo |
| M3 — Captions & transcripts | VLM captioner, Whisper, text embeddings, keyword index. | `m3-captions-asr` | todo |
| M4 — Hybrid retrieval | Multiple retrievers, RRF, adjacent-segment merging, optional re-ranker. | `m4-hybrid` | todo |
| M5 — Evaluation | Query set, eval harness, ablations, `eval/RESULTS.md`. | `m5-eval` | todo |
| M6 — Polish | API cleanup, UI, README with results table + architecture diagram, demo script. | `m6-polish` | todo |
| Stretch | Temporal queries, live RTSP with saved-query alerts, edge-side embedding, caption-on-change cost optimization. | — | to discuss |

## M0 checklist
- [x] Create `CLAUDE.md`, `docs/DECISIONS.md`, `docs/HLD.md`, `docs/LLD.md`, `docs/ROADMAP.md`, `docs/LEARNING.md`
- [ ] M0 pre-implementation quiz
- [ ] D-0a target domain
- [ ] D-0b compute
- [ ] D-0c Python version + package manager
- [ ] D-12 hardware budget
- [ ] Agree repo layout
- [x] `git init`, `.gitignore`, first commit on `main` (pushed)
- [ ] Environment + tooling (lint/format/test)
- [ ] M0 pre-merge quiz → "approve merge"

## Open questions
- Should a tiny eval set exist from M1 onward (rather than waiting for M5), so every milestone has a number to beat? (raise before M1)
- Where do test/sample videos come from, and what are their licenses?
