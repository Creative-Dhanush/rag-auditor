# 🛡️ RAG Auditor

**Free, browser-based RAG (Retrieval-Augmented Generation) system — powered by Google Gemini API.**

Upload PDFs, ask questions in natural language, and get grounded answers with source citations. Runs entirely in your browser — no backend server needed.

![GitHub](https://img.shields.io/badge/license-MIT-blue)
![PRs](https://img.shields.io/badge/PRs-welcome-brightgreen)

---

## ✨ Features

| Feature | Description |
|---|---|
| **📄 PDF Upload** | Drag-and-drop PDF/TXT files. Text extracted via PDF.js in-browser |
| **✂️ Smart Chunking** | 512-token sliding window with 50-token overlap, sentence-boundary aware |
| **🔍 TF-IDF Vector Search** | In-browser TF-IDF + cosine similarity — no API cost, runs fully client-side |
| **↕️ BM25 Reranking** | Keyword overlap reranker filters top-50 → top-8 most relevant chunks |
| **🧠 Gemini Generation** | Google Gemini 2.0 Flash (free tier) generates grounded answers from retrieved chunks |
| **🔒 Guardrails** | Injection detection, out-of-scope filter, output hallucination checks |
| **🔁 Graceful Fallback** | On API failure, shows raw retrieved chunks — never crashes |
| **🗄️ Vector CRUD** | Targeted page-level upsert/delete without full rebuild |
| **🌓 Dark Theme** | Modern dark UI, responsive layout, works on mobile |

---

## 🚀 Quick Start

### 1. Open the app

Open `rag-auditor.html` in any modern browser (Chrome, Edge, Firefox).

> ⚠️ If you get CORS errors, serve via a local server:
> ```bash
> npx serve .
> # or
> python -m http.server 8000
> ```

### 2. Get a free Gemini API key

1. Go to [aistudio.google.com/apikey](https://aistudio.google.com/apikey)
2. Click **"Create API Key"**
3. Copy your key (starts with `AIza...`)

### 3. Upload documents

- Drag PDFs onto the upload zone, or click to browse
- Supports `.pdf` and `.txt` files up to 50 MB
- The pipeline extracts text, chunks, and indexes automatically

### 4. Ask questions

Type any question about your documents. The system:
1. Checks guardrails (injection / out-of-scope)
2. Retrieves top-50 chunks via TF-IDF
3. Reranks to top-8 via BM25 overlap
4. Sends context to Gemini 2.0 Flash for a grounded answer
5. Displays answer with source citations

No API key? The app works in **local-only mode** — upload docs, search, and browse raw chunks.

---

## 🏗️ Architecture

### Ingestion Pipeline

```
PDF/TXT → PDF.js Extraction → Chunking (512t, 50 overlap) → TF-IDF Vectorization → In-Memory Store
```

### Query Pipeline

```
Question → Input Guardrail → TF-IDF Search (top-50) → BM25 Rerank (top-8) → Gemini Generation → Output
```

### Why in-browser?

| Component | Where it runs | Cost |
|---|---|---|
| PDF extraction | Browser (PDF.js) | Free |
| Text chunking | Browser (JS) | Free |
| Vector search | Browser (TF-IDF) | Free |
| Answer generation | Google Gemini API | Free tier (1500 req/day) |
| All data | In-memory (never leaves your machine except to Gemini API) | — |

---

## ⚙️ Configuration

### Settings panel

| Setting | Default | Description |
|---|---|---|
| Model | `gemini-2.0-flash` | Choose from Flash, Flash-Lite, or 1.5 Flash |
| Max tokens | 1024 | Maximum response length |
| Temperature | 0.2 | Lower = more factual, higher = more creative |
| Top-K (initial) | 50 | Chunks retrieved before reranking |
| Top-K (rerank) | 8 | Chunks sent to Gemini after reranking |
| Chunk size | 1800 chars (~512 tokens) | Size of each text chunk |

### Toggle guardrails

- **Input injection detection** — blocks prompt injection / jailbreak attempts
- **Out-of-scope filter** — blocks unrelated queries (recipes, weather, etc.)
- **Output hallucination check** — verifies claims against source chunks
- **Graceful API fallback** — shows raw chunks when API is unavailable

---

## 🔧 API Reference

### `POST` Gemini Generate Content

```http
POST https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=YOUR_KEY
```

Request body:

```json
{
  "contents": [{ "parts": [{ "text": "Your question with context" }] }],
  "systemInstruction": { "parts": [{ "text": "System prompt..." }] },
  "generationConfig": {
    "maxOutputTokens": 1024,
    "temperature": 0.2
  }
}
```

---

## 🧪 Testing Failover

Navigate to **API Failover** panel to simulate:

| Scenario | Behavior |
|---|---|
| Timeout (30s) | Request aborts → fallback chunks shown |
| Rate limit (429) | Error caught → graceful degradation |
| Server error (500) | Same fallback path |

All simulated failures trigger the amber warning banner. Click **Restore API** to recover.

---

## 🛣️ Roadmap

- [x] PDF upload & text extraction
- [x] TF-IDF vector search
- [x] BM25 reranking
- [x] Gemini API integration
- [x] Guardrails (injection, OOS, hallucination)
- [x] Graceful API failover
- [x] Vector index CRUD
- [ ] Persistent storage (IndexedDB)
- [ ] Multi-document comparison
- [ ] Export Q&A sessions
- [ ] Hugging Face / Ollama provider support
- [ ] Sentence embedding (all-MiniLM-L6-v2 via ONNX)

---

## 📝 License

MIT — free to use, modify, and distribute.

Built with [PDF.js](https://mozilla.github.io/pdf.js/) and [Google Gemini API](https://ai.google.dev/).
