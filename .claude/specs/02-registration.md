# Spec: Registration

## Overview
Wire up the registration form so new users can create a Spendly account. This step adds the `POST /register` handler that validates input, hashes the password, inserts a new row into the `users` table, and redirects to the login page on success. The `GET /register` route and `register.html` template already exist — only the form-processing logic is missing.

## Depends on
Step 01 — Database Setup (users table and `get_db()` must exist).

## Routes
- `GET /register` — already implemented; renders `register.html`
- `POST /register` — **new**; processes registration form — public

## Database changes
No new tables or columns. The `users` table from Step 01 is sufficient.

A new helper must be added to `database/db.py`:

```
create_user(name, email, password_hash) → None
```

Inserts one row into `users`. Raises `sqlite3.IntegrityError` if the email is already taken (UNIQUE constraint).

## Templates
- **Modify:** `templates/register.html`
  - Change `action="/register"` → `action="{{ url_for('register') }}"`
  - Ensure `value="{{ name }}"` and `value="{{ email }}"` are set on inputs so the form re-populates on validation error

## Files to change
- `app.py` — add `POST /register` handler; update imports if needed
- `database/db.py` — add `create_user()` helper
- `templates/register.html` — fix hardcoded action URL; add `value` attributes

## Files to create
None.

## New dependencies
No new dependencies.

## Rules for implementation
- No SQLAlchemy or ORMs
- Parameterised queries only — never f-strings in SQL
- Hash passwords with `werkzeug.security.generate_password_hash` — never store plaintext
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- `create_user()` belongs in `database/db.py`, not inline in the route
- On duplicate email: catch `sqlite3.IntegrityError` in the route and re-render the form with `error="An account with that email already exists."`
- On any validation failure: re-render `register.html` with the `error` variable and the previously entered `name` and `email` so the user does not lose their input
- Validate in the route (not in JS): name non-empty, email non-empty, password at least 8 characters
- On success: `redirect(url_for('login'))` — do not start a session (that is Step 3)
- Use `abort(400)` only for malformed requests; prefer re-rendering with `error` for user-facing validation failures
- The `POST /register` route function must be named `register_post` (or the existing `register` function must handle both methods via `methods=["GET", "POST"]`)

## Definition of done
- [ ] Submitting the form with valid data creates a new row in `users` with a hashed password
- [ ] After successful registration the user is redirected to `/login`
- [ ] Submitting with an email that already exists re-renders the form with an error message and does not create a duplicate row
- [ ] Submitting with a password shorter than 8 characters re-renders the form with a validation error
- [ ] Submitting with an empty name or email re-renders the form with a validation error
- [ ] The name and email fields are repopulated after a failed submission
- [ ] The form action uses `url_for('register')`, not a hardcoded string
- [ ] The demo user created by `seed_db()` cannot be re-registered (duplicate email rejected)
- [ ] App starts without errors on `python app.py`
