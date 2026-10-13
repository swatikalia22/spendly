# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Spendly is a Flask + SQLite personal expense tracker (amounts in ₹). The repo is a **step-by-step teaching starter**: the UI shell exists, and most features are placeholders labeled with the course step that implements them (e.g. `"Add expense — coming in Step 7"` in `app.py`). Keep this step numbering in mind when you add features, and don't implement later steps unless asked.

**Progress:** Step 1 (database setup) is done.

**Workflow per step:**
- Each step has a spec in `.claude/specs/NN-<name>.md` (e.g. `01-database-setup.md`). Read the step's spec before implementing it.
- Implementation plans are saved to `.claude/plans/NN-<name>.md`. `.claude/plans/` is gitignored.
- Each step is built on its own `feature/<name>` branch.

## Commands

The virtual environment lives at `.venv/` (not `venv/`).

```bash
source .venv/bin/activate
pip install -r requirements.txt
python app.py                         # dev server at http://localhost:5001 (debug=True)
pytest                                # run all tests
pytest tests/test_file.py::test_name  # run a single test
```

- pytest and pytest-flask are in `requirements.txt`, but there is no `tests/` directory yet.
- No linter is configured.
- The dev server keeps running until you stop it. Don't leave it as a long-lived background command; start it only briefly to check it boots, or let the user run it in their own terminal.

## Architecture

- **`app.py`** is a single module with no app factory or blueprints. It holds the `app = Flask(__name__)` instance and every route.
  - At import time it runs `init_db()` and `seed_db()` inside `app.app_context()`, so the DB exists for both `python app.py` and the test client.
  - `/`, `/login` and `/register` render real templates.
  - `/logout`, `/profile` and `/expenses/add|<id>/edit|<id>/delete` return placeholder strings.
- **Database (`database/db.py`)** uses the stdlib `sqlite3`; there is no ORM.
  - `get_db()`: a new connection to `spendly.db` in the project root, with `row_factory = sqlite3.Row` and `PRAGMA foreign_keys = ON`. Callers close it themselves.
  - `init_db()`: creates the tables with `CREATE TABLE IF NOT EXISTS`; safe to call repeatedly.
  - `seed_db()`: inserts a demo user (`demo@spendly.com` / `demo123`, hashed with `werkzeug.security`) and 8 expenses dated in the current month. It returns early if any user exists.
  - `CATEGORIES`: the fixed category list (Food, Transport, Bills, Health, Entertainment, Shopping, Other).
  - Tables:
    - `users`: id, name, email (UNIQUE), password_hash, created_at
    - `expenses`: id, user_id (FK → users.id), amount (REAL), category, date (`YYYY-MM-DD` TEXT), description (nullable), created_at
  - Rules: parameterized queries only (never string-format SQL), amounts stored as REAL, dates as `YYYY-MM-DD`.
  - The DB file is gitignored. Delete `spendly.db` to reset; it is recreated and reseeded on the next startup. A schema change needs this reset, because `CREATE TABLE IF NOT EXISTS` won't alter existing tables.
- **Templates** use Jinja inheritance from `templates/base.html`, which provides the navbar, footer, font loading, `style.css` and `main.js`.
  - Pages override the `title`, `head`, `content` and `scripts` blocks.
  - Links use `url_for('<view function name>')`, so renaming a route function breaks the templates that link to it.
- **Known gap:** `login.html` and `register.html` post forms (`method="POST"`), but their routes only accept GET. Submitting either form returns a 405 until the auth steps add `methods=["GET", "POST"]`.
- **Styling:** all styling is in `static/css/style.css`, built on CSS custom properties defined in `:root`:
  - colors: `--ink*`, `--paper*`, `--accent`, `--accent-2`, `--danger`
  - fonts: `--font-display` (DM Serif Display) and `--font-body` (DM Sans)
  - radii: `--radius-*`
  - widths: `--max-width`, `--auth-width`

  Reuse these variables rather than hardcoding values. `static/js/main.js` is empty and meant for scripts added later.
