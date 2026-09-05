---
name: wagtail-cms
description: "Knowledge base from Wagtail CMS documentation. Use when building with Wagtail, applying its page/tree architecture, StreamField content modeling, search integration, admin customization, API design, or deploying Wagtail sites."
---

<!-- argument-hint: [topic, framework name, or chapter number] -->

# Wagtail CMS Documentation
**Author**: Wagtail community (Torchbox) | **Sources**: 404 docs files | **Chapters**: 20 | **Generated**: 2026-09-03

## How to Use This Skill

- **Without arguments** — load core frameworks for reference
- **With a topic** — ask about `StreamField`, `page model`, `search`, or another indexed topic; I find and read the relevant chapter
- **With chapter** — ask for `ch04`; I load that specific chapter
- **Browse** — ask "what chapters do you have?" to see the full index

When you ask about a topic not covered in Core Frameworks below, I will read
the relevant chapter file before answering.

---

## Core Frameworks & Mental Models

### Page Tree Architecture
Wagtail organizes all pages in a **tree structure** where every page has exactly one parent. Use `Page` as the base class for all content types. The tree enables:
- Automatic URL routing via `get_urlpath()`
- Hierarchical queries via `PageQuerySet`
- Draft/public workflow with `go_live_at` / `expire_at`
- Trail breadcrumb navigation

**Think of the page tree as your site's database schema.** Every content type is a Django model that extends `Page`. You never query `Page.objects.all()` directly — use `LivePageQuerySet` or filter by specific page types.

### StreamField: Content Modeling
StreamField provides **ordered, heterogeneous block content** within a single field. Unlike traditional CMS fields, blocks are stacked and can be repeated, nested, and reordered by editors.

**Use StreamField when**: content has variable structure (blog posts with embeds, galleries, code blocks).
**Use non-StreamField when**: content is rigidly structured (simple landing pages with fixed fields).

Block hierarchy: `StreamBlock` → `StructBlock` (grouped fields) → leaf blocks (`CharBlock`, `ImageBlock`, etc.). Nesting is limited to 10 levels deep.

### Chooser Panels Pattern
Wagtail uses **chooser widgets** for ForeignKey/M2M relationships to pages, images, and documents. The pattern:
1. Define the relationship as `ForeignKey` with `on_delete` strategy
2. Use `InlinePanel` or field panel with appropriate chooser
3. The chooser provides search, browse, and preview in the admin

### WagtailSettings: Global Configuration
`WagtailSettings` (via `wagtail.models.Site`) stores site-wide configuration as a single-instance model. Access via `request.site` in templates or `Site.objects.get(is_default_site=True)` in code.

### Hook System
Wagtail's **hook system** (`@hooks.register`) allows extending admin views, page serving, and more without modifying core code. Hooks are function decorators that run at specific points in the request lifecycle.

- `before_serve_page` — modify responses before page delivery
- `register_admin_url` — add custom admin views
- `register_page_inspector` — add data to page inspection panels
- `register_rich_text_features` — extend Draftail editor

### Search Integration Pattern
Wagtail provides a **pluggable search backend** (Elasticsearch, PostgreSQL FTS, etc.) with automatic indexing. The flow:
1. **Register** searchable fields via `search_fields` on models
2. **Index** happens automatically on `save()` (or via `update_index` command)
3. **Query** via `SearchBackend.search()` with boost and filtering

Use `index.SearchField` for full-text, `index.RelatedFields` for related model fields, and `index.FilterField` for faceted filtering.

### Permission Model
Wagtail's permission system combines **Django groups** with **Wagtail-specific permissions**:
- `GroupPagePermission` — page-level CRUD access per group
- `CollectionPermission` — image/document access per collection
- `GroupCollectionPermission` — fine-grained media access

**Use groups, not individual users.** Assign roles (Editors, Moderators, Admins) to groups, then add users to groups.

### Image Renditions
Images are served through **renditions** — pre-generated thumbnails at specific sizes. Define renditions in the model:
```python
class BlogPage(Page):
    body = StreamField([
        ('image', ImageChooserBlock()),
    ])
```
Templates access renditions via `image.get_rendition('width-800')` or named renditions.

### Deployment Pattern
Wagtail is a standard Django app. Deploy with:
1. Static files via `whitenoise` or CDN
2. Media via S3/Cloudflare R2 with `django-storages`
3. Database migrations via `manage.py migrate`
4. WSGI server (Gunicorn/uwsgi) behind a reverse proxy

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-getting-started.md) | Getting Started | Installation, Django integration, project structure |
| [ch02](chapters/ch02-tutorial.md) | Tutorial: Portfolio Site | Page creation, StreamField basics, template rendering |
| [ch03](chapters/ch03-pages.md) | Pages | Page model, tree structure, page types, serving |
| [ch04](chapters/ch04-streamfield.md) | StreamField | Block types, nesting, migration, validation |
| [ch05](chapters/ch05-images.md) | Images | Upload, renditions, focal points, templates |
| [ch06](chapters/ch06-documents.md) | Documents | Custom models, storage backends, serving |
| [ch07](chapters/ch07-search.md) | Search | Backends, indexing, querying, autocomplete |
| [ch08](chapters/ch08-snippets.md) | Snippets | Registration, mixins, admin views, rendering |
| [ch09](chapters/ch09-templates.md) | Templates | Tags, filters, caching, page rendering |
| [ch10](chapters/ch10-permissions.md) | Permissions | Groups, roles, page-level access, collections |
| [ch11](chapters/ch11-deployment.md) | Deployment | WSGI, static files, cloud, Fly.io |
| [ch12](chapters/ch12-extending-admin.md) | Extending Admin | ViewSets, hooks, forms, generic views |
| [ch13](chapters/ch13-extending-frontend.md) | Extending Frontend | Draftail, JS, CSS, panels |
| [ch14](chapters/ch14-api.md) | API | WagtailAPIViewSet, Django Ninja, endpoints |
| [ch15](chapters/ch15-headless.md) | Headless | Architecture, preview, limitations |
| [ch16](chapters/ch16-customization.md) | Customization | Base models, admin templates, user models |
| [ch17](chapters/ch17-performance.md) | Performance | Caching, pagination, queries, lazy loading |
| [ch18](chapters/ch18-testing.md) | Testing | WagtailPageTests, fixtures, assertions |
| [ch19](chapters/ch19-contributing.md) | Contributing | Dev setup, standards, PR process |
| [ch20](chapters/ch20-advanced-topics.md) | Advanced Topics | Tags, i18n, personalization, privacy |

## Topic Index

- **Page model** → ch03, ch04, ch16
- **StreamField** → ch04, ch14, ch16
- **Block types** → ch04
- **Image renditions** → ch05
- **Document management** → ch06
- **Search backends** → ch07
- **Search indexing** → ch07
- **Snippets** → ch08
- **Template tags** → ch09
- **Template filters** → ch09
- **Permissions** → ch10
- **Groups** → ch10
- **Deployment** → ch11
- **Static files** → ch11
- **Admin views** → ch12
- **ViewSets** → ch12
- **Hooks** → ch12, ch13
- **Draftail** → ch13
- **Client-side JS** → ch13
- **API endpoints** → ch14
- **Django Ninja** → ch14
- **Headless** → ch15
- **Preview** → ch15
- **Custom page models** → ch16
- **Admin templates** → ch16
- **Caching** → ch17
- **Pagination** → ch17
- **Testing** → ch18
- **Contributing** → ch19
- **Tags** → ch20
- **i18n** → ch20
- **Privacy** → ch20

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers the Wagtail CMS documentation only. For hands-on implementation in your specific project, combine with project-specific code review. For topics beyond the official docs (third-party packages, community plugins), ask the agent directly or check Wagtail's GitHub ecosystem.
