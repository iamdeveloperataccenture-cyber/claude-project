# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Activate virtual environment (Windows)
venv\Scripts\activate

# Run the development server (port 5001)
python app.py

# Run all tests
pytest

# Run a single test file
pytest tests/test_auth.py

# Run a specific test
pytest tests/test_auth.py::test_login
```

## Architecture

This is a Flask expense tracking app named **Spendly**, structured as a teaching project where students implement features step by step.

**Entry point:** `app.py` — all routes live here. Fully-implemented routes: `/`, `/register`, `/login`, `/terms`, `/privacy`. Stub routes (return plain strings, students implement): `/logout`, `/profile`, `/expenses/add`, `/expenses/<id>/edit`, `/expenses/<id>/delete`.

**Templates:** Jinja2, all extending `base.html`. `base.html` provides the navbar (Spendly brand + Sign in / Get started), `<main>` block, and footer (Terms + Privacy links via `url_for`). Font stack: DM Serif Display (headings) + DM Sans (body) from Google Fonts.

**Database:** `database/db.py` is a stub. Students implement three functions:
- `get_db()` — SQLite connection with `row_factory` and foreign keys enabled
- `init_db()` — creates tables with `CREATE TABLE IF NOT EXISTS`
- `seed_db()` — inserts sample data

No ORM — raw SQLite only.

**Static assets:** `static/css/style.css` uses CSS custom properties (`:root` variables) for the design system. `static/js/main.js` is a stub. No build step — plain CSS and JS.

**CSS design system** (in `:root`): `--color-ink`, `--color-paper`, `--color-accent` (green), `--color-muted`, `--font-display`, `--font-body`, `--radius-*`, `--shadow-*`. Landing page classes are prefixed `.lp-*`.

## Key constraints

- No ORM — database interactions use raw SQLite via `sqlite3` module.
- Passwords must be hashed with `werkzeug.security` (never stored plain).
- The app runs on port **5001** (not the Flask default 5000).
- On corporate networks, SSL verification may need to be disabled for git: `git config --global http.sslVerify false`.
