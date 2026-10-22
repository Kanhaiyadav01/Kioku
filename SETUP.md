# Setup & Installation Guide

This guide details how to get **Kioku** running locally on your system.

Because Kioku includes **126 pre-generated transcripts** and a **pre-computed vector index** (`index/vectors.npz`), you can run the entire system locally in under 3 minutes without requiring a GPU, transcription downloads, or a vector database cloud account.

---

## Prerequisites

1. **Python**: Python 3.11 or 3.12 installed.
2. **Package Manager**: [`uv`](https://docs.astral.sh/uv/) (recommended for speed and consistency).
   ```bash
   # On macOS/Linux:
   curl -LsSf https://astral.sh/uv/install.sh | sh

   # On Windows (PowerShell):
   powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"

   # Or via pip:
   pip install uv
   ```

---

## Quick Setup Steps

### 1. Clone & Install Dependencies

```bash
git clone https://github.com/Kanhaiyadav01/Kioku.git
cd Kioku
uv sync
```

*(Optional for NVIDIA GPU users running ingestion)*:
```bash
uv sync --extra cuda
```

### 2. Configure Your API Key

Create your local `.env` file from the provided example template:

```bash
# Windows PowerShell
copy .env.example .env

# Linux / macOS
cp .env.example .env
```

Open `.env` in your text editor and set **ONE** of the following free LLM API keys:

- **Google Gemini (Recommended)**: Get a free key at [Google AI Studio](https://aistudio.google.com/apikey)
  ```env
  GEMINI_API_KEY=your_gemini_api_key_here
  ```
- **Groq**: Get a free key at [Groq Console](https://console.groq.com/keys)
  ```env
  GROQ_API_KEY=your_groq_api_key_here
  ```

> **Note**: You do not need a Qdrant account or Docker container. Kioku stores vectors locally in `~/.ytrag/qdrant` by default.

### 3. Load the Prebuilt Index

Populate your local vector store with the 2,933 pre-computed lecture chunks in seconds:

```bash
uv run ytrag load
```

This reads `index/vectors.npz` (4.3 MB, included in the repo) and loads it into your local store. The only one-time download is the 87 MB `all-MiniLM-L6-v2` query encoder.

### 4. Start the Application

```bash
uv run ytrag serve
```

Access the UI at:
👉 **`http://127.0.0.1:8000`**

---

## Daily Usage & CLI Reference

Once initial setup is complete, you can interact with Kioku via the browser or CLI:

### Web UI
```bash
uv run ytrag serve
```

### Terminal Commands

- **Ask with Written Grounded Explanation**:
  ```bash
  uv run ytrag ask "two pointer technique kab use karte hain"
  ```
- **Fast Timestamp Search (Zero LLM / Zero Quota)**:
  ```bash
  uv run ytrag search "dijkstra algorithm intuition"
  ```
- **Verify Index Statistics**:
  ```bash
  uv run ytrag stats
  ```
- **Run System Preflight Check**:
  ```bash
  uv run ytrag preflight
  ```

> ⚠️ **Single Process Lock**: The embedded local Qdrant engine uses a single-writer file lock. Stop (`Ctrl+C`) `ytrag serve` before executing `ytrag reindex` or `ytrag ask` in another terminal, or configure `QDRANT_URL` in `.env` for multi-client concurrency.

---

## Ingesting a Custom Playlist

If you want to index your own video playlist:

1. **Verify your environment**:
   ```bash
   uv run ytrag preflight
   ```
2. **Ingest the playlist**:
   ```bash
   uv run ytrag ingest --playlist "https://www.youtube.com/playlist?list=YOUR_PLAYLIST_ID"
   ```
3. **Export your new index** to share:
   ```bash
   uv run ytrag export-vectors-cmd
   uv run ytrag export-transcripts
   ```

---

## Troubleshooting

| Symptom | Cause & Solution |
|---|---|
| `GROQ_API_KEY` / `GEMINI_API_KEY is not set` | Ensure `.env` exists in the `Kioku` root folder and contains a valid key. |
| `Local index is already open in another process` | Stop running instances of `ytrag serve` before running terminal queries. |
| `These vectors were built with...` | Your configured `YTRAG_EMBED_MODEL` does not match the loaded index. Keep default `all-MiniLM-L6-v2` or run `uv run ytrag reindex`. |
| `No prebuilt index found` | Run `uv run ytrag load` from the project root. |
| `Warning: You are sending unauthenticated requests to the HF Hub` | Safe warning from HuggingFace Hub when loading public models without a token. It can be safely ignored. |
