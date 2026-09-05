# Setup

## Core Idea
Installing and configuring Wagtail Transfer involves four steps: pip install, adding to INSTALLED_APPS, wiring URLs, and setting the secret keys.

## Frameworks Introduced
- **WAGTAILTRANSFER_SECRET_KEY / SOURCES**: A two-layer auth model — each site has a global secret key, and each source entry in SOURCES must share that key to authenticate transfers. Prevents unauthorized content retrieval.

## Key Concepts
- **INSTALLED_APPS**: Add `wagtail_transfer` to your project's installed apps.
- **URL routing**: Mount `wagtail_transfer.urls` under `wagtail-transfer/` in your top-level urls.py, above `wagtail_urls`.
- **SOURCES dictionary**: Maps source site names to `{BASE_URL, SECRET_KEY}` entries.
- **SECRET_KEY**: Must match between source and destination for each source entry.

## Key Takeaways
1. `pip install wagtail-transfer` is the first step.
2. Always store SECRET_KEYs in environment variables, not hardcoded in settings.
3. The source site's `WAGTAILTRANSFER_SOURCES[name]['SECRET_KEY']` must match the source site's own `WAGTAILTRANSFER_SECRET_KEY`.
4. URL pattern must be registered before `wagtail_urls`.

## Connects To
- **Ch 02**: Basic Usage — once setup is complete, start importing
- **Ch 04**: Settings — additional configuration beyond the basics
