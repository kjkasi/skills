# Cheatsheet

## Setup Checklist
1. `pip install wagtail-transfer`
2. Add `wagtail_transfer` to INSTALLED_APPS
3. Wire URLs: `url(r'^wagtail-transfer/', include(wagtailtransfer_urls))` before wagtail_urls
4. Set WAGTAILTRANSFER_SOURCES and WAGTAILTRANSFER_SECRET_KEY
5. Store SECRET_KEYs in environment variables

## Settings Quick Reference

| Setting | Purpose | Default |
|---|---|---|
| SECRET_KEY | Authenticates this site as an import source | — (required) |
| SOURCES | Dict of source sites with BASE_URL + SECRET_KEY | — (required) |
| UPDATE_RELATED_MODELS | Models to update on re-import | `[]` |
| LOOKUP_FIELDS | Field-based identity matching | tag, locale, contenttype defaults |
| NO_FOLLOW_MODELS | Skip recursive import | `['wagtailcore.page', 'contenttypes.contenttype']` |
| FOLLOWED_REVERSE_RELATIONS | Follow reverse relations | `[('wagtailimages.image', 'tagged_items', True)]` |
| CHOOSER_API_PROXY_TIMEOUT | API call timeout (seconds) | `5` |

## Decision Rules

- **Existing content on both sites?** → Run `preseed_transfer_table` first
- **Want images updated on re-import?** → Add `wagtailimages.image` to UPDATE_RELATED_MODELS
- **Model has natural unique key?** → Use LOOKUP_FIELDS instead of UUID matching
- **Don't want cascading page imports?** → Page is in NO_FOLLOW_MODELS by default
- **Using tags on non-Page models?** → Add to FOLLOWED_REVERSE_RELATIONS
- **Testing locally with two servers?** → Increase CHOOSER_API_PROXY_TIMEOUT

## Common Commands

```bash
# Standard pre-seeding for all common models
./manage.py preseed_transfer_table auth wagtailcore wagtailimages.image wagtaildocs

# Pre-seed specific page range
./manage.py preseed_transfer_table wagtailcore.page --range=1-199

# Pre-seed all models in an app
./manage.py preseed_transfer_table wagtaildocs
```

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| Duplicate objects on import | No pre-seeded IDs | Run preseed_transfer_table |
| Broken links after import | Referenced model in NO_FOLLOW_MODELS | Remove from list or import manually |
| Import rejected | SECRET_KEY mismatch | Ensure source and destination keys match |
| Timeout browsing source | Slow source server | Increase CHOOSER_API_PROXY_TIMEOUT |
| Objects not updating | Not in UPDATE_RELATED_MODELS | Add model to the list |
