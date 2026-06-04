# RAG-Based Video Q&A (Sigma Course)

This project is a Retrieval-Augmented Generation (RAG) assistant that makes a video course instantly searchable and answerable. It transcribes course videos into time-stamped text chunks, creates semantic embeddings, stores them in a local Chroma vector database, and uses an LLM to answer user questions with relevant excerpts and exact video timestamps.

**Key features**
- Convert video → MP3 → time‑stamped JSON transcripts
- Combine short transcript chunks into richer segments for better context
- Create semantic embeddings with Sentence‑Transformers
- Store and query vectors in a local Chroma DB (`chroma_db/`)
- Build prompt with retrieved context and call an LLM (Groq) for synthesis
- Offline mock mode (`process_incoming_mock.py`) for development without API keys

**Repository layout**
- `video_to_mp3.py` — helpers to extract MP3 from video files
- `mp3_to_json.py` — transcribe MP3s into `jsons/*.mp3.json` with `{title,number,start,end,text}` chunks
- `preprocess_json.py` — combines chunks, creates embeddings, writes to `chroma_db`
- `process_incoming.py` — interactive query flow using Groq LLM
- `process_incoming_mock.py` — local mock for testing retrieval + synthesis
- `jsons/` — input transcription JSON files
- `chroma_db/` — local Chroma persistent database (ignored by git)
- `requirements.txt` / `requirements-dev.txt` — dependencies

**How it works (high level)**
1. Transcribe videos into short time-stamped chunks (few seconds each).
2. Combine consecutive short chunks into larger segments (preprocess step) so each document contains meaningful context (recommended 15–30s per segment).
3. Generate embeddings (`sentence-transformers/all-MiniLM-L6-v2`) for each combined segment and store them with metadata in Chroma.
4. On a user query: encode the question, retrieve top‑k segments from Chroma, construct a prompt with text + metadata, and call an LLM to synthesize an answer that cites video titles and timestamps.

## Quickstart

Prerequisites
- Python 3.10+ (3.14 in tested environment)
- Git (optional)

1. Create and activate virtual environment (PowerShell):

```powershell
python -m venv .venv
. .\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

2. (Optional) Install dev tools:

```powershell
pip install -r requirements-dev.txt
```

3. Preprocess and index transcripts (this will create `chroma_db/`):

```powershell
# (optional) remove existing DB if you want a fresh index
Remove-Item -Recurse -Force chroma_db
.\.venv\Scripts\python.exe preprocess_json.py
```

4. Run the interactive query (real LLM via Groq):

```powershell
$env:GROQ_API_KEY = 'your_api_key_here'
.\.venv\Scripts\python.exe process_incoming.py
# then type your question at the prompt
```

5. Run in offline/mock mode (no API key):

```powershell
.\.venv\Scripts\python.exe process_incoming_mock.py
```

## Using Your Own Videos

To index your own video course:

1. Place your video files (MP4, MKV, etc.) in a folder (e.g., `input_videos/`)

2. Generate MP3s and transcripts:

```powershell
.\.venv\Scripts\python.exe video_to_mp3.py  # extracts audio from videos
.\.venv\Scripts\python.exe mp3_to_json.py    # transcribes to timestamped JSON
```

3. Preprocess and build the vector database:

```powershell
Remove-Item -Recurse -Force chroma_db  # start fresh
.\.venv\Scripts\python.exe preprocess_json.py
```

4. Query your indexed course using steps 4–5 above

**Note:** The `jsons/` folder contains sample transcripts from a web development course. Replace them with your own transcriptions or regenerate using the steps above.

## Configuration & Environment
- `GROQ_API_KEY` — your Groq API key for production LLM calls (do not commit).
- `HF_TOKEN` — optional Hugging Face token to increase model download limits.

## Troubleshooting
- If your LLM returns only timestamps and no explanations, the indexed chunks are probably too small or fragmented. Re-run `preprocess_json.py` after adjusting `chunk_size` (default is 5) to combine more consecutive fragments.
- If pip installs fail due to disk space on C:, use a different temp folder on a drive with space:

```powershell
New-Item -ItemType Directory -Path D:\pip_temp -Force
$env:TEMP='D:\pip_temp'; $env:TMP='D:\pip_temp'
.\.venv\Scripts\python.exe -m pip install chromadb --no-cache-dir
```

## Development tips
- Experiment with chunking strategies: fixed grouping, sliding windows, sentence-boundary merging, or semantic segmentation.
- Consider adding a reranker (cross-encoder) after retrieval for better precision.
- Precompute summaries for each chunk and index summaries alongside full text.
- Add logging and error handling around LLM requests and model deprecations.

## Next steps / Improvements
- Add a web UI or Slack bot interface for end users.
- Add automated tests for retrieval quality and prompt generation.
- Integrate a cross-encoder reranker and a summarization stage.

## License & Attribution
This project uses open-source libraries: SentenceTransformers, ChromaDB, HuggingFace transformers, and Groq client. Check each package's license for redistribution terms.

---
