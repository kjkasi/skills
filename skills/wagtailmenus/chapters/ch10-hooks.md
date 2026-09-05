# Chapter 10: Hooks

## Core Idea
wagtailmenus provides Django signals/hooks that fire at specific points in the menu rendering lifecycle, allowing you to modify menu data before it reaches templates.

## Frameworks Introduced
- **Hook-Based Modification**: Use Django signals to intercept and modify menu items before rendering
  - When to use: Adding custom logic to all menus (analytics, A/B testing, permission filtering)
  - How: Connect to wagtailmenus signals in your app's ready() method
- **Before Render Hook**: Modify menu items after they're fetched but before template rendering
  - When to use: Filtering, reordering, or enriching menu items globally
  - How: Connect to 'wagtailmenus.before_render_menu' signal

## Key Concepts
- **before_render_menu**: Signal fired after menu items are fetched, before template rendering
- **menu_items**: List of menu item objects available in the signal
- **request**: HttpRequest object for accessing user, session, etc.
- **signal**: Django signal mechanism for decoupled communication

## Mental Models
- Hooks are "interceptors" — they let you modify menu data without changing core code
- Use hooks for global behavior; use template overrides for presentation changes
- The before_render_menu signal gives you the final chance to modify items before they reach templates
- Hooks are per-request; consider caching for performance

## Anti-patterns
- **Overusing hooks**: Prefer model overrides for data changes; hooks are for runtime modifications
- **Ignoring performance**: Hooks run on every request; avoid expensive operations
- **Tight coupling**: Hooks should be self-contained; don't depend on external state that might not exist

## Code Examples
```python
# apps/core/signals.py
from django.dispatch import receiver
from wagtailmenus.signals import before_render_menu

@receiver(before_render_menu)
def filter_menu_by_user_role(sender, menu_items, request, **kwargs):
    """Filter menu items based on user role"""
    if request.user.is_authenticated:
        if request.user.is_superuser:
            return menu_items  # Superusers see everything
        else:
            # Filter out admin-only items
            return [item for item in menu_items if not item.link_url.startswith('/admin/')]
    else:
        # Anonymous users see public items only
        return [item for item in menu_items if item.show_in_menus]
```
- **What it demonstrates**: Filtering menu items based on user role

```python
# apps/core/apps.py
from django.apps import AppConfig

class CoreConfig(AppConfig):
    name = 'core'
    
    def ready(self):
        import core.signals  # noqa: F401
```
- **What it demonstrations**: Registering signal handlers

```python
# apps/core/signals.py - adding analytics attributes
@receiver(before_render_menu)
def add_analytics_attributes(sender, menu_items, request, **kwargs):
    """Add data-analytics attributes to menu items"""
    for item in menu_items:
        item.analytics_data = f"menu_{item.menu.title}_{item.pk}"
    return menu_items
```
- **What it demonstrations**: Enriching menu items with custom attributes

## Reference Tables

| Signal | Sender | Arguments | When Fired |
|--------|--------|-----------|------------|
| before_render_menu | Menu class | menu_items, request | After items fetched, before template render |

## Worked Example
Adding A/B test variants to menu items:

1. Create signal handler:
```python
@receiver(before_render_menu)
def add_ab_test_variants(sender, menu_items, request, **kwargs):
    """Add A/B test variant to menu items"""
    if hasattr(request, 'ab_test_variant'):
        for item in menu_items:
            item.ab_variant = request.ab_test_variant
    return menu_items
```

2. In templates, use the variant attribute:
```html
<a href="{{ item.href }}" data-variant="{{ item.ab_variant }}">{{ item.text }}</a>
```

Result: Menu items now carry A/B test variant information for analytics tracking.

## Key Takeaways
1. Hooks provide runtime modification of menu data without changing core code
2. before_render_menu signal fires after items are fetched, before template rendering
3. Return modified menu_items from signal handlers
4. Use for global behavior; prefer model overrides for data changes
5. Consider performance; hooks run on every request

## Connects To
- **Ch 5**: Rendering Menus — where hooks fit in the rendering pipeline
- **Ch 9**: Custom Menu Classes — model overrides for data changes
