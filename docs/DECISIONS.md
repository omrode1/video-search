# Decision Log (ADR style)

Every gated decision gets an entry. Statuses: `proposed` | `accepted` | `superseded by D-<m>`.

## Index
| ID | Title | Milestone | Status |
|---|---|---|---|
| D-0a | Target footage domain for v1 | M0 | open |
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
