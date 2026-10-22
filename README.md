# Kioku (YT Lecture RAG) 

> **Timestamp-level Retrieval-Augmented Generation over YouTube Data Structures & Algorithms (DSA) Lectures.**  
> Ask technical questions in Hinglish or English and get accurate, grounded answers paired with **direct, clickable links that jump to the exact second in the lecture video**.

```
Query: "bhai memoization aur tabulation ka difference kahan explain kiya hai?"

Output:
→ Answer: Concise 3-4 line explanation grounded purely in the lecture transcript.
→ Grounded Sources:
   [1] DP Episode 08 | Subset Sum Equals to Target  @ 24:03   [Jump to Video]
   [2] DP Episode 06 | Tabulation Recipe Explained  @ 15:43   [Jump to Video]
```

Unlike traditional RAG systems that only return raw text snippets, **Kioku returns the exact moment in video** where the concept is taught.

---

## Key Features

-  **Timestamp-Accurate Video Citations**: Leverages time-aligned transcript segmentation so you jump straight to the relevant explanation.
- 🇮🇳 **Bilingual Hinglish/English Understanding**: Handles code-switched Hindi-English queries and technical jargon seamlessly.
-  **Three-Layer Grounding Guardrails**:
  1. **Strict Distance Thresholding**: Unrelated queries are dropped before reaching any LLM.
  2. **Model Semantic Refusal**: Instructed to declare *"Ye topic in lectures me cover nahi hua"* when context is insufficient.
  3. **Zero-Citation Filter**: Answers without grounded timestamp citations are stripped of links to prevent hallucinations.
-  **Zero-Compute Fast Search Mode**: A pure vector-search endpoint (`/search`) providing instant timestamps without consuming LLM quotas.
-  **Battery-Included Bundled Corpus**: Ships with **126 lecture transcripts** and pre-computed vectors (`index/vectors.npz`), enabling instant startup with no GPU required.
-  **Interactive Web Interface**: Single-page dark mode UI with embedded YouTube IFrame API supporting in-page playback and instant seeking.

---

## Tech Stack

| Component | Technology | Rationale |
|---|---|---|
| **Audio Extraction** | `yt-dlp` | Direct stream handling without separate ffmpeg conversion |
| **Transcription** | `faster-whisper` (CTranslate2) | Batched VAD-filtered speech recognition (`large-v3`) |
| **Segmentation** | Custom Segment Chunker | Time-windowed chunking with 75s duration & 15s overlap |
| **Embeddings** | `sentence-transformers` (`all-MiniLM-L6-v2`) | Compact (87MB), fast (384-dim), top-notch retrieval with title re-ranking |
| **Vector Store** | **Qdrant** | Embedded local storage or Qdrant Cloud |
| **LLM Backend** | **Google Gemini** or **Groq** | Fast inference with generous free-tier options |
| **CLI Framework** | `typer` + `rich` | Intuitive command-line interface with progress reporting |
| **API & Web UI** | `FastAPI` + Vanilla CSS/HTML | Lightweight, zero-build frontend with YouTube IFrame player |

---

## Repository Structure

```
Kioku/
├── api/
│   ├── main.py              # FastAPI server with rate limiting & endpoints
│   └── static/
│       └── index.html       # Single-page UI with YouTube IFrame player
├── eval/
│   ├── golden.json          # Golden evaluation dataset (retrieval & refusal test suite)
│   └── golden_draft.json    # Extended 30+ question evaluation set
├── index/
│   └── vectors.npz          # Pre-computed vectors (2,933 chunks, 384-dim, ~4.3MB)
├── transcripts/             # 126 transcribed lecture JSON files (~6MB)
├── ytrag/
│   ├── __init__.py
│   ├── answer.py            # RAG retrieval, grounding, citation renumbering, LLM caller
│   ├── chunk.py             # Time-based segment chunking & repetition filtering
│   ├── cli.py               # Typer CLI application commands
│   ├── config.py            # Unified configuration & environment variable parsing
│   ├── embed.py             # SentenceTransformers embedder protocol
│   ├── evaluate.py          # Benchmark runner against golden evaluation set
│   ├── index.py             # Qdrant client, local/cloud storage, title-boost ranking
│   ├── models.py            # Pydantic & dataclass definitions
│   ├── playlist.py          # yt-dlp playlist scraper & audio downloader
│   ├── transcribe.py        # faster-whisper batched transcriber & atomic cache
│   └── util.py              # Retry decorators and network utilities
├── .env.example             # Environment variable template
├── .gitignore               # Comprehensive Git ignore rules
├── main.py                  # Direct runner entrypoint
├── pyproject.toml           # Hatchling build specification & dependencies
├── README.md                # Project documentation & architectural guide
├── SETUP.md                 # Step-by-step local setup guide
└── uv.lock                  # Deterministic dependency lockfile
```

---

## Quickstart

### 1. Installation
Clone the repository and install dependencies with [`uv`](https://docs.astral.sh/uv/):
```bash
git clone https://github.com/Kanhaiyadav01/Kioku.git
cd Kioku
uv sync
```

### 2. Configure Environment
Copy `.env.example` to `.env` and provide either a Gemini or Groq API key:
```bash
cp .env.example .env
```
In `.env`:
```env
# Get a free key: https://aistudio.google.com/apikey
GEMINI_API_KEY=your_gemini_api_key_here

# OR get a free Groq key: https://console.groq.com/keys
# GROQ_API_KEY=your_groq_api_key_here
```

### 3. Load Prebuilt Index
Populate your local vector store in seconds from the included vectors:
```bash
uv run ytrag load
```

### 4. Run the Web Application
```bash
uv run ytrag serve
```
Open **`http://127.0.0.1:8000`** in your browser.

---

## CLI Usage

| Task | Command | Description |
|---|---|---|
| **Ask Question** | `uv run ytrag ask "kadane's algorithm intuition"` | Full RAG answer with citation links |
| **Search Timestamps** | `uv run ytrag search "binary search on answer"` | Direct timestamp search without LLM overhead |
| **Inspect Index** | `uv run ytrag stats` | Display chunk count, indexed videos, and models |
| **Run Evaluation** | `uv run ytrag eval --verbose` | Run retrieval & refusal accuracy tests |
| **Verify System** | `uv run ytrag preflight` | End-to-end health check across all dependencies |
| **Rebuild Vector Index**| `uv run ytrag reindex` | Re-embed from bundled transcripts (CPU-friendly) |
| **Clean Temp Audio** | `uv run ytrag clean-audio` | Purge any residual downloaded audio files |

---

## Ingesting Your Own Playlist (Optional)

If you wish to index a custom YouTube playlist, ensure you have an NVIDIA GPU (recommended) and execute:

```bash
# 1. (Optional) Test language output on one sample video
uv run ytrag langtest "https://www.youtube.com/watch?v=<VIDEO_ID>"

# 2. Ingest playlist
uv run ytrag ingest --playlist "<PLAYLIST_URL>"
```

- **Resilience**: Ingestion is fully idempotent; partial runs resume where they left off.
- **Atomic Cache**: Transcripts are saved atomically into `~/.ytrag/transcripts` before audio is purged.

---

## Evaluation Benchmark

The system includes a testing suite under `eval/golden.json` measuring:
- **Retrieval Accuracy**: Whether the correct video and timestamp window are retrieved within top-k.
- **Refusal Discipline**: Ensuring out-of-domain questions (e.g., Kubernetes, React, Income Tax) are rejected cleanly with `"Ye topic in lectures me cover nahi hua"`.

Run the evaluator:
```bash
uv run ytrag eval
```

---

## License

This project is licensed under the MIT License.
