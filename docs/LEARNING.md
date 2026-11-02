# Learning Log

## Concepts learned
_(filled in as we go)_

## Concepts to revisit
_(filled in from quiz gaps)_

## Links worth reading
_(added as concepts come up)_

---

# Quiz log

<!-- Template
## <date> — <milestone/task> — pre-implementation | pre-merge
| # | Question | My answer (summary) | Grade | Notes |
|---|---|---|---|---|
| 1 | ... | ... | correct/partly/incorrect | ... |

Concepts to revisit:
- ...
-->

## 2026-11-02 — M0 — pre-implementation
| # | Question | My answer (summary) | Grade | Notes |
|---|---|---|---|---|
| 1 | Content hash vs file path as the video's identity; cost of hashing | Paths differ across systems; hash depends on content ("what the video is about") | partly | Portability point correct. A hash fingerprints the *bytes*, not the meaning: re-encoding the same footage gives a new hash. Missed: moving/renaming a file breaks path-keyed caches, and duplicate copies get indexed twice. Missed cost: hashing reads every byte (a 10 h file can be 10–40 GB, which takes minutes of disk I/O). |
| 2 | Drop `config_hash` from cache key, change 1 FPS → 0.5 FPS | Didn't know | incorrect | Stale cache hit: the cache returns the old 1 FPS frames under a key that still matches. No error, plausible results, and eval numbers get credited to the wrong config. Silent correctness bugs are the worst kind. |
| 3 | Why an eval harness instead of eyeballing | Prevents regressions as features are added | partly | Regression detection is right. Missed: (a) to *compare* alternatives (SigLIP vs CLIP, RRF k) you need a number, not a feeling; (b) eyeballing a few queries is a small, biased sample. Note: we don't train, so nothing "forgets"; regressions come from pipeline/config changes. |
| 4 | Slowest indexing stage on 6 GB VRAM | Image embedding (counted all frames) | incorrect | Arithmetic: 25 fps = 1,500 frames/min = 90,000/h. But we sample (~1 FPS = 3,600 frames), and a SigLIP-base pass on that takes well under a minute on a 3050. VLM captioning dominates: a few hundred to ~700 segments × ~1–3 s each ≈ 10–35 min, roughly 10–50× the other stages (estimates; we'll measure in M3). Generative models are slow because they produce tokens one at a time; embedders do one forward pass. |
| 5 | Situation where skipping config hash + dependency versions hurts | Restated why eval matters | incorrect | Answered a different question. Example: Recall@5 was 0.72 three weeks ago, now 0.64. Was it the motion threshold you changed, a library update to image preprocessing, or a new model checkpoint? Without the config hash and pinned versions you can't reproduce 0.72 or find the cause. |

Score: 0 correct, 2 partly, 3 incorrect.

Concepts to revisit:
- Hashes fingerprint bytes, not meaning; cost of full-file hashing
- Cache keys must include everything that affects the output (stale cache hits)
- Eval as a comparison tool, not only a regression guard
- Throughput: generative (token-by-token) vs single-pass embedding models; frames sampled vs frames in the file
- Reproducibility: recording config and library versions with every result

### Follow-up experiment (2026-11-02): stale cache + hashing cost
Script: throwaway, not in the repo.
- **Stale cache:** with key `(hash, stage, model)`, switching 1 FPS → 0.5 FPS gave a cache HIT that returned 10 frames instead of 5, with no error. Adding `config_hash` to the key turned it into a MISS that returned the correct 5.
- **Hashing a 4 GB file:** full SHA-256 took 4.22 s (~0.95 GB/s, file already in RAM, so this is CPU-bound). Partial hash (size + first/last 4 MB) took 0.01 s. Takeaway: a 40 GB file needs ≥40 s even when it's already in RAM, and longer from a cold disk. Partial hashing is fast but can, rarely, treat two different files as the same; that tradeoff gets decided when we build ingest in M1.

## 2026-11-02 — M0 — pre-merge
| # | Question | My answer (summary) | Grade | Notes |
|---|---|---|---|---|
| 1 | Untyped `def iou(a, b)` in `vsearch/eval/metrics.py`: does mypy complain? | Yes, it will trigger (called it "UV") | incorrect | The tool is **mypy**, not uv (uv only manages Python and packages). Strict flags apply only to `vsearch.core.*`, so mypy reports *Success*: unannotated parameters are silently `Any`. Demo: the same file under `core/` raised `no-untyped-def`. Fix: add `"vsearch.eval.metrics"` to the override `module` list (as the pyproject comment says). |
| 2 | Commit with an unused `import os`: what does pre-commit do? | Commit is auto formatted/linted; unused import gets caught | partly | Caught: yes. Missed the mechanics: `ruff check --fix` *removes* the import, which modifies the file, so the hook reports **Failed** and the **commit is aborted**. Your index still has the old version; the fixed version is in the working tree (`AM`). You `git add` and commit again. Demo confirmed: log unchanged after the attempt. |
| 3 | Why is `data/` ignored but `uv.lock` committed? Swap consequences? | data has files not needed for config; lock changes with config so must be committed | partly | Direction right, reasons missing. `data/`: large (GBs; GitHub rejects files >100 MB), possibly private (CCTV), and regenerable (cache). Once committed, it stays in history forever even if deleted later. `uv.lock`: exact versions of every package, for reproducibility (quiz M0-pre Q5). It is generated by `uv lock`/`uv sync`, never hand-edited. Swapped: repo bloat + privacy leak, and every `uv sync` resolves fresh versions → silent drift. |

Score: 0 correct, 2 partly, 1 incorrect.

Concepts to revisit:
- Tool roles: **uv** = Python + packages + lockfile; **ruff** = lint + format; **mypy** = type checking; **pytest** = tests; **pre-commit** = runs checks on `git commit`
- mypy per-module strictness; unannotated = `Any` = unchecked
- pre-commit "files were modified" → commit aborted → re-add
- What belongs in git: source + lockfiles, not data/derived artifacts
