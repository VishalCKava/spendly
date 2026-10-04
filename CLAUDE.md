# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Spendly: a Flask expense tracker built as a step-by-step course project. Placeholder routes in `app.py` return strings like "coming in Step 7"; they are meant to be implemented in order. `database/db.py` is comments only and describes the Step 1 functions to write: `get_db()` (SQLite connection with `row_factory` and foreign keys on), `init_db()` (`CREATE TABLE IF NOT EXISTS`) and `seed_db()` (sample data).

## Commands

Windows, PowerShell. Use the venv's Python directly. The system Python 3.8 has a broken pip, and activation does not persist between tool calls.

```powershell
.\venv\Scripts\python.exe -m pip install -r requirements.txt
.\venv\Scripts\python.exe app.py                  # dev server, debug=True, http://127.0.0.1:5001
.\venv\Scripts\python.exe -m pytest               # all tests (none exist yet; pytest + pytest-flask are installed)
.\venv\Scripts\python.exe -m pytest tests/test_x.py::test_name   # single test
```

The app runs on port **5001**, not Flask's default 5000. Background servers started from Claude Code are killed at the timeout, so ask the user to run the server in their own terminal.

There is no linter or build step configured.

## Architecture

- The git root and project root are the inner `expense-tracker/` folder (the outer folder of the same name is only a container).
- `app.py` holds all routes. Public pages (`/`, `/login`, `/register`, `/terms`, `/privacy`) only call `render_template`. There is no form handling, session or database code yet.
- All templates extend `templates/base.html`, which owns the navbar, the footer (including the Terms and Privacy links) and the `title`, `head`, `content` and `scripts` blocks. Footer changes go in `base.html`, not in `landing.html`.
- Styling: `static/css/style.css` is loaded globally. `landing.css` is loaded only by `landing.html`. `terms.html` and `privacy.html` reuse the `auth-section`, `auth-container` and `auth-card` classes from `style.css` instead of having their own CSS.
- Link between pages with `url_for('<function name>')`, using the route function names in `app.py`.

## Code style
- Pythong: PEP8, snake_case for all variables and functions
- Templates: Jingja2 with `url_for()` for every internal link - never hard code urls
- Route funtions: one responsibility only - fetch data, render template, done
- DB queries: always user parameterized queries (`?` placeholders ) - never f-strings in SQL
- Error handling: use `abort()` for http errors, not bare `return "error string"`


## Tech constraints

- **Flask only** — no FastAPI, no Django, no other web frameworks
- **SQLite only** — no PostgreSQL, no SQLAlchemy ORM, no external DB
- **Vanilla JS only** — no React, no jQuery, no npm packages
- **No new pip packages** — work within `requirements.txt` as-is unless explicitly told otherwise
- Python 3.10+ assumed — f-strings and `match` statements are fine

---

## Subagent Policy
- Always use a builtin explore subagent for codebase exploration 
  before implementing any new feature
- Always use a subagent to verify test results 
  after any implementation
- When asked to plan, delegate codebase research 
  to a subagent before presenting the plan
- always use a builtin plan subagent in plan mode

---

## Commands
```bash
# Setup
python -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate
pip install -r requirements.txt

# Run dev server (port 5001)
python app.py

# Run all tests
pytest

# Run a specific test file
pytest tests/test_foo.py

# Run a specific test by name
pytest -k "test_name"

# Run tests with output visible
pytest -s


## Implemented vs stub routes

| Route | Status |
|---|---|
| `GET /` | Implemented — renders `landing.html` |
| `GET /register` | Implemented — renders `register.html` |
| `GET /login` | Implemented — renders `login.html` |
| `GET /logout` | Stub — Step 3 |
| `GET /profile` | Stub — Step 4 |
| `GET /expenses/add` | Stub — Step 7 |
| `GET /expenses/<id>/edit` | Stub — Step 8 |
| `GET /expenses/<id>/delete` | Stub — Step 9 |

**Do not implement a stub route unless the active task explicitly targets that step.**

---

## Warnings and things to avoid

- **Never use raw string returns for stub routes** once a step is implemented — always render a template
- **Never hardcode URLs** in templates — always use `url_for()`
- **Never put DB logic in route functions** — it belongs in `database/db.py`
- **Never install new packages** mid-feature without flagging it — keep `requirements.txt` in sync
- **Never use JS frameworks** — the frontend is intentionally vanilla
- **`database/db.py` is currently empty** — do not assume helpers exist until the step that implements them
- **FK enforcement is manual** — SQLite foreign keys are off by default; `get_db()` must run `PRAGMA foreign_keys = ON` on every connection
- The app runs on **port 5001**, not the Flask default 5000 — don't change this
