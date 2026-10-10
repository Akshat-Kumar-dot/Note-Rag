# Notes RAG

[![backend](https://github.com/Akshat-Kumar-dot/Note-Rag/actions/workflows/backend.yml/badge.svg)](https://github.com/Akshat-Kumar-dot/Note-Rag/actions/workflows/backend.yml)
[![frontend](https://github.com/Akshat-Kumar-dot/Note-Rag/actions/workflows/frontend.yml/badge.svg)](https://github.com/Akshat-Kumar-dot/Note-Rag/actions/workflows/frontend.yml)

Ask questions about **your own notes** and get answers that come only from
them, with every sentence linked to the exact passage it came from.

I kept pasting lecture notes into chatbots and getting answers that sounded
right but weren't actually in my notes. Notes RAG fixes that: you upload your
files, ask in plain language, and the answer cites the file and page for each
claim, so you can check it instead of trusting it. If your notes don't cover
the question, it says so instead of making something up.

Answers come from **Gemma 4** (`gemma-4-26b-a4b-it`), Google's open-weights
model, through the Gemini API.

**Live demo:** https://notes-rag.akshat-kumar-singh05.workers.dev

## Features

- **Ask AI**: answers streamed back with numbered citations to your passages.
- **Search Notes**: the same search with no AI at all, just the matching
  passages, so nothing can be invented.
- **Says when it doesn't know**: low-confidence answers are marked as such.
- **Multimodal input**: PDFs, Word (.docx), Markdown, plain text, and images
  of handwritten or printed notes (PNG/JPG/WebP) through OCR.
- **Study map**: a grid of every passage in your notes that lights up where
  you've studied and stays dark where you haven't.
- **Teach me / quiz me**: pick a dark spot and it explains that part, or quizzes
  you and grades your answer against your notes.
- **Folders**: each chat is scoped to the folders you pick, so courses don't
  bleed into each other.
- **Chat graph**: chats that used the same passages are linked together.
- **Free trial without signing in**: one document and two questions, limited
  per device and per network.

## How it works

1. **Upload**: the file is parsed and split into ~450-token chunks that keep
   their headings and page numbers. Each chunk is embedded and stored in
   Postgres with pgvector. The original file is thrown away after that.
2. **Retrieve**: the question runs through vector search and Postgres
   full-text search at the same time. The two lists are merged with Reciprocal
   Rank Fusion, so names, formulas and course codes that embeddings miss still
   get found.
3. **Answer**: the top passages go to Gemma 4 with strict instructions to use
   only them and cite them. The answer streams back over SSE.

### Why Gemma 4

Gemma 4 is open-weights, so the answering model isn't locked to one provider.
The 26B mixture-of-experts variant (4B active parameters) answers in about two
seconds. It thinks before answering by default; the client sets
`thinkingLevel: minimal` and drops the `thought` parts so only the answer
reaches you (see [`backend/app/llm/gemini.py`](backend/app/llm/gemini.py)).

## Stack

| Part | Tech |
|---|---|
| Answers | Gemma 4 via the Gemini API, streamed |
| Embeddings | `gemini-embedding-001` (768 dims) |
| API | FastAPI, Python 3.12, async SQLAlchemy |
| Database | Postgres + pgvector |
| Chat history | MongoDB |
| Search | Vector + Postgres full-text, merged with RRF |
| Parsing | PyMuPDF, python-docx, Tesseract for images |
| Frontend | Next.js 15 static export |
| Hosting | Cloudflare Worker (site), Render (API, Docker) |

## Running it locally

You need Python 3.12, Node 22, Postgres with pgvector (Neon's free tier works),
and MongoDB (local, or Atlas).

```bash
cp .env.example backend/.env
```

Fill in the values (table below), then set up the backend:

```bash
cd backend
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
alembic upgrade head
```

Build the frontend and copy it into the backend, so the page and the API share
an origin and the login cookie works:

```bash
cd ../frontend
npm install
npm run build
Remove-Item -Recurse -Force ..\backend\static -ErrorAction SilentlyContinue
Copy-Item -Recurse out ..\backend\static
```

Start it:

```bash
cd ../backend
uvicorn app.main:app --reload
```

Open http://localhost:8000.

## Environment variables

| Variable | Where to get it |
|---|---|
| `DATABASE_URL` | Neon → Connect → connection string (paste it as is) |
| `MONGODB_URL` | `mongodb://localhost:27017` locally, Atlas connection string in production |
| `SECRET_KEY` | `python -c "import secrets; print(secrets.token_hex(32))"` |
| `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` | Google Cloud Console → Google Auth Platform → Clients |
| `GEMINI_API_KEY` | Google AI Studio → Get API key (the same key serves Gemma 4) |
| `CHAT_MODEL` | Optional. Defaults to `gemma-4-26b-a4b-it`. |
| `PUBLIC_BASE_URL` | `http://localhost:8000` locally. No trailing slash. |
| `ENV` | `dev` locally, `prod` in production |

The Google OAuth redirect URI has to be `<PUBLIC_BASE_URL>/api/v1/auth/google/callback`.

## Tests and CI

```bash
cd backend
pip install -r requirements-dev.txt
python -m pytest tests -q
```

Every pull request runs lint and tests for the backend, and a type check and
build for the frontend, in GitHub Actions.

## Where things are

```
backend/app/
  main.py          app setup, routes
  config.py        all settings in one place
  auth/            Google login, sessions, the guest trial
  api/             folders, files, search, chat (SSE), study map and quizzes
  parsers/         one file per format
  rag/             chunking, embeddings, hybrid search, prompts
  llm/             Gemma 4 client (Gemini API)
  chat_store.py    chat history in MongoDB
  study_store.py   study events and quizzes in MongoDB
  ingest.py        parse -> chunk -> embed -> store
frontend/
  app/             landing page (/) and the workspace (/app)
  components/      sidebar, chat, files, study map, chat graph
  lib/             API client, dictation, scroll helpers
  worker/          the Cloudflare Worker that proxies /api
```

[ARCHITECTURE.md](ARCHITECTURE.md) explains the design choices.

## Team

- Akshat Kumar ([@Akshat-Kumar-dot](https://github.com/Akshat-Kumar-dot))
- Kuresh Raj ([@kuresh-username](https://github.com/kuresh-username))
- Neel Patel ([@neel-username](https://github.com/neel-username))

## License

[MIT](LICENSE)
