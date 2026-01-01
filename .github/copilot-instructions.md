# Guidance for AI coding agents

This repository is a minimal FastAPI app serving a small in-memory activities API and a static frontend. Keep edits focused, minimal, and consistent with existing patterns.

## Big picture
- Service: `src/app.py` — FastAPI application exposing two main behaviors:
  - `GET /activities` — returns an in-memory `activities` dict keyed by activity name.
  - `POST /activities/{activity_name}/signup?email=...` — appends an email to the activity's `participants` list.
- Frontend: `src/static/` — simple static site (`index.html`, `app.js`, `styles.css`) mounted at `/static` by the FastAPI app. The frontend fetches `/activities` and posts to `/activities/{name}/signup` (it URL-encodes activity names).
- Data: purely in-memory (Python dict in `src/app.py`) — no persistence; server restart resets state.

## Developer workflow (how to run & test locally)
- Install dependencies:

  pip install -r requirements.txt

- Run the app (development):

  uvicorn src.app:app --reload

  or (simple):

  python src/app.py

- Open: `http://localhost:8000/docs` for interactive API docs, or `http://localhost:8000/static/index.html` for the UI.

## Project-specific conventions & patterns
- Activity identifier: the app uses the activity name (string) as the unique key. Any code changes must account for URL-encoding in the frontend (`encodeURIComponent`) and path parameters.
- Participant storage: participants are stored as a list of emails in `activities[activity_name]['participants']`. Current code appends blindly — there are no duplicate checks or capacity enforcement. If you add validation, update both backend and frontend error handling.
- Static mounting: `app.mount("/static", StaticFiles(directory=...))` points to `src/static`. When changing static files, edit `src/static/*`.
- Tests: there are no tests in the repo. `pytest.ini` sets `pythonpath = .` so tests should import `src.app` directly (use `from src import app` or `from src.app import app` when writing tests).

## Integration points & external dependencies
- Runtime: FastAPI + Uvicorn (see `requirements.txt`).
- Frontend to backend: calls to `/activities` and `/activities/{activity_name}/signup` (POST with query `email`). The frontend expects JSON responses and uses HTTP status codes to differentiate success vs error.

## Pitfalls and things reviewers should check
- When modifying signup behavior, ensure the frontend `src/static/app.js` handles non-200 responses (it expects an error object with `detail`).
- If renaming an activity or changing identifier semantics, update the dropdown population code in `src/static/app.js` which uses object keys from `/activities`.
- Be explicit about URL encoding when constructing paths containing activity names.

## Small examples (use these in PRs and edits)
- GET activities (curl):

  curl http://localhost:8000/activities

- Sign up (curl) — note encoding of the activity name:

  curl -X POST "http://localhost:8000/activities/Chess%20Club/signup?email=student%40example.com"

## When you change behavior
- Update `src/README.md` and the root `README.md` with any new developer commands or API changes.
- If you add persistence, move data access into a small adapter module (e.g., `src/storage.py`) and provide an in-memory fallback for development.

## Final notes for the agent
- Keep changes small and easily reversible. This is an exercise repo: prefer clarity over cleverness.
- When in doubt about the frontend contract, run `uvicorn src.app:app --reload` and exercise the UI at `/static/index.html`.

Please review and tell me if you'd like additional examples, tests, or a storage adapter scaffold.
