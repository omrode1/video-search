# High-Level Design

Status: **proposed** (from kickoff, 2026-11-02). Items marked **[D-x]** are open decisions; see `docs/DECISIONS.md`.

## 1. Goals
- Natural-language search over long videos, returning timestamped segments.
- Handle object queries ("red truck"), action queries ("reversing"), spoken-content queries ("where is backprop explained"), and multi-condition queries.
- Measurable quality: Recall@K and temporal IoU on our own query set, plus an ablation across retrievers.
- Efficient indexing: avoid wasting compute on static footage.

## 2. Non-goals (for now)
- Multi-user auth, production-grade deployment, face recognition.

## 3. Components

```
                 ┌────────────────────── INDEXING PIPELINE ───────────────────────┐
 Video files ──► │ Ingest ─► Segmenter ─► Frame Sampler ─┬─► Visual Embedder ──────┤
                 │   │         (scene/      (motion-gated)├─► Captioner (VLM) ─────┤
                 │   │          fixed)                     │      └─► Text Embedder┤
                 │   └─► Audio Extract ─► ASR (Whisper) ───┴─► Text Embedder ──────┤
                 └────────────────────────────────────┬───────────────────────────┘
                                                      ▼
                                        Index Store (vectors + metadata
                                        + keyword index), keyed by segment
                                                      ▲
                 ┌──────────────────────── QUERY PIPELINE ────────────────────────┐
 User query ───► │ Query Encoder ─► Retrievers (visual | caption | keyword/BM25)  │
                 │      ─► Fusion (RRF) ─► Optional VLM Re-ranker ─► Results      │
                 └───────────────────────────────────┬────────────────────────────┘
                                                     ▼
                                    API ─► UI (search box + video player
                                            that seeks to the timestamp)

 Evaluation Harness: query set with ground-truth timestamps ─► runs any retriever
 config ─► Recall@1/5/10, temporal IoU, latency ─► eval/RESULTS.md
```

1. **Ingest:** registers a video, computes a content hash, extracts metadata (duration, FPS, resolution).
2. **Segmenter:** splits the video into time segments. **[D-1: scene detection vs fixed windows vs both, by domain]**
3. **Frame Sampler:** picks representative frames per segment; motion gating skips near-duplicate frames. **[D-2: sampling rate and motion threshold strategy]**
4. **Visual Embedder:** image embeddings per frame, pooled per segment. **[D-3: SigLIP 2 vs OpenCLIP, model size; pooling method]**
5. **Captioner:** VLM describes each segment from a few frames, focusing on actions and relationships. **[D-4: which VLM, local vs API, frames per caption, prompt design]**
6. **ASR:** Whisper transcripts aligned to segments (skipped for silent footage). **[D-5: Whisper variant]**
7. **Text Embedder:** embeds captions and transcripts. **[D-6: model choice]**
8. **Index Store:** vectors, metadata, and keyword index. **[D-7: FAISS vs LanceDB vs Qdrant; where BM25 lives]**
9. **Retrievers and Fusion:** each retriever returns ranked segments; reciprocal rank fusion merges them. **[D-8: fusion method and weights]**
10. **Re-ranker (optional):** VLM checks the top-N against the query. **[D-9: include it or not, N, cost budget]**
11. **API and UI.** **[D-10: FastAPI + Gradio/Streamlit vs FastAPI + small React app]**
12. **Evaluation Harness:** runs configurations and writes metrics. **[D-11: query set design and size]**

## 4. Cross-cutting concerns
- **Caching:** every expensive stage caches outputs keyed by `(video_hash, stage, model_id, config_hash)`, so changing one model doesn't recompute everything.
- **Config-driven:** models, thresholds, and retriever toggles live in config files, so ablations are just config changes.
- **Reproducibility:** pinned dependencies, seeded runs, eval results tied to config hashes.
- **Hardware awareness:** must run on the user's hardware. **[D-12]** Observed: RTX 3050 Laptop 6 GB VRAM, 15 GB RAM, 20 CPU threads.

## 5. Project-level decisions (M0)
- **[D-0a] ✅** Target domain: **CCTV primary; 1–2 lecture videos added in M3** for speech queries.
- **[D-0b] ✅** Compute: **laptop GPU only** (RTX 3050 6 GB) for now; device configurable so a cloud GPU can be added later.
- **[D-0c] ✅** **uv + Python 3.12** (`uv.lock`, `.python-version`).
- **[D-12] ✅** Budget: ~40 GB disk, ≤5 GB VRAM with one model loaded at a time, ≤6 GB RAM per process, indexing ≤1 h per hour of CCTV, ~5–10 h of test footage.
