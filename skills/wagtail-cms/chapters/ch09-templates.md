# Chapter 9: Writing Templates

## Core Idea
Wagtail uses Django's template language with additional template tags and filters for images, rich text, internal links, static files, and caching. Templates follow a naming convention derived from model class names, and provide access to page content via the `page` variable.

## Frameworks Introduced
- **Template Naming Convention**: Wagtail converts CamelCase model names to snake_case to find templates (e.g., `BlogPage` → `blog_page.html`).
  - When to use: Always — this is the default template resolution strategy.
  - How: Place templates in `app/templates/app/model_name.html`; override with `template` attribute on the model if needed.
- **Image Rendering Pipeline**: Use `{% image %}`, `{% picture %}`, or `{% srcset_image %}` tags for on-the-fly image manipulation.
  - When to use: Any time you render an image from the Wagtail image library.
  - How: Load `wagtailimages_tags`, use the tag with a resize rule (e.g., `fill-80x80`, `width-400`).
- **Preview-Aware Caching**: Use `{% wagtailcache %}` and `{% wagtailpagecache %}` instead of Django's `{% cache %}` to avoid serving cached preview content to end users.
  - When to use: Any template fragment caching in Wagtail.
  - How: Load `wagtail_cache`, use `{% wagtailcache %}` or `{% wagtailpagecache %}` with timeout and cache key.

## Key Concepts
- **`page` variable**: The current page object in templates; all field values accessed via `{{ page.field_name }}`.
- **`{{ page.body|richtext }}`**: Renders `RichTextField` content as safe HTML, expanding internal references.
- **`{% image page.photo fill-80x80 %}`**: On-the-fly image resizing with various crop/resize rules.
- **`{% picture page.photo format-{avif,webp,jpeg} %}`**: Generates `<picture>` element with multiple format sources.
- **`{% srcset_image page.photo width-{400,800} %}`**: Generates `srcset` for responsive images.
- **`{% pageurl page %}`**: Returns relative URL for same-site pages, absolute for cross-site.
- **`{% fullpageurl page %}`**: Always returns absolute URL.
- **`{% slugurl 'news' %}`**: Returns URL for a page by slug.
- **`{% static "app/css/style.css" %}`**: Standard Django static file tag.
- **`{% wagtailuserbar %}`**: Contextual admin flyout for logged-in editors.
- **`{% wagtail_site as current_site %}`**: Returns the current `Site` object.
- **`request.is_preview`**: Boolean indicating preview mode; use to exclude analytics, etc.
- **`request.in_preview_panel`**: Boolean indicating rendering inside the live preview iframe.

## Mental Models
- Wagtail templates are standard Django templates — all Django template language features work.
- Images from the library must use `{% image %}` or `{% picture %}` tags for on-the-fly processing; static images use standard `<img>` tags.
- Template fragment caching must use Wagtail-aware tags (`wagtailcache`/`wagtailpagecache`) to prevent preview content leakage.

## Anti-patterns
- **Using Django's `{% cache %}` tag directly**: Can serve preview content to end users — use `{% wagtailcache %}` instead.
- **Hardcoding image sizes in `<img>` tags**: Misses on-the-fly processing; always use `{% image %}` for library images.
- **Forgetting `{% load %}` for Wagtail tags**: Tags like `pageurl`, `image`, `richtext` require explicit loading from their respective tag libraries.
- **Accessing page fields without `page.` prefix**: The default context variable is `page` — omitting it causes silent template errors.

## Code Examples
```html+django
{# basic page template #}
{% extends "base.html" %}
{% block content %}
    <h1>{{ page.title }}</h1>
    <p>By {{ page.author }}</p>
    {{ page.body|richtext }}
{% endblock %}
```
- **What it demonstrates**: Basic page template with rich text rendering.

```html+django
{# image tag — resize on the fly #}
{% load wagtailimages_tags %}
{% image page.photo width-400 %}
{% image page.photo fill-80x80 %}
{% image page.photo url %}
```
- **What it demonstrates**: Various image resize rules and URL generation.

```html+django
{# picture tag — multiple formats #}
{% load wagtailimages_tags %}
{% picture page.photo format-{avif,webp,jpeg} width-400 %}
```
- **What it demonstrates**: Generating a `<picture>` element with format fallbacks.

```html+django
{# responsive images with srcset #}
{% load wagtailimages_tags %}
{% srcset_image page.photo width-{400,800} sizes="(max-width: 600px) 400px, 80vw" %}
```
- **What it demonstrates**: Responsive image with multiple sizes.

```html+django
{# internal links #}
{% load wagtailcore_tags %}
<a href="{% pageurl page.get_parent %}">Back to index</a>
<a href="{% fullpageurl page %}">Share this page</a>
<a href="{% slugurl 'news' %}">News</a>
```
- **What it demonstrates**: Various internal link tags.

```html+django
{# static files #}
{% load static %}
<img src="{% static "myapp/images/logo.png" %}" alt="Logo">
```
- **What it demonstrates**: Referencing static assets.

```html+django
{# wagtail user bar #}
{% load wagtailuserbar %}
<body>
    {% wagtailuserbar 'top-left' %}
    <nav>...</nav>
    <main>{{ page.body|richtext }}</main>
</body>
```
- **What it demonstrating**: Adding the admin user bar with positioning.

```html+django
{# preview-aware caching #}
{% load wagtail_cache %}
{% wagtailcache 500 sidebar %}
    <!-- sidebar content -->
{% endwagtailcache %}

{% wagtailpagecache 500 hero %}
    <!-- hero content -->
{% endwagtailpagecache %}
```
- **What it demonstrates**: Safe template fragment caching that skips preview.

```html+django
{# preview vs live #}
{% if not request.is_preview %}
    <script>/* analytics */</script>
{% endif %}

{% if request.in_preview_panel %}
    <base target="_blank">
{% endif %}
```
- **What it demonstrating**: Conditional output for preview vs live rendering.

```html+django
{# multi-site support #}
{% load wagtailcore_tags %}
{% wagtail_site as current_site %}
<p>You are on: {{ current_site.site_name }}</p>
```
- **What it demonstrating**: Accessing the current site object.

## Reference Tables

| Tag/Filter | Library | Purpose |
|---|---|---|
| `{% image %}` | `wagtailimages_tags` | On-the-fly image resize |
| `{% picture %}` | `wagtailimages_tags` | Multiple format sources |
| `{% srcset_image %}` | `wagtailimages_tags` | Responsive srcset |
| `{{ field\|richtext }}` | `wagtailcore_tags` | Render RichTextField as HTML |
| `{% pageurl page %}` | `wagtailcore_tags` | Relative/absolute page URL |
| `{% fullpageurl page %}` | `wagtailcore_tags` | Always absolute page URL |
| `{% slugurl 'slug' %}` | `wagtailcore_tags` | URL by page slug |
| `{% static %}` | `static` | Static file URL |
| `{% wagtailuserbar %}` | `wagtailuserbar` | Admin flyout menu |
| `{% wagtail_site %}` | `wagtailcore_tags` | Current Site object |
| `{% wagtailcache %}` | `wagtail_cache` | Preview-safe fragment cache |
| `{% wagtailpagecache %}` | `wagtail_cache` | Page-aware fragment cache |

| Image Resize Rule | Example | Effect |
|---|---|---|
| `width-N` | `width-400` | Scale to N pixels wide |
| `height-N` | `height-300` | Scale to N pixels tall |
| `fill-WxH` | `fill-80x80` | Crop and resize to WxH |
| `max-WxH` | `max-600x400` | Fit within WxH bounding box |
| `min-WxH` | `min-200x200` | Scale up to fill WxH minimum |
| `original` | `original` | No processing, full size |

## Worked Example
A complete blog page template:

```html+django
{% extends "base.html" %}
{% load wagtailcore_tags wagtailimages_tags wagtailuserbar wagtail_cache %}

{% block title %}{{ page.title }} | My Blog{% endblock %}

{% block body %}
    {% wagtailuserbar 'top-left' %}

    <article>
        <h1>{{ page.title }}</h1>
        <p>Published {{ page.first_published_at|date:"F j, Y" }}</p>

        {% wagtailpagecache 3600 articlehero %}
            {% image page.hero_image fill-1200x400 %}
        {% endwagtailpagecache %}

        {{ page.body|richtext }}

        {% if page.document %}
            <a href="{{ page.document.url }}">Download {{ page.document.title }}</a>
        {% endif %}
    </article>

    <a href="{% pageurl page.get_parent %}">Back to blog</a>
{% endblock %}
```

## Key Takeaways
1. Templates follow CamelCase → snake_case naming; override with `template` attribute if needed.
2. Always use `{% image %}` for library images — never `<img>` with hardcoded URLs.
3. Use `{{ field|richtext }}` for `RichTextField` — it expands internal references.
4. Replace `{% cache %}` with `{% wagtailcache %}` / `{% wagtailpagecache %}` to avoid preview leakage.
5. `{% pageurl %}` gives relative URLs for same-site pages — better for SEO and multi-site.

## Connects To
- **Ch 6**: Documents linked to pages are rendered via `{{ page.document.url }}` in templates.
- **Ch 7**: Search results use `{% pageurl result %}` for result links.
- **Ch 8**: Snippets are included in templates via custom template tags or ForeignKey bindings.
- **Ch 10**: `{% wagtailuserbar %}` respects the current user's permissions to show relevant admin actions.
