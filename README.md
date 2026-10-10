# Notes RAG

Ask questions about **your own notes** and get answers that come only from
them, with every sentence linked to the exact passage it came from.

I kept pasting lecture notes into chatbots and getting answers that sounded
right but weren't actually in my notes. Notes RAG fixes that: you upload your
files, ask in plain language, and the answer cites the file and page for each
claim, so you can check it instead of trusting it. If your notes don't cover
the question, it says so instead of making something up.

Built on **Gemma 4** through the Gemini API.

## Features

- **Ask AI**: answers streamed back with numbered citations to your passages.
- **Search Notes**: the same search with no AI at all, just the matching
  passages, so nothing can be invented.
- **Says when it doesn't know**: low-confidence answers are marked as such.
- **Multimodal input**: PDFs, Word (.docx), Markdown, plain text, and images
  of handwritten or printed notes (PNG/JPG/WebP).
- **Study map**: a grid of every passage in your notes that lights up where
  you've studied and stays dark where you haven't.
- **Teach me / quiz me**: pick a dark spot and it explains that part, or quizzes
  you and grades your answer against your notes.
- **Folders**: each chat is scoped to the folders you pick, so courses don't
  bleed into each other.
- **Chat graph**: chats that used the same passages are linked together.

## How it works

1. **Upload**: the file is parsed and split into ~450-token chunks that keep
   their headings and page numbers. Each chunk is embedded and stored in
   Postgres with pgvector.
2. **Retrieve**: the question runs through vector search and Postgres
   full-text search at the same time. The two lists are merged with Reciprocal
   Rank Fusion, so names, formulas and course codes that embeddings miss still
   get found.
3. **Answer**: the top passages go to Gemma 4 with strict instructions to use
   only them and cite them. The answer streams back over SSE.

## Stack

| Part | Tech |
|---|---|
| Model | Gemma 4 (Gemini API) |
| API | FastAPI, Python 3.12, async SQLAlchemy |
| Database | Postgres + pgvector |
| Chat history | MongoDB |
| Search | Embeddings + Postgres full-text, merged with RRF |
| Parsing | PyMuPDF, python-docx, OCR for images |
| Frontend | Next.js 15 |

## Status

This repo is being built in the open. Work is planned in
[issues](../../issues), one branch and one pull request per issue. Setup
instructions will land here as the backend and frontend are merged in.

## Team

- Akshat Kumar ([@Akshat-Kumar-dot](https://github.com/Akshat-Kumar-dot))

## License

[MIT](LICENSE)
