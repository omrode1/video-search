# Low-Level Design

Status: **draft for discussion** (from kickoff, 2026-11-02). Nothing here is accepted until it has a decision entry or the user confirms it.

## 1. Repository layout (proposed — confirm in M0)

```
video-search/
├── CLAUDE.md
├── README.md
├── pyproject.toml
├── configs/
│   ├── default.yaml
│   └── ablations/          # one file per retriever configuration
├── data/
│   ├── raw/                # input videos (gitignored)
│   └── cache/              # stage outputs (gitignored)
├── src/vsearch/
│   ├── ingest/             # video registration, hashing, metadata
│   ├── segment/            # segmenters
│   ├── sample/             # frame sampling, motion gating
│   ├── embed/              # visual + text embedders
│   ├── caption/            # VLM captioner
│   ├── asr/                # Whisper wrapper
│   ├── index/              # vector store + keyword index adapters
│   ├── retrieve/           # retrievers, fusion, re-ranker
│   ├── pipeline/           # orchestration + caching
│   ├── api/                # HTTP API
│   └── ui/                 # demo UI
├── eval/
│   ├── queries/            # query set with ground truth
│   ├── run_eval.py
│   └── RESULTS.md
├── tests/
└── docs/
```

## 2. Core data models (draft)

```python
Video:        id, path, content_hash, duration_s, fps, width, height, domain
Segment:      id, video_id, start_s, end_s, keyframe_paths, motion_score
VisualEmb:    segment_id, model_id, vector, pooling
Caption:      segment_id, model_id, prompt_version, text
CaptionEmb:   segment_id, model_id, vector
TranscriptChunk: segment_id, start_s, end_s, text, asr_model_id
SearchHit:    segment_id, video_id, start_s, end_s, score, per_retriever_ranks, thumbnail
EvalQuery:    id, text, category (object|action|speech|multi), video_id, gt_start_s, gt_end_s
```

## 3. Interfaces (draft protocols)

Every swappable component sits behind a small interface so ablations and model swaps are easy.

```python
class Segmenter(Protocol):
    def segment(self, video: Video) -> list[Segment]: ...

class FrameSampler(Protocol):
    def sample(self, video: Video, segment: Segment) -> list[Frame]: ...

class ImageEmbedder(Protocol):
    model_id: str
    def embed_images(self, frames: list[Frame]) -> np.ndarray: ...
    def embed_text(self, texts: list[str]) -> np.ndarray: ...   # same space, for queries

class TextEmbedder(Protocol):
    model_id: str
    def embed(self, texts: list[str]) -> np.ndarray: ...

class Captioner(Protocol):
    model_id: str
    def caption(self, frames: list[Frame], prompt_version: str) -> str: ...

class VectorStore(Protocol):
    def upsert(self, collection: str, ids: list[str], vectors: np.ndarray, metadata: list[dict]): ...
    def search(self, collection: str, vector: np.ndarray, k: int, filters: dict | None) -> list[tuple[str, float]]: ...

class Retriever(Protocol):
    name: str
    def retrieve(self, query: str, k: int, filters: dict | None) -> list[RankedSegment]: ...

class Fuser(Protocol):
    def fuse(self, ranked_lists: dict[str, list[RankedSegment]], k: int) -> list[SearchHit]: ...
```

## 4. Key algorithms
- **Motion gating:** compare downscaled grayscale frames (frame difference or SSIM); keep a frame only if the change exceeds a threshold or a max interval has passed. Report the percentage of frames skipped.
- **Segment pooling:** mean of L2-normalized frame embeddings, re-normalized. Alternatives (max pooling, keyframe-only) are candidates for an ablation.
- **Reciprocal rank fusion:** `score(s) = Σ_r w_r / (k + rank_r(s))`, with `k ≈ 60` as a starting point; weights default to 1.
- **Temporal IoU:** `overlap(pred, gt) / union(pred, gt)`; a hit counts if IoU ≥ threshold (e.g., 0.3 and 0.5).
- **Adjacent-segment merging:** consecutive high-scoring segments from the same video are merged into one result.

## 5. Pipeline and caching
- Each stage is a function `(inputs, config) -> outputs`, wrapped by a cache layer keyed as in HLD §4.
- Indexing is resumable: rerunning skips cached stages.
- CLI commands (draft): `vsearch ingest <path>`, `vsearch index <video_id|--all>`, `vsearch search "<query>"`, `vsearch eval --config configs/ablations/x.yaml`.

## 6. API (draft)
- `POST /videos` registers a video
- `POST /videos/{id}/index` starts indexing
- `GET /videos/{id}/status` returns per-stage progress
- `GET /search?q=...&k=10&video_id=...` returns `SearchHit[]`
- `GET /segments/{id}/thumbnail`

## 7. Testing strategy
- Unit tests for pure logic: fusion, IoU, motion gating, segment merging.
- Small fixture video (a few seconds) for integration tests of the pipeline.
- Model-dependent components are mocked in unit tests.
