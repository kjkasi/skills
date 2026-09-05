# Chapter 14: Wagtail API

## Core Idea
Wagtail provides two API versions: v2 (read-only, Django REST Framework) and v3 (read/write, Django Ninja with OpenAPI 3.1). The v3 API supports full CMS operations — creating, editing, deleting pages, managing revisions, and handling StreamField content programmatically.

## Frameworks Introduced
- **Django REST Framework (v2)**: Battle-tested read-only API for serving content to external clients.  - When to use: Headless sites, mobile apps, simple content consumption.  - How: Enable `wagtail.api.v2` in `INSTALLED_APPS`, configure `WAGTAILAPI_*` settings.
- **Django Ninja (v3)**: Type-hint-based API framework with OpenAPI schema generation.  - When to use: Full CMS operations via API, programmatic content management.  - How: Enable `wagtail.api.v3` in `INSTALLED_APPS`, mount at `/api/v3-preview/`.
- **WagtailAPIViewSet**: Configures which fields and operations are exposed for each content type.  - When to use: Customizing the v2 API's field exposure and permissions.  - How: Subclass `wagtail.api.v2.viewsets.PagesViewSet` or create custom viewsets.

## Key Concepts
- **`WAGTAILAPI_BASE_URL`**: Configures the base URL for API responses (used in `detail_url`, `html_url`).
- **`WAGTAILAPI_LIMIT_MAX`**: Caps the maximum `?limit` value for paginated list endpoints.
- **`WAGTAILAPI_SEARCH_ENABLED`**: Enables/disables the `?search` query parameter.
- **`WAGTAILAPI_RICH_TEXT_FORMAT`**: Controls rich text output format (`html` or `plain`).
- **Limit/offset pagination**: All list endpoints use `count` + `items` envelope with `?limit` and `?offset`.
- **RFC 7807 errors**: v3 API returns structured `application/problem+json` error responses.
- **`meta.type`**: Discriminator field in v3 API payloads identifying the concrete page/model type.
- **`api_fields`**: Declares which model fields are exposed in the API response.
- **`writable`**: Marks API fields as editable (v3 only).
- **Page actions**: v3 API endpoints for publish, unpublish, copy, move, revert, alias, and translation operations.
- **`expand_db_html`**: Converts Wagtail's internal rich text format to renderable HTML.

## Mental Models
1. **API as admin mirror**: The v3 API provides nearly identical operations to the Wagtail admin UI — create, edit, delete, publish, move, revert — all with the same permission and validation logic.
2. **Schema-driven development**: Django Ninja generates OpenAPI schemas from type hints. Define your response schema, and the docs/endpoint are automatic.
3. **Content type discrimination**: The `meta.type` field in v3 responses tells clients which concrete schema to use. Combine with union types for polymorphic responses.
4. **StreamField as a block list**: StreamField values are JSON arrays of `{type, id, value}` objects. PATCH replaces the entire list — no block-level patching.

## Anti-patterns
- **Exposing all model fields via API**: Always declare `api_fields` to control what's public. Default exposes nothing.
- **Ignoring `WAGTAILAPI_LIMIT_MAX`**: Without it, clients can request unbounded result sets, causing performance issues.
- **Using v2 for write operations**: v2 is read-only. Use v3 or build a custom Django Ninja/DRF endpoint for writes.
- **Assuming StreamField PATCH merges blocks**: PATCH replaces the entire StreamField value. Send the complete block list.
- **Not authenticating write requests**: v3 write endpoints require authentication and specific permissions (add, change, publish).

## Code Examples
```python
# v3 API quick start
# settings.py
INSTALLED_APPS = [
    ...
    'wagtail.api.v3',
    ...
]

# urls.py
from wagtail.api.v3.urls import api
urlpatterns = [
    path("api/v3-preview/", api.urls),
    path("", include(wagtail_urls)),
]
```
- **What it demonstrates**: Enabling the v3 API with auto-generated OpenAPI documentation.

```python
# Django Ninja custom endpoint with page schemas
from ninja import Field, ModelSchema, NinjaAPI
from wagtail.models import Page

api = NinjaAPI()

class BasePageSchema(ModelSchema):
    url: str = Field(None, alias="get_url")
    content_type: str

    @staticmethod
    def resolve_content_type(page: Page) -> str:
        return page.specific_class._meta.model_name

    class Meta:
        model = Page
        fields = ["id", "title", "slug"]

@api.get("/pages/", response=list[BasePageSchema])
def list_pages(request, child_of: int = None):
    if child_of:
        return get_object_or_404(Page, id=child_of).get_children().live().public()
    return Page.objects.live().public().exclude(id=1)
```
- **What it demonstrates**: Building a custom API endpoint with Django Ninja using Wagtail page models.

```python
# Rich text resolver for API output
from wagtail.rich_text import expand_db_html

class HomePageSchema(BasePageSchema, ModelSchema):
    content_type: Literal["homepage"]
    body: str

    @staticmethod
    def resolve_body(page: HomePage, context) -> str:
        return expand_db_html(page.body)
```
- **What it demonstrates**: Converting Wagtail's internal rich text format to HTML for API consumers.

```sh
# v3 API: Create and publish a page
curl -X POST "https://example.com/api/v3/pages/" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "meta": {"type": "blog.BlogPage", "parent_id": 3, "action": "publish"},
    "title": "Example",
    "slug": "example",
    "body": [
      {"type": "heading", "id": "abc-123", "value": "Hello"},
      {"type": "paragraph", "id": "def-456", "value": "World"}
    ]
  }'
```
- **What it demonstrates**: Creating a page with StreamField content via the v3 API.

## Reference Tables

| v3 Endpoint | Method | Permission | Description |
|---|---|---|---|
| `/pages/` | GET | Anonymous | List pages |
| `/pages/` | POST | add | Create page |
| `/pages/{id}/` | GET | Anonymous | Page detail |
| `/pages/{id}/` | PATCH | change | Edit page |
| `/pages/{id}/` | DELETE | change | Delete page |
| `/pages/{id}/actions/publish/` | POST | publish | Publish page |
| `/pages/{id}/actions/unpublish/` | POST | publish | Unpublish page |
| `/pages/{id}/actions/copy/` | POST | add | Copy page |
| `/pages/{id}/actions/move/` | POST | change | Move page |
| `/pages/{id}/actions/revert/` | POST | change | Revert to revision |
| `/pages/{id}/revisions/` | GET | change | List revisions |

| Query Parameter | Endpoint | Description |
|---|---|---|
| `?type=model.Model` | `/pages/` | Filter by page type |
| `?child_of=N` | `/pages/` | Direct children of page N |
| `?descendant_of=N` | `/pages/` | All descendants of page N |
| `?locale=fr` | `/pages/` | Filter by locale |
| `?search=query` | `/pages/` | Full-text search |
| `?order=field` | `/pages/` | Sort (prefix `-` for desc) |
| `?limit=N&offset=N` | All lists | Pagination |
| `?version=draft` | `/pages/{id}/` | Latest draft revision |

## Worked Example
Build a headless blog API:
1. Enable `wagtail.api.v3` in `INSTALLED_APPS`.
2. Mount at `/api/v3-preview/` in `urls.py`.
3. Define `api_fields` on `BlogPage` model: `api_fields = ["intro", "body", "featured_image"]`.
4. Create `BlogPageSchema` inheriting from `BasePageSchema` with `content_type: Literal["blogpage"]`.
5. Add resolvers for rich text (`expand_db_html`) and images (`get_renditions()`).
6. Test at `/api/v3-preview/docs/` — the OpenAPI viewer shows all endpoints.
7. Create a page: `POST /api/v3/pages/` with `meta.type = "blog.BlogPage"`, `meta.parent_id`, and field values.

## Key Takeaways
1. v2 is read-only (DRF); v3 is read/write (Django Ninja) with OpenAPI 3.1 schemas.
2. The v3 API mirrors admin operations — same permissions, validation, revisions, and audit logs.
3. StreamField values are JSON arrays; PATCH replaces the entire list, not individual blocks.
4. Use `expand_db_html()` to convert rich text internal format for API consumers.
5. Declare `api_fields` on models to control which fields are exposed in the API.

## Connects To
- **Ch 11 (Deployment)**: The API runs on the same WSGI server and deployment infrastructure.
- **Ch 15 (Headless)**: The API is the primary interface for headless Wagtail architectures.
- **Ch 12 (Admin)**: The API provides programmatic access to the same models managed through admin views.
