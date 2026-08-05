# Plan: 01 — Database Setup

## Context

`database/db.py` is currently a stub with only comments. All future steps (auth, expenses, profile) depend on a working SQLite data layer. This plan implements the three required functions and wires them into `app.py` startup so the database is ready before any request is handled.

---

## Files to Change

| File | What changes |
|------|-------------|
| `database/db.py` | Implement `get_db()`, `init_db()`, `seed_db()` from scratch |
| `app.py` | Add imports + call `init_db()` / `seed_db()` inside an app context at startup |

No new files. No new pip packages.

---

## Implementation

### `database/db.py`

**Imports needed:**
```python
import sqlite3
import os
from werkzeug.security import generate_password_hash
```

**`get_db()`**
- Resolve path: `os.path.join(os.path.dirname(__file__), '..', 'spendly.db')`  
  (puts `spendly.db` in the project root, one level above `database/`)
- Set `conn.row_factory = sqlite3.Row`
- Execute `PRAGMA foreign_keys = ON`
- Return the connection

**`init_db()`**
- Call `get_db()` to open a connection
- Execute `CREATE TABLE IF NOT EXISTS users` with exact schema:
  - `id INTEGER PRIMARY KEY AUTOINCREMENT`
  - `name TEXT NOT NULL`
  - `email TEXT UNIQUE NOT NULL`
  - `password_hash TEXT NOT NULL`
  - `created_at TEXT DEFAULT (datetime('now'))`
- Execute `CREATE TABLE IF NOT EXISTS expenses` with exact schema:
  - `id INTEGER PRIMARY KEY AUTOINCREMENT`
  - `user_id INTEGER NOT NULL REFERENCES users(id)`
  - `amount REAL NOT NULL`
  - `category TEXT NOT NULL`
  - `date TEXT NOT NULL`
  - `description TEXT`
  - `created_at TEXT DEFAULT (datetime('now'))`
- `conn.commit()` then `conn.close()`

**`seed_db()`**
- Call `get_db()`
- Check `SELECT COUNT(*) FROM users` — if count > 0, close and return early
- Insert demo user with parameterized query:
  - name: `Demo User`, email: `demo@spendly.com`
  - password_hash: `generate_password_hash("demo123")`
- Retrieve the new user's `id` via `cursor.lastrowid`
- Insert 8 sample expenses (all linked to demo user, parameterized):

| amount | category | date (use current month) | description |
|--------|----------|--------------------------|-------------|
| 450.00 | Food | 2026-08-01 | Grocery run |
| 120.00 | Transport | 2026-08-02 | Auto rickshaw |
| 1200.00 | Bills | 2026-08-03 | Electricity bill |
| 300.00 | Health | 2026-08-05 | Pharmacy |
| 850.00 | Entertainment | 2026-08-07 | Movie tickets |
| 2200.00 | Shopping | 2026-08-10 | Clothing |
| 180.00 | Food | 2026-08-12 | Restaurant lunch |
| 500.00 | Other | 2026-08-15 | Miscellaneous |

- `conn.commit()` then `conn.close()`

---

### `app.py`

Add import at the top:
```python
from database.db import get_db, init_db, seed_db
```

Add startup block **before** the route definitions, after `app = Flask(__name__)`:
```python
with app.app_context():
    init_db()
    seed_db()
```

---

## Rules to Enforce

- All SQL uses `?` placeholders — no f-strings or `.format()` in any query
- `PRAGMA foreign_keys = ON` is set inside `get_db()` on every connection
- `amount` stored as `REAL`, dates as `TEXT` in `YYYY-MM-DD`
- `seed_db()` guard prevents duplicate inserts on every server restart

---

## Verification

1. Run `python app.py` — server should start on port 5001 with no errors
2. Confirm `spendly.db` file is created in the project root
3. Open the DB with any SQLite viewer (or `sqlite3 spendly.db`):
   - `.tables` → should show `users` and `expenses`
   - `SELECT * FROM users;` → 1 row: Demo User / demo@spendly.com
   - `SELECT * FROM expenses;` → 8 rows across 7 categories
4. Restart the server — `seed_db()` should not add duplicate rows
5. Verify foreign key enforcement:
   ```sql
   INSERT INTO expenses (user_id, amount, category, date) VALUES (999, 100, 'Food', '2026-08-01');
   -- Should fail with FOREIGN KEY constraint error
   ```
