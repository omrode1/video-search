# Decision Log (ADR style)

Every gated decision gets an entry. Statuses: `proposed` | `accepted` | `superseded by D-<m>`.

## Index
| ID | Title | Milestone | Status |
|---|---|---|---|
| D-0a | Target footage domain for v1 | M0 | accepted |
| D-0b | Compute environment | M0 | open |
| D-0c | Python version and package manager | M0 | open |
| D-12 | Hardware budget / what compute is available | M0 | open |
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
