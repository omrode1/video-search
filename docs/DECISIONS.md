# Decision Log (ADR style)

Every gated decision gets an entry. Statuses: `proposed` | `accepted` | `superseded by D-<m>`.

## Index
| ID | Title | Milestone | Status |
|---|---|---|---|
| D-0a | Target footage domain for v1 | M0 | accepted |
| D-0b | Compute environment | M0 | accepted |
| D-0c | Python version and package manager | M0 | accepted |
| D-12 | Hardware budget / what compute is available | M0 | accepted |
| D-LAYOUT | Repository layout | M0 | open |
| D-1 | Segmentation strategy | M2 | open |
| D-2 | Sampling rate and motion threshold | M1/M2 | open |
| D-3 | Visual embedding model, size, pooling | M1 | open |
| D-4 | Captioning VLM, local vs API, frames, prompt | M3 | open |
| D-5 | Whisper variant | M3 | open |
| D-6 | Text embedding model | M3 | open |
| D-7 | Vector store + where BM25 lives | M1 | open |
| D-8 | Fusion method and weights | M4 | open |
| D-9 | Re-ranker: include?, N, cost budget | M4 | open |
| D-10 | API + UI stack | M1/M6 | open |
| D-11 | Eval query set design and size | M5 | open |

---

## D-12: Hardware budget
- Date: 2026-11-02
- Milestone: M0
- Status: accepted
- Context: Budgets set what "good enough" means for model sizes, footage volume, and frame-skipping aggressiveness. Machine: RTX 3050 Laptop 6 GB VRAM, 15 GB RAM, 20 threads, ~90 GB free disk (shared HF cache already 16 GB).
- Options considered:
  1. Moderate — ~40 GB disk, ≤5 GB VRAM one model at a time, ≤6 GB RAM/process, indexing ≤1 h per hour of CCTV, ~5–10 h footage
  2. Tight — ~15 GB, ≤4 GB VRAM, ≤30 min/h, ~2 h footage; eval set too small to be trustworthy
  3. Generous — ~70 GB, ≤5.5 GB VRAM, no time limit, 20 h+; labeling is the real bottleneck, hides slow components
- Claude's recommendation: Option 1.
- Decision (by me): Option 1 — moderate.
- Rationale: Accepted the recommendation.
- Consequences: Pipeline runs stage by stage and frees VRAM between stages. Indexing-time budget (≤1 h per hour of CCTV) is a target for M2 motion gating. Data directory is configurable, never hardcoded. Budgets are numbers in docs/config, easy to revise.

## D-0c: Python version and package manager
- Date: 2026-11-02
- Milestone: M0
- Status: accepted
- Context: Reproducibility (pinned deps, quiz Q5) needs a lockfile; GPU libraries need a Python version with prebuilt wheels. Machine has Python 3.12.3, uv 0.12.7, ffmpeg 6.1.1, driver supporting CUDA 13.1.
- Options considered:
  1. uv + Python 3.12 — installed, fast, `uv.lock`, `.python-version`, PyTorch CUDA index configurable in pyproject / newer tool / low complexity
  2. Poetry + 3.12 — mature / slower, struggles with PyTorch CUDA builds / medium
  3. pip + venv + requirements.txt — familiar / no real lockfile, version drift / weak reproducibility
  4. conda / pixi — handles non-Python deps / heavy, conda+pip mixing issues / unneeded here
- Claude's recommendation: uv + Python 3.12.
- Decision (by me): Option 1 — uv + Python 3.12.
- Rationale: Accepted the recommendation.
- Consequences: `pyproject.toml` + `uv.lock` + `.python-version` committed; PyTorch CUDA index pinned in pyproject. Reversible: pyproject is standard (pip/Poetry can read it); only the lockfile would be regenerated.

## D-0b: Compute environment
- Date: 2026-11-02
- Milestone: M0
- Status: accepted
- Context: Model sizes, captioning throughput and cost all depend on where compute runs. Hardware: RTX 3050 Laptop 6 GB VRAM, 15 GB RAM (~11 GB in use), 20 threads, 14 GB free disk.
- Options considered:
  1. Laptop GPU only — free, private, fast iteration, forces efficiency / VLMs limited to ~2–4B quantized, thermals, disk / low complexity, $0
  2. Local dev + cloud GPU burst — larger VLMs possible / two environments, data sync, $ / medium complexity
  3. Colab — free T4 / ephemeral, poor fit for repo + CLI workflow
  4. CPU only — not viable for captioning
- Claude's recommendation: Option 1, built so Option 2 can be added later (device + models in config, caches keyed by content hash so they can move between machines).
- Decision (by me): Option 1 — laptop GPU only for now.
- Rationale: Accepted the recommendation.
- Consequences: Captioning VLM limited to small/quantized models (affects D-4); one big model loaded at a time. Code must not hardcode device or paths. Revisit if M3/M5 eval shows caption quality is the bottleneck. Open: is the `GDC-T4-12-50GB-953` host available for this project?

## D-0a: Target footage domain for v1
- Date: 2026-11-02
- Milestone: M0
- Status: accepted
- Context: The footage type decides which components get stressed, the eval query set, and per-domain defaults (segmenter, motion threshold, caption prompt). Needed before any labeling or default tuning.
- Options considered:
  1. CCTV — motion gating pays off, object/action queries, easy ground truth / no audio, low-res small objects, research-only datasets, large files / medium complexity, high compute, high visual learning value
  2. Lectures — open licensed sources, speech queries / visually boring, reduces to transcript search / low cost, text-side learning only
  3. Dashcam — rich object/action queries / constant motion defeats gating, large research-licensed datasets / high complexity
  4. Mixed — exercises everything / three sets of defaults, thin noisy per-domain eval / high complexity
- Claude's recommendation: CCTV primary + 1–2 lecture videos in M3 so ASR/BM25 are exercised.
- Decision (by me): CCTV primary; add lectures in M3.
- Rationale: Accepted the recommendation (no further reasoning given).
- Consequences: Defaults (motion threshold, segmenter, caption prompt) tuned for static cameras. M1–M2 eval queries are object/action on CCTV; speech queries arrive with lectures in M3, and the eval set gets a domain field. Dashcam is out of scope for v1. Reversing costs mainly the labeled query set and tuned defaults, not code.

<!-- Template
## D-<n>: <title>
- Date:
- Milestone:
- Status: proposed | accepted | superseded by D-<m>
- Context: <why this decision is needed now>
- Options considered:
  1. <option> — pros / cons / cost
  2. <option> — pros / cons / cost
- Claude's recommendation: <option + reason>
- Decision (by me): <chosen option>
- Rationale: <my reasoning, in my words if I gave it>
- Consequences: <what this enables or rules out; how to reverse>
-->
