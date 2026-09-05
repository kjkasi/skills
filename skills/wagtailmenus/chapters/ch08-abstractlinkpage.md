# Chapter 8: AbstractLinkPage Model

## Core Idea
AbstractLinkPage provides a way to add menu links without creating full page content; link pages can point to other pages or custom URLs and automatically hide when targets are unpublished.

## Frameworks Introduced
- **Link Page Pattern**: Lightweight pages that exist only to provide menu links; no content, no children, excluded from sitemaps
  - When to use: Adding links to menus that don't need full page content (external links, anchors, redirects)
  - How: Subclass AbstractLinkPage, create instances in Wagtail admin, add to menus
- **Automatic Link Hiding**: Link pages automatically hide when their target page is unpublished or expires
  - When to use: Ensuring menu links don't point to unavailable content
  - How: Built-in behavior; no configuration needed

## Key Concepts
- **AbstractLinkPage**: Abstract model providing link-only page behavior
- **link_page**: ForeignKey to target page (optional)
- **link_url**: Custom URL field (optional, supports anchors like #signup)
- **url_append**: Additional path/params added to page URL
- **link_text**: Custom display text (overrides page title)
- **allow_subnav**: Whether to show children in sub-menus (default False for link pages)

## Mental Models
- Link pages are "menu connectors" — they exist to link, not to contain content
- Think of them as bookmarks in the page tree; they appear in navigation but don't render as full pages
- Automatic hiding prevents broken links in menus when content is removed
- Link pages are excluded from sitemaps and search results by default

## Anti-patterns
- **Using link pages for real content**: Create actual pages for content; link pages are for navigation only
- **Ignoring link page visibility**: Remember that link pages appear in menus but not in page listings
- **Overcomplicating link pages**: Keep them simple; they're connectors, not content containers

## Code Examples
```python
# models.py - implementing AbstractLinkPage
from wagtailmenus.models import AbstractLinkPage

class LinkPage(AbstractLinkPage):
    """A link-only page type for menu navigation"""
    pass
```
- **What it demonstrates**: Creating a link page type

```console
# Create and apply migrations
python manage.py makemigrations appname
python manage.py migrate appname
```
- **What it demonstrates**: Database setup for link pages

```html
{# Link pages work seamlessly in menu templates #}
{% for item in menu_items %}
    <li>
        <a href="{{ item.href }}">{{ item.text }}</a>
        {% if item.has_children_in_menu %}
            {% sub_menu item %}
        {% endif %}
    </li>
{% endfor %}
```
- **What it demonstrates**: Link pages render like regular menu items

## Reference Tables

| Field | Type | Default | Purpose |
|-------|------|---------|---------|
| link_page | ForeignKey(Page) | null | Target page for link |
| link_url | URLField | '' | Custom URL (supports anchors) |
| url_append | CharField | '' | Additional path/params |
| link_text | CharField | '' | Custom display text |
| show_in_menus | BooleanField | True | Show in menus |
| show_in_sitemap | BooleanField | False | Include in sitemaps |
| show_in_search_results | BooleanField | False | Include in search |

## Worked Example
Adding a "Request Callback" link to the main navigation:

1. Create LinkPage model (if not already done):
```python
class LinkPage(AbstractLinkPage):
    pass
```

2. Run migrations

3. In Wagtail admin, create a LinkPage:
   - Title: "Request Callback"
   - Link URL: #request-callback
   - Link Text: "Request Callback"
   - Parent: Home page (or wherever it should appear in tree)

4. In main menu, add this LinkPage as a menu item

5. Result: "Request Callback" appears in navigation, scrolls to #request-callback section on the current page

## Key Takeaways
1. AbstractLinkPage provides lightweight menu links without full page content
2. Link pages automatically hide when target pages are unpublished or expire
3. They support custom URLs, anchors, and additional path parameters
4. Link pages are excluded from sitemaps and search results by default
5. Use for external links, anchors, redirects, or any menu link that doesn't need page content

## Connects To
- **Ch 1**: Overview — understanding menu link types
- **Ch 3**: Managing Main Menus — adding link pages to main navigation
- **Ch 4**: Managing Flat Menus — using link pages in flat menus
