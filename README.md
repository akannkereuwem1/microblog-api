## Microblog API

**Microblog API** is a modern Flask-based REST API for building microblogging applications.  
It provides a complete, production-ready backend that you can plug any frontend into (SPA, mobile app, or server‑rendered UI).

The project is maintained by **Akan Nkereuwem** and is designed to be:

- **Opinionated but practical**: sensible defaults, clear patterns.
- **API‑first**: documented schema and interactive docs out of the box.
- **Educational**: a great backend to pair with a frontend framework when learning.

---

### Features

- **User management**
  - Registration, login, logout
  - Password hashing and verification
  - Profile fields (`about_me`, avatar via Gravatar)
  - “Me” endpoints for the authenticated user

- **Authentication & authorization**
  - Access and refresh tokens (JWT‑backed)
  - Configurable token lifetimes
  - Optional “disable auth” mode for local development

- **Microblogging**
  - Create, update, delete posts
  - Per‑user post listing
  - Personalized feed from followed users
  - Pagination with `limit`, `offset`, and time‑based cursors

- **Social graph**
  - Follow and unfollow users
  - List followers and following

- **API documentation**
  - Auto‑generated OpenAPI docs via `APIFairy`
  - Interactive documentation UI at `/docs`

- **Operational capabilities**
  - Alembic migrations
  - Docker and Docker Compose support
  - Heroku‑style configuration via environment variables

---

### Architecture Overview

- **Framework**: `Flask`
- **Database layer**: `alchemical` / SQLAlchemy models in `api/models.py`
- **Serialization & validation**: Marshmallow schemas in `api/schemas.py`
- **Auth**: `flask-httpauth` with:
  - Basic auth for credential exchange
  - Token auth for protected endpoints
- **Docs**: `APIFairy` generates OpenAPI-compatible docs and UI
- **Configuration**: centralized in `config.py`, driven by environment variables

Key entry points:

- `microblog.py` – WSGI/Flask entrypoint and CLI `db upgrade` command.
- `api/app.py` – Flask application factory and blueprint registration.
- `api/users.py`, `api/posts.py`, `api/tokens.py` – main resource endpoints.

---

### Getting Started (Local Development)

#### 1. Clone the repository

```bash
git clone https://github.com/akannkereuwem/microblog-api.git
cd microblog-api
cp .env.example .env
```

Edit `.env` and set values for any environment variables you care about (see **Configuration** below).

#### 2. Create and activate a virtual environment

```bash
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

#### 3. Create and migrate the database

```bash
alembic upgrade head
```

Optionally seed the database with fake data:

```bash
flask fake users 10
flask fake posts 100
```

#### 4. Run the development server

```bash
flask run
```

By default the API will be available at `http://localhost:5000`, and the interactive documentation at `http://localhost:5000/docs`.

---

### Running with Docker

You can run Microblog API fully containerized using the provided `Dockerfile` and `docker-compose.yml`.

#### 1. Environment configuration

Ensure you have a `.env` file next to `docker-compose.yml` with any overrides you need (for example, email or OAuth2 settings).

#### 2. Build and start the stack

```bash
docker-compose up -d
```

The service exposes the API on:

- `http://localhost:5000` by default
- Or on `http://localhost:<MICROBLOG_API_PORT>` if you set `MICROBLOG_API_PORT` in `.env`

Populate the database with fake data:

```bash
docker-compose run --rm microblog-api \
  bash -c "flask fake users 10 && flask fake posts 100"
```

Shut everything down:

```bash
docker-compose down
```

---

### Configuration

Configuration is provided through environment variables (directly in your environment or via `.env`).  
The most important options (see `api/__init__.py` for more details) include:

- **Core**
  - `SECRET_KEY` – secret key for signing tokens (default: `top-secret!`)
  - `DATABASE_URL` – SQLAlchemy database URL (default: `sqlite:///db.sqlite`)
  - `SQL_ECHO` – echo SQL statements for debugging (`true`/`false`)

- **Authentication**
  - `DISABLE_AUTH` – if set, token auth is bypassed and user with `id=1` is assumed
  - `ACCESS_TOKEN_MINUTES` – access token lifetime in minutes (default: `15`)
  - `REFRESH_TOKEN_DAYS` – refresh token lifetime in days (default: `7`)
  - `REFRESH_TOKEN_IN_COOKIE` – return refresh token in a secure cookie (default: `yes`)
  - `REFRESH_TOKEN_IN_BODY` – also return refresh token in the response body

- **Password reset**
  - `RESET_TOKEN_MINUTES` – password reset token lifetime in minutes (default: `15`)
  - `PASSWORD_RESET_URL` – base URL used in password reset links (default: `http://localhost:3000/reset`)

- **CORS & docs**
  - `USE_CORS` – enable/disable CORS support (default: `yes`)
  - `DOCS_UI` – documentation UI (`swagger_ui`, `redoc`, `rapidoc`, `elements`; default: `elements`)

- **Mail**
  - `MAIL_SERVER`, `MAIL_PORT`, `MAIL_USE_TLS`
  - `MAIL_USERNAME`, `MAIL_PASSWORD`
  - `MAIL_DEFAULT_SENDER`

- **OAuth2**
  - `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`
  - `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`
  - `OAUTH2_REDIRECT_URI` – e.g. `http://localhost:3000/oauth2/{provider}/callback`

---

### API Overview

The full, up‑to‑date reference is available at the `/docs` endpoint.  
At a high level, the API surface includes:

- **Tokens**
  - `POST /api/tokens` – obtain access and refresh tokens
  - `PUT /api/tokens` – refresh tokens
  - `POST /api/tokens/reset` – request password reset
  - `PUT /api/tokens/reset` – reset password with token

- **Users**
  - `POST /api/users` – register a new user
  - `GET /api/users` – list users (paginated)
  - `GET /api/users/<id>` – get user by id
  - `GET /api/users/<username>` – get user by username
  - `GET /api/me` – get current user
  - `PUT /api/me` – update current user profile / password
  - Follow/followers endpoints under `/api/me/following`, `/api/me/followers`, `/api/users/<id>/followers`, `/api/users/<id>/following`

- **Posts**
  - `POST /api/posts` – create a post
  - `GET /api/posts` – list posts (paginated)
  - `GET /api/posts/<id>` – retrieve a post
  - `PUT /api/posts/<id>` – update a post (author only)
  - `DELETE /api/posts/<id>` – delete a post (author only)
  - `GET /api/users/<id>/posts` – posts from a specific user
  - `GET /api/feed` – personalized feed for the authenticated user

All error responses follow a consistent JSON format with `code`, `message`, `description`, and, for validation errors, an `errors` field.

---

### Testing

The project ships with a comprehensive test suite under `tests/`.

With a virtual environment active:

```bash
pip install -r requirements-dev.txt
pytest
```

You can also run tests with coverage:

```bash
pytest --cov=api
```

---

### Deployment Notes

- **Gunicorn** is included in the dependencies and can be used as the WSGI server:

  ```bash
  gunicorn -w 4 -b 0.0.0.0:5000 microblog:app
  ```

- The app is designed to work well in environments like Heroku or any platform that:
  - Exposes a `PORT` environment variable
  - Provides a managed PostgreSQL instance (configure via `DATABASE_URL`)

Ensure that:

- Environment variables for the database, mail, and OAuth2 providers are set.
- `SECRET_KEY` is a secure, random value in production.

---

### License

This project is licensed under the MIT License.  
Copyright (c) 2021 **Akan Nkereuwem**
