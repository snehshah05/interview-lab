# Interview Lab AI workspace — version 2

The GitHub Pages frontend connects to this separately running API. A static Pages deployment does not activate the API, an LLM or a database. The API supports a free local Ollama model, or any configured OpenAI-compatible chat-completions provider supporting JSON mode. Hosted model/hosting fees depend on the provider; none are provisioned by this project.

## Windows / Miniforge: local setup

Run these commands from the extracted `interview-lab` project directory, not SadTalker. Use a separate environment:

```bat
conda create -n interview-ai python=3.11 -y
conda activate interview-ai
python -m pip install -r backend/requirements.txt
ollama list
python backend/setup_local.py
python -m uvicorn backend.main:app --host 127.0.0.1 --port 8000
```

Install Ollama from its official site if needed, and install a model that fits your machine. `ollama list` shows the exact installed name to enter during setup. Keep Ollama running. Larger interview batches can be slow on small local models. Start with 5 questions and verify the quality of generated references and feedback before using larger sessions.

Open the published app, choose **Connections**, enter `http://localhost:8000` and the private workspace token printed by setup, then connect. Browser local-network permissions or browser policy can affect HTTPS Pages → localhost access. If blocked, run the frontend locally (`corepack enable`, `pnpm install --frozen-lockfile`, `pnpm dev`) and open `http://localhost:5173`. Hosted usage requires an HTTPS API URL.

The local database is `backend/data/interviews.sqlite3`. It persists when the API stops. Preserve the database **and** `backend/.env`; the same workspace token is needed to retrieve the same history. Reopening the UI requires reconnecting; tokens are deliberately kept only in tab memory. Database reports include partial sessions if the browser closes after evaluated answers have saved. Resuming unfinished interviews is not implemented; saved answers remain readable.

## What version 2 adds

- Questions depend on role, experience, previous generated questions and weak concepts from recent sessions of that role/level.
- Exact and near-text repeats are rejected against all stored exposures, with up to three batch attempts. Recent 300 titles enter the model context. Novel meanings cannot be guaranteed by text similarity; review model quality. Generation errors never silently substitute bank questions.
- Evaluations accept paraphrases and alternative valid approaches. They include approximate scores, strengths, gaps, quoted mistakes with corrections, and an actionable improvement. Quotes are checked against the transcript. Model errors never invent scores.
- Adaptive follow-ups use the actual answer and recent conversation. Each has its own ID, reference and concepts. Follow-ups count toward the selected 5/10/15/20/25 answer budget; only one follow-up per main question is allowed.
- Each evaluated answer saves immediately. Retrying the same question/transcript retrieves saved feedback; changing an evaluated transcript returns a conflict.
- History and role/level progress come from the database, separated by workspace token. The dashboard shows recent session scores and recurring gaps. A generated but unanswered session remains in history with zero answers.

The frontend's **disconnected** mode remains the original randomized bank with local browser history. Old browser sessions are not automatically uploaded or mixed into AI score trends.

## PostgreSQL / Docker

Copy `backend/.env.example` to `backend/.env` and configure a model and at least one workspace token (32+ characters). For each candidate provision a different high-entropy token; anyone with that token can read that workspace. This is a small private-workspace system, not public self-service registration.

Create a private root `.env` containing `POSTGRES_PASSWORD` (URL-safe random value). Do not commit it. Then:

```sh
docker compose up --build -d
```

Compose starts PostgreSQL with a persistent named volume and the API on localhost port 8000. To use host Ollama from Docker, Ollama must accept traffic from the Docker host gateway according to your OS/network setup; do not expose it to the public Internet. You may instead set root `.env` `LLM_BASE_URL` to a hosted compatible HTTPS endpoint and put its key in `backend/.env`.

For independent hosting set `DATABASE_URL=postgresql+psycopg://...`, `WORKSPACE_TOKENS`, `LLM_BASE_URL`, `LLM_MODEL`, `LLM_API_KEY` and `ALLOWED_ORIGINS=https://snehshah05.github.io` in the host's secret settings. Deploy `backend/Dockerfile` from the project root and attach HTTPS. GitHub Actions is only the frontend build; it must never run the persistent API or contain these secrets. SQLite hosting requires a persistent disk; ephemeral filesystems lose its data.

Deploy one API worker by default. A process-local semaphore permits one concurrent model request and rejects additional requests with 429. Multiple workers need a shared rate limiter. Database constraints and optimistic revisions prevent duplicate answers and budget races. Add account management, managed secrets, monitoring, backups and shared rate limits before wider public service use. Back up PostgreSQL with your host's supported backup process. Initial tables have versioned names; future schema changes require explicit migrations.

## Tests and limitations

```sh
python -m pip install pytest
python -m pytest backend/test_api.py -q
pnpm test
pnpm build
```

API tests use deterministic model fixtures and a real temporary SQLite database: ownership isolation, persistence after reopening the engine, retries, history-aware question generation, repeat rejection, correct follow-up references, unsupported quote rejection and model failures. These are not evidence of a live model's accuracy. Verify your chosen model on correct paraphrases, wrong claims and incomplete answers before relying on feedback. PostgreSQL is supported via SQLAlchemy/psycopg; the automated local suite uses SQLite.

Speech recognition and spoken questions use browser APIs. Camera and microphone remain required; video is not uploaded or analyzed. Raw audio is not saved. Transcripts are sent to your API and configured LLM provider and stored in your database. No lip sync, inferred confidence or personality scoring is included. The earlier unused Whisper/archive endpoints have been replaced by this integrated API.
