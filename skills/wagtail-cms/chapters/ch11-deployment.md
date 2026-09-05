# Chapter 11: Deploying Wagtail

## Core Idea
Wagtail is a Django application; deploying it follows Django deployment practices — a WSGI server, static file handling, database configuration, and optionally cloud storage for media. Hosting providers range from Wagtail-specific platforms to raw infrastructure.

## Frameworks Introduced
- **Gunicorn**: WSGI HTTP server for running Wagtail in production.  - When to use: Any production deployment.  - How: Add `gunicorn` to requirements, configure `CMD` in Dockerfile: `CMD ["gunicorn", "--bind", ":8000", "--workers", "1", "mysite.wsgi"]`
- **WhiteNoise**: Serves static files efficiently in production without a separate CDN.  - When to use: When you don't have a reverse proxy or CDN for static assets.  - How: Add to `MIDDLEWARE`, set `STORAGES["staticfiles"]["BACKEND"]` to `whitenoise.storage.CompressedManifestStaticFilesStorage`.
- **django-storages (S3)**: Offloads user-uploaded files to cloud storage (e.g., Backblaze B2, Amazon S3).  - When to use: Production sites where media files shouldn't live on the app server.  - How: Set `STORAGES["default"]["BACKEND"]` to `storages.backends.s3boto3.S3Boto3Storage`, configure `AWS_*` env vars.
- **dj-database-url**: Simplifies database configuration via a single `DATABASE_URL` environment variable.  - When to use: Any deployment using PostgreSQL or similar.  - How: `DATABASES = {"default": dj_database_url.config(conn_max_age=600)}`

## Key Concepts
- **WSGI vs ASGI**: Wagtail supports both, but recommends WSGI since it doesn't natively use async views.
- **ManifestStaticFilesStorage**: Ensures different versions of static files get distinct URLs, preventing cache-related breakage after Wagtail upgrades.
- **`MEDIA_ROOT` / `MEDIA_URL`**: Default local storage path for user-uploaded files (images, documents).
- **`WAGTAILDOCS_SERVE_METHOD`**: Controls how documents are served — `serve_view` (default, enforces permissions), `direct`, or `redirect`.
- **`AWS_S3_FILE_OVERWRITE`**: Must be `False` when using django-storages S3 backends to prevent duplicate filename collisions.
- **`WAGTAIL_REDIRECTS_FILE_STORAGE`**: Set to `"cache"` for cloud deployments to avoid filesystem dependencies.

## Mental Models
1. **Three-layer deployment stack**: WSGI server (Gunicorn) → Django/Wagtail app → Database + File storage. Each layer can be independently scaled or swapped.
2. **Static vs Media separation**: Static files (CSS/JS) are served by WhiteNoise or a CDN. Media files (user uploads) go to S3-compatible storage. Never mix these concerns.
3. **Environment-based configuration**: Use `os.environ` for secrets (`SECRET_KEY`, `AWS_*` keys) and `DJANGO_SETTINGS_MODULE` to swap between `base.py` and `production.py` settings.
4. **Fly.io mental model**: One machine, one worker for low-memory plans. Use `fly deploy --ha=false` to avoid double costs.

## Anti-patterns
- **Serving static files through Django in production**: Causes performance bottlenecks. Always use WhiteNoise, a CDN, or a reverse proxy.
- **Committing `.env` files**: Exposes secrets. Add `.env*` to `.gitignore` and `.dockerignore`.
- **Using `DEBUG = True` in production**: Exposes sensitive information and disables security features.
- **Not setting `ALLOWED_HOSTS`**: Django will refuse to serve requests if the host header doesn't match.
- **Running multiple Gunicorn workers on low-memory plans**: Exceeds memory limits. Use `--workers 1` on shared-cpu-1x instances.

## Code Examples
```python
# production.py — Minimal production settings
import os
import dj_database_url
from .base import *

DEBUG = False
DATABASES = {"default": dj_database_url.config(conn_max_age=600)}
SECRET_KEY = os.environ["SECRET_KEY"]
SECURE_PROXY_SSL_HEADER = ("HTTP_X_FORWARDED_PROTO", "https")
SECURE_SSL_REDIRECT = True
ALLOWED_HOSTS = os.getenv("DJANGO_ALLOWED_HOSTS", "*").split(",")
CSRF_TRUSTED_ORIGINS = os.getenv("DJANGO_CSRF_TRUSTED_ORIGINS", "").split(",")

MIDDLEWARE.append("whitenoise.middleware.WhiteNoiseMiddleware")
STORAGES["staticfiles"]["BACKEND"] = "whitenoise.storage.CompressedManifestStaticFilesStorage"

if "AWS_STORAGE_BUCKET_NAME" in os.environ:
    STORAGES["default"]["BACKEND"] = "storages.backends.s3boto3.S3Boto3Storage"
```
- **What it demonstrates**: Production-ready Django settings with WhiteNoise and optional S3 storage.

```toml
# fly.toml — Fly.io deployment config
[deploy]
  release_command = "python manage.py migrate --noinput"

[env]
  PORT = "8000"

[http_service]
  internal_port = 8000
  force_https = true

[[statics]]
  guest_path = "/code/static"
  url_prefix = "/static/"
```
- **What it demonstrates**: Fly.io configuration with auto-migration on deploy and static file serving.

## Reference Tables

| Hosting Tier | Examples | You Handle |
|---|---|---|
| Wagtail-level | CodeRed Cloud, Divio | Nothing — turnkey |
| Python-level | Fly.io | WSGI, file storage, database |
| Infrastructure-level | AWS, Azure, DigitalOcean | Linux, reverse proxy, everything |

| Dependency | Purpose |
|---|---|
| `gunicorn` | WSGI HTTP server |
| `psycopg[binary]` | PostgreSQL adapter |
| `dj-database-url` | Database URL parsing |
| `whitenoise` | Static file serving |
| `django-storages[s3]` | Cloud media storage |

## Worked Example
Deploy a Wagtail portfolio to Fly.io with Backblaze B2:
1. Create a Backblaze B2 bucket (public, disable encryption).
2. Generate application keys (Read and Write access).
3. Create `.env.production` with `AWS_STORAGE_BUCKET_NAME`, `AWS_S3_ENDPOINT_URL`, `AWS_S3_REGION_NAME`, `AWS_S3_ACCESS_KEY_ID`, `AWS_S3_SECRET_ACCESS_KEY`, `DJANGO_ALLOWED_HOSTS`, `DJANGO_CSRF_TRUSTED_ORIGINS`, `DJANGO_SETTINGS_MODULE`.
4. Run `fly launch` → configure region, CPU, database (Fly Postgres).
5. Modify `Dockerfile` CMD to `gunicorn` with 1 worker.
6. Add `release_command` to `fly.toml` for migrations.
7. Import secrets: `flyctl secrets import < .env.production`.
8. Deploy: `fly deploy --ha=false`.
9. Create superuser via `flyctl ssh console`.

## Key Takeaways
1. Wagtail deployment is Django deployment — follow Django's deployment checklist.
2. Always use `ManifestStaticFilesStorage` to prevent stale admin JS/CSS after upgrades.
3. Separate static files (WhiteNoise/CDN) from media files (S3/cloud storage) for security and performance.
4. Use environment variables for all secrets; never commit `.env` files.
5. The audit log and `LOGGING` config are critical for production monitoring.

## Connects To
- **Ch 14 (API)**: API endpoints share the same WSGI server and deployment infrastructure.
- **Ch 15 (Headless)**: Headless deployments often add a separate frontend (Next.js/Nuxt) alongside the Wagtail API backend.
