# Chapter 6: Documents

## Core Idea
Wagtail provides a document management system for uploading, storing, and serving files like PDFs and spreadsheets. Documents support custom models, configurable storage backends, collection-based organization, and privacy controls.

## Frameworks Introduced
- **Custom Document Model**: Extend `AbstractDocument` to add custom fields, then point `WAGTAILDOCS_DOCUMENT_MODEL` to the new model.
  - When to use: You need extra metadata on documents (e.g., source, author, license).
  - How: Inherit from `AbstractDocument`, add fields, append to `admin_form_fields`, set `WAGTAILDOCS_DOCUMENT_MODEL` in settings.
- **Document Serving Methods**: Control how documents are delivered to users via `WAGTAILDOCS_SERVE_METHOD`.
  - When to use: When you need to balance security (permission checks) against performance (direct serving).
  - How: Choose `serve_view` (Django view, enforces privacy), `redirect` (cloud URL), or `direct` (filesystem path).

## Key Concepts
- **`get_document_model()`**: Returns the active document model class — always use this instead of importing the model directly, so custom models are respected.
- **Collections**: Group documents into tree-structured collections with independent privacy settings; documents default to the root collection.
- **`DocumentChooserBlock`**: A StreamField block that lets editors pick a document in the stream editor.
- **`WAGTAILDOCS_SERVE_METHOD`**: Settings key controlling document serving strategy (`serve_view`, `redirect`, `direct`).
- **`WAGTAILDOCS_EXTENSIONS`**: Allowlist of permitted file extensions for uploads.
- **`WAGTAILDOCS_MAX_UPLOAD_SIZE`**: Maximum upload size in bytes to prevent DoS.

## Mental Models
- Documents are separate from pages — they live in a flat library, not in the page tree, and are referenced by pages via ForeignKey or StreamField.
- Privacy is collection-based: place documents in private collections to restrict access; the serve method determines whether server-level controls can enforce this.
- Always use `get_document_model()` rather than importing `Document` directly — this ensures your custom model is used everywhere.

## Anti-patterns
- **Importing `Document` directly instead of using `get_document_model()`**: Breaks custom document models; always use the helper function.
- **Using `direct` or `redirect` serve methods without server-level controls**: Bypasses permission checks and allows script execution in uploaded files.
- **Ignoring file type validation**: Uploading unrestricted file types opens XSS and malware vectors — always set `WAGTAILDOCS_EXTENSIONS` and `WAGTAILDOCS_CONTENT_TYPES`.

## Code Examples
```python
# settings.py — register the app
INSTALLED_APPS = [
    # ...
    "wagtail.documents",
]
```
- **What it demonstrates**: Enabling the wagtail.documents app.

```python
# urls.py — wire up document URLs
from wagtail.documents import urls as wagtaildocs_urls

urlpatterns = [
    path("documents/", include(wagtaildocs_urls)),
]
```
- **What it demonstrates**: Making documents accessible via URL.

```python
# models.py — link a document to a page
from wagtail.admin.panels import FieldPanel
from wagtail.documents import get_document_model

class YourPage(Page):
    document = models.ForeignKey(
        get_document_model(),
        null=True, blank=True,
        on_delete=models.SET_NULL,
    )
    content_panels = Page.content_panels + [
        FieldPanel("document"),
    ]
```
- **What it demonstrates**: ForeignKey to the active document model.

```python
# models.py — custom document model
from wagtail.documents.models import Document, AbstractDocument

class CustomDocument(AbstractDocument):
    source = models.CharField(max_length=255, blank=True, null=True)
    admin_form_fields = Document.admin_form_fields + ("source",)

# settings.py
WAGTAILDOCS_DOCUMENT_MODEL = "app_label.CustomDocument"
```
- **What it demonstrates**: Extending the document model with custom fields.

```python
# models.py — document in StreamField
from wagtail.fields import StreamField
from wagtail.documents.blocks import DocumentChooserBlock

class BlogPage(Page):
    documents = StreamField(
        [("document", DocumentChooserBlock())],
        null=True, blank=True, use_json_field=True,
    )
```
- **What it demonstrates**: Embedding document references in a StreamField.

```html+django
{# template — render document link #}
{% if page.document %}
    <a href="{{ page.document.url }}" target="_blank">{{ page.document.title }}</a>
{% endif %}
```
- **What it demonstrates**: Accessing a linked document in a template.

```python
# serving and security settings
WAGTAILDOCS_SERVE_METHOD = "redirect"
WAGTAILDOCS_EXTENSIONS = ["pdf", "docx"]
WAGTAILDOCS_MAX_UPLOAD_SIZE = 10 * 1024 * 1024  # 10MB
WAGTAILDOCS_CONTENT_TYPES = {
    "pdf": "application/pdf",
    "txt": "text/plain",
}
```
- **What it demonstrates**: Configuring serving, allowed types, and size limits.

## Reference Tables

| Serve Method | Permission Check | Performance | Security |
|---|---|---|---|
| `serve_view` | Enforced via Django view | Lower (serves through app server) | Highest (CSP header set automatically) |
| `redirect` | Bypassed | Higher (served from cloud) | Medium (different domain reduces XSS impact) |
| `direct` | Bypassed | Highest (served from web server) | Lowest (must harden server manually) |

| Setting | Purpose |
|---|---|
| `WAGTAILDOCS_DOCUMENT_MODEL` | Point to custom document model |
| `WAGTAILDOCS_SERVE_METHOD` | Control document serving strategy |
| `WAGTAILDOCS_EXTENSIONS` | Allowlist of file extensions |
| `WAGTAILDOCS_CONTENT_TYPES` | Map extensions to MIME types |
| `WAGTAILDOCS_MAX_UPLOAD_SIZE` | Maximum upload size in bytes |
| `WAGTAILDOCS_INLINE_CONTENT_TYPES` | MIME types displayed inline in rich text |

## Worked Example
Building a resources page with downloadable PDFs:

```python
# models.py
from wagtail.admin.panels import FieldPanel
from wagtail.documents.blocks import DocumentChooserBlock
from wagtail.fields import StreamField

class ResourcePage(Page):
    documents = StreamField(
        [("document", DocumentChooserBlock())],
        null=True, blank=True, use_json_field=True,
    )
    content_panels = Page.content_panels + [FieldPanel("documents")]
```

```html+django
{# templates/resource_page.html #}
{% extends "base.html" %}
{% block content %}
    <h1>{{ page.title }}</h1>
    {% for block in page.documents %}
        <a href="{{ block.value.url }}">{{ block.value.title }}</a>
    {% endfor %}
{% endblock %}
```

## Key Takeaways
1. Always use `get_document_model()` — never import `Document` directly.
2. Choose your serve method based on your security model; `serve_view` is safest for local storage.
3. Validate uploads with `WAGTAILDOCS_EXTENSIONS` and `WAGTAILDOCS_MAX_UPLOAD_SIZE`.
4. Organize documents into collections and use private collections for access control.

## Connects To
- **Ch 10**: Document and image permissions are managed through collections and the Groups admin.
- **Ch 8**: Snippets can reference documents via ForeignKey or DocumentChooserBlock.
- **Ch 9**: Templates use `{{ document.url }}` and `{% image %}` for rendering document-linked content.
