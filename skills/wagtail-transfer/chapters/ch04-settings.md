# Settings and Hooks

## Core Idea
Wagtail Transfer is configured via Django settings (SECRET_KEY, SOURCES, UPDATE_RELATED_MODELS, LOOKUP_FIELDS, NO_FOLLOW_MODELS, FOLLOWED_REVERSE_RELATIONS, CHOOSER_API_PROXY_TIMEOUT) and extended via hooks (register_field_adapters, register_custom_serializers).

## Frameworks Introduced
- **SECRET_KEY / SOURCES**: Two-layer auth — global key per site, per-source key for cross-site auth.
- **UPDATE_RELATED_MODELS**: List of model labels that should be updated on re-import (e.g. images).
- **LOOKUP_FIELDS**: Dict mapping model labels to field lists for identity matching instead of UUIDs.
- **NO_FOLLOW_MODELS**: Models that should NOT be recursively imported (default: Page, ContentType).
- **FOLLOWED_REVERSE_RELATIONS**: List of (model, relation, track_deletions) tuples for following reverse relations.
- **CHOOSER_API_PROXY_TIMEOUT**: Timeout in seconds for API calls to browse source page tree (default: 5).

## Key Concepts
- **Field adapters**: Classes that serialize/deserialize field data during export/import. Register custom adapters via `register_field_adapters` hook.
- **Custom serializers**: Override PageSerializer for models that need custom export logic (e.g. excluding fields). Register via `register_custom_serializers` hook.
- **track_deletions**: When True in FOLLOWED_REVERSE_RELATIONS, destination-side objects not in source are deleted.

## Reference Tables

### Default Settings

| Setting | Default |
|---|---|
| `WAGTAILTRANSFER_LOOKUP_FIELDS` | `{'taggit.tag': ['slug'], 'wagtailcore.locale': ['language_code'], 'contenttypes.contenttype': ['app_label', 'model']}` |
| `WAGTAILTRANSFER_NO_FOLLOW_MODELS` | `['wagtailcore.page', 'contenttypes.contenttype']` |
| `WAGTAILTRANSFER_FOLLOWED_REVERSE_RELATIONS` | `[('wagtailimages.image', 'tagged_items', True)]` |
| `WAGTAILTRANSFER_CHOOSER_API_PROXY_TIMEOUT` | `5` |

## Anti-patterns
- **Overriding ContentType LOOKUP_FIELDS**: Can cause unexpected behavior; default is carefully tuned.
- **Using NO_FOLLOW_MODELS for multi-table subclasses**: These settings don't accept subclass models — only base models.

## Key Takeaways
1. SECRET_KEYs must match between source and destination for each source entry.
2. UPDATE_RELATED_MODELS controls which referenced objects are updated (vs just ensuring they exist).
3. LOOKUP_FIELDS is for models with natural unique keys (e.g. authors by name).
4. NO_FOLLOW_MODELS prevents recursive import of Page and similar models.
5. Custom serializers let you exclude fields from export.

## Connects To
- **Ch 03**: How It Works — settings directly control the ID Mapping and reference-finding behavior
- **Ch 02**: Basic Usage — settings shape what gets imported and how
