# Interview Lab

A voice-based interview practice application with a React frontend and an authenticated FastAPI AI workspace.

Live frontend: https://snehshah05.github.io/interview-lab/

## AI workspace (version 2)

Connect a configured backend to enable:

1. Role- and experience-specific LLM questions informed by previous sessions.
2. Meaning-based evaluation with grounded corrections and recognition of valid paraphrases.
3. Answer-specific follow-ups with their own references and scoring criteria.
4. Database-backed history and progress across sessions, using SQLite locally or PostgreSQL when hosted.

Seven domains; 5, 10, 15, 20 or 25 answers per session; required camera and microphone; browser speech recognition and spoken questions; report export. Follow-ups count toward the answer budget. Lip sync has been removed.

**GitHub Pages only hosts the interface.** Until an AI workspace is connected, it uses a randomized bank, concept checks and browser-local history. A separately running API, database and configured model are required for the four AI features. See [backend setup](backend-setup.md) for Windows/Miniforge, Ollama, PostgreSQL and Docker instructions. AI feedback is fallible practice guidance.

## Source and frontend development

The Pages workflow extracts `Interview-Lab-GitHub.zip` and builds its `interview-lab/` directory. Download and extract that archive for the complete source. The files inside the archive contain a normal source-based Pages workflow for a future unpacked repository layout; the deployed root workflow currently builds the archive.

From the extracted project directory:

```sh
corepack enable
pnpm install --frozen-lockfile
pnpm dev
pnpm test
pnpm build
```

The frontend uses relative asset paths for GitHub Pages. The API keeps model credentials server-side; each candidate uses a distinct private workspace token. History is isolated by that token. Local browser history is separate from AI workspace history.

See `backend/README.md` for setup, deployment boundaries, test coverage and limitations. No model key, workspace token, interview database or private transcript is included in the source archive.
