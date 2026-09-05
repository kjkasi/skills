# Chapter 12: Extending the Wagtail Admin

## Core Idea
Wagtail's admin is a suite of Django apps. Custom views, forms, menu items, and model management can be added using standard Django patterns (views, templates, URL routes) combined with Wagtail's hooks system and ViewSet abstractions.

## Frameworks Introduced
- **ViewSet**: Bundles related admin views, URLs, and menu items into a single registrable unit.  - When to use: When you have multiple related views (list, create, edit, inspect) for a model.  - How: Subclass `wagtail.admin.viewsets.base.ViewSet`, define `get_urlpatterns()`, register via `register_admin_viewset` hook.
- **ModelViewSet**: Pre-built ViewSet providing CRUD views (list, create, edit, delete, inspect, copy) for any Django model.  - When to use: When you need full admin management for a model without writing views.  - How: Subclass `wagtail.admin.viewsets.model.ModelViewSet`, set `model`, `form_fields`, register via hook.
- **ChooserViewSet**: Provides modal chooser UI for selecting model instances in ForeignKey fields.  - When to use: When you need a reusable picker widget for a model.  - How: Subclass `wagtail.admin.viewsets.chooser.ChooserViewSet`, set `model`, register via hook.
- **ViewSetGroup**: Groups multiple ViewSets under a single top-level menu item.  - When to use: When related ViewSets should appear together in the sidebar.  - How: Subclass `wagtail.admin.viewsets.base.ViewSetGroup`, set `items` and `menu_label`.
- **Panels (Edit Handlers)**: Declarative mechanism for model form layout without templates.  - When to use: Editing interfaces for pages, snippets, and settings.  - How: Define `panels` or `edit_handler` on the model or ViewSet.

## Key Concepts
- **`wagtail_hooks.py`**: Module in each app where hooks are registered to extend the admin (URLs, menu items, ViewSets).
- **`register_admin_urls` hook**: Adds URL routes under the `/admin/` namespace.
- **`register_admin_menu_item` hook**: Adds items to the admin sidebar.
- **`register_admin_viewset` hook**: Registers a ViewSet or ViewSetGroup with the admin.
- **`MenuItem` / `SubmenuMenuItem` / `Menu`**: Classes for building sidebar navigation.
- **`WagtailAdminModelForm`**: Base form class that auto-selects Wagtail widgets (date pickers, choosers) for model fields.
- **`register_form_field_override`**: Maps custom model field types to custom form widgets.
- **`AdminURLFinder`**: Utility to programmatically find admin edit URLs for any model instance.

## Mental Models
1. **Hook-based extensibility**: The admin is extended by registering callbacks with named hooks at import time. Wagtail calls these hooks during startup to build URLs, menus, and viewsets.
2. **ViewSet as a view bundle**: Instead of registering individual views and menu items separately, a ViewSet groups them and registers as one unit — reducing boilerplate and ensuring consistency.
3. **Panel → Form → Template pipeline**: Panels define the form layout → `get_form_class()` generates a `ModelForm` → the view renders it. You can intercept at any stage.
4. **Template inheritance for admin views**: Custom views extend `"wagtailadmin/base.html"` and override `content`, `extra_css`, `titletag` blocks.

## Anti-patterns
- **Bypassing hooks for admin URLs**: Adding URLs directly to `urls.py` skips authentication and menu integration. Always use `register_admin_urls`.
- **Ignoring `WagtailAdminModelForm`**: Using plain `ModelForm` loses Wagtail's auto-widget selection for dates, images, documents, and snippets.
- **Hardcoding admin URL patterns**: Use `reverse()` with named URLs rather than hardcoded paths, as Wagtail manages the `/admin/` namespace.
- **Not registering custom widgets in `AppConfig.ready`**: Widget overrides must be registered early via `register_form_field_override` in `ready()`.

## Code Examples
```python
# Custom admin view with menu item
# wagtail_hooks.py
from django.urls import path, reverse
from wagtail.admin.menu import MenuItem
from wagtail import hooks

from .views import index

@hooks.register("register_admin_urls")
def register_calendar_url():
    return [path("calendar/", index, name="calendar")]

@hooks.register("register_admin_menu_item")
def register_calendar_menu_item():
    return MenuItem("Calendar", reverse("calendar"), icon_name="date")
```
- **What it demonstrates**: Adding a custom admin view with a sidebar menu item using hooks.

```python
# ModelViewSet for full CRUD
from wagtail.admin.viewsets.model import ModelViewSet
from .models import Person

class PersonViewSet(ModelViewSet):
    model = Person
    form_fields = ["first_name", "last_name"]
    icon = "user"
    add_to_admin_menu = True
    inspect_view_enabled = True

person_viewset = PersonViewSet("person")

# wagtail_hooks.py
from wagtail import hooks
from .views import person_viewset

@hooks.register("register_admin_viewset")
def register_viewset():
    return person_viewset
```
- **What it demonstrates**: Full admin CRUD for a model with zero custom views.

```python
# WagtailAdminModelForm with custom widget override
from wagtail.admin.forms.models import WagtailAdminModelForm

class FeaturedImageForm(WagtailAdminModelForm):
    class Meta:
        model = FeaturedImage

# In AppConfig.ready():
from wagtail.admin.forms.models import register_form_field_override
register_form_field_override(ForeignKey, to=Video, override={"widget": VideoChooser})
```
- **What it demonstrates**: Auto-selecting Wagtail widgets for model fields.

```python
# AdminURLFinder for programmatic admin URL lookup
from wagtail.admin.admin_url_finder import AdminURLFinder

finder = AdminURLFinder()
edit_url = finder.get_edit_url(model_instance)
```
- **What it demonstrates**: Finding the admin edit URL for any model instance at runtime.

## Reference Tables

| Hook Name | Purpose |
|---|---|
| `register_admin_urls` | Add URL routes under `/admin/` |
| `register_admin_menu_item` | Add sidebar menu items |
| `register_admin_viewset` | Register ViewSet/ViewSetGroup |
| `register_rich_text_features` | Extend rich text editor |

| ViewSet Attribute | Description |
|---|---|
| `model` | Django model class |
| `form_fields` | Fields to show on create/edit forms |
| `icon` | Wagtail icon name |
| `add_to_admin_menu` | Show in main sidebar |
| `inspect_view_enabled` | Enable read-only detail view |
| `copy_view_enabled` | Enable copy functionality |
| `sort_order_field` | Enable drag-and-drop reordering |
| `list_filter` | Enable listing filters (django-filter) |

## Worked Example
Build a calendar admin view:
1. Create `wagtailcalendar` app, add to `INSTALLED_APPS`.
2. Write a plain Django view returning `calendar.HTMLCalendar().formatyear(year)`.
3. In `wagtail_hooks.py`, register the URL via `register_admin_urls` and a menu item via `register_admin_menu_item`.
4. Create a template extending `"wagtailadmin/base.html"`, overriding `content` and `extra_css`.
5. Use `{% include "wagtailadmin/shared/header.html" %}` for the standard header bar.
6. Optionally, group multiple views using `ViewSet` or `ViewSetGroup`.

## Key Takeaways
1. The admin is a Django app — use standard Django views, templates, and URL routes.
2. Hooks (`wagtail_hooks.py`) are the entry point for all admin extensions.
3. Use `ModelViewSet` for model CRUD; use `ViewSet`/`ViewSetGroup` for custom view bundles.
4. `WagtailAdminModelForm` auto-selects appropriate Wagtail widgets — always prefer it over plain `ModelForm`.
5. `AdminURLFinder` provides programmatic access to admin edit URLs for any model.

## Connects To
- **Ch 11 (Deployment)**: Admin extensions are deployed alongside the main Wagtail app.
- **Ch 13 (Frontend)**: Client-side admin customization (JavaScript, panels) complements server-side view extensions.
- **Ch 14 (API)**: The API can expose the same models managed through custom admin views.
