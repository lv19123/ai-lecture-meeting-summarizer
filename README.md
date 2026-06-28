# AI Lecture / Meeting Summarizer

AI-powered web application for processing lectures, meetings, documents, audio/video files, and YouTube transcripts. It turns raw material into extracted text, cleaned notes, short reports, topics, key terms, searchable context, and downloadable reports.

This is a portfolio MVP, not a production SaaS. It is intentionally built as a deterministic AI pipeline, not an autonomous agent.

## Features

- PDF, DOCX, TXT, and Markdown document processing
- Audio/video transcription with timestamps using `faster-whisper`
- YouTube transcript/subtitle processing without downloading video or audio
- Short report generation
- Full cleaned notes generation
- Topics with source references
- Key terms with definitions and source references
- RAG question answering over uploaded material
- Markdown, PDF, and DOCX export
- FastAPI backend
- Streamlit frontend
- Docker and Docker Compose support
- Pytest test suite with deterministic fallback modes

## Tech Stack

- Python
- FastAPI
- Streamlit
- faster-whisper
- OpenRouter-compatible LLM API
- scikit-learn TF-IDF retrieval
- ReportLab
- python-docx
- pypdf
- Docker
- pytest

## Architecture

Pipeline summary:

```text
file upload / YouTube URL
-> source detection
-> text extraction or transcript extraction
-> segments.json + extracted_text.txt
-> reports, topics, terms
-> TF-IDF RAG index
-> grounded question answering
-> Markdown / PDF / DOCX export
```

See [docs/architecture.md](docs/architecture.md) for the full architecture and [docs/api_overview.md](docs/api_overview.md) for endpoint groups.

## Quick Start Locally

Create a virtual environment:

```bash
python -m venv .venv
source .venv/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Run the backend:

```bash
uvicorn backend.app.main:app --reload
```

Run the frontend in another terminal:

```bash
streamlit run frontend/app.py
```

Local URLs:

- FastAPI: http://localhost:8000
- FastAPI docs: http://localhost:8000/docs
- Streamlit UI: http://localhost:8501

## Run With Docker

Create a local environment file:

```bash
cp .env.example .env
```

Build and start the services:

```bash
docker compose build
docker compose up
```

Docker URLs:

- FastAPI: http://localhost:8000
- FastAPI docs: http://localhost:8000/docs
- Streamlit UI: http://localhost:8501

The `data/` directory is mounted into the containers, so uploaded files, processed text, reports, and RAG indexes persist on the host. The Docker image includes `ffmpeg` for video audio extraction.

## Demo

Use the included original sample lecture:

```text
sample_data/ml_lecture_sample.txt
```

Manual demo steps are in [docs/demo_scenario.md](docs/demo_scenario.md).

You can also run a smoke demo against an already running backend:

```bash
python scripts/smoke_demo.py
```

The script uploads the sample lecture, processes it, generates reports/topics/terms, builds a RAG index, asks a question, and downloads a Markdown report.

## Environment Variables

Copy `.env.example` to `.env` for local configuration. Important variables:

- `BACKEND_URL`: Streamlit backend URL. Use `http://localhost:8000` locally; Docker Compose sets `http://backend:8000` for the frontend container.
- `OPENROUTER_API_KEY`: Optional API key for real LLM generation. If empty, deterministic fallback responses are used.
- `OPENROUTER_BASE_URL`: OpenRouter-compatible API base URL.
- `LLM_MODEL`: Chat model name.
- `STT_MODEL_SIZE`, `STT_DEVICE`, `STT_COMPUTE_TYPE`: faster-whisper settings.
- `STT_USE_FAKE_TRANSCRIBER`: Set `true` for deterministic local/test audio transcription without model downloads.
- `YOUTUBE_USE_FAKE_TRANSCRIPT`: Set `true` for deterministic local/test YouTube transcripts without internet.
- `YOUTUBE_LANGUAGES`: Preferred YouTube transcript languages, for example `ru,en`.

First real `faster-whisper` transcription may take time because the model can download and load on first use. YouTube processing uses only available transcripts/subtitles and does not download YouTube video or audio.

## Testing

```bash
pytest
```

The tests are designed to run without OpenRouter API access, real Whisper model downloads, real YouTube access, GPU, or ffmpeg-dependent video fixtures.

## Limitations

- This is a portfolio MVP, not a production-ready multi-user service.
- Long documents and transcripts use simple truncation in some LLM prompts; map-reduce processing is a future improvement.
- RAG currently uses local TF-IDF retrieval rather than semantic embeddings.
- YouTube processing depends on available transcripts/subtitles and does not download video or audio.
- Real audio/video transcription can take time and may require model downloads on first use.

## API Reference

See [docs/api_overview.md](docs/api_overview.md). FastAPI interactive docs are available at:

```text
http://localhost:8000/docs
```

## Project Status

Portfolio MVP.

Future improvements:

- Embeddings plus ChromaDB or FAISS instead of TF-IDF
- Async background tasks for long media processing
- Better long-document map-reduce summarization
- Authentication and user accounts
- Richer frontend for multi-material workflows
- More robust Markdown rendering and report layout controls
