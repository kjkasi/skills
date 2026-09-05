# Cheatsheet

## Quick Decision Rules

| Situation | Action | Why |
|-----------|--------|-----|
| Need primary navigation | Use main_menu tag + CMS menu | Editors control top-level items |
| Need footer/sidebar menu | Use flat_menu tag + CMS menu | Flexible, supports custom URLs |
| Need section sub-navigation | Use section_menu tag | Auto-generated from page tree |
| Page should be clickable parent | Subclass MenuPage, enable repeat_in_menu | Solves "clickable parent" problem |
| Need menu link without content | Subclass AbstractLinkPage | Lightweight, auto-hides when target unpublished |
| Add image/description to items | Subclass AbstractMainMenuItem | Custom fields on menu items |
| Multi-site menu sharing | flat_menu with fall_back_to_default_site_menus=True | Reduces duplication |
| Custom menu behavior | Override menu classes via settings | Change query/rendering logic |
| Global menu filtering | Connect to before_render_menu signal | Runtime modifications |
| Different templates per site | Enable SITE_SPECIFIC_TEMPLATE_DIRS | Per-site branding |

## Template Tag Quick Reference

```html
{% load menu_tags %}

{# Main navigation #}
{% main_menu max_levels=2 template="nav.html" %}

{# Flat menu by handle #}
{% flat_menu 'footer' show_menu_heading=False %}

{# Current section #}
{% section_menu max_levels=2 %}

{# Children of page #}
{% children_menu page=some_page %}

{# Sub-menu (recursive) #}
{% sub_menu item template="sub.html" %}
```

## Settings Quick Reference

```python
# Admin UI
WAGTAILMENUS_ADD_EDITOR_OVERRIDE_STYLES = True
WAGTAILMENUS_FLATMENU_MENU_ICON = 'list-ol'
WAGTAILMENUS_FLAT_MENUS_EDITABLE_IN_WAGTAILADMIN = True

# Templates
WAGTAILMENUS_DEFAULT_MAIN_MENU_TEMPLATE = 'menus/main_menu.html'
WAGTAILMENUS_DEFAULT_FLAT_MENU_TEMPLATE = 'menus/flat_menu.html'
WAGTAILMENUS_SITE_SPECIFIC_TEMPLATE_DIRS = False

# Model Overrides
WAGTAILMENUS_MAIN_MENU_MODEL = 'wagtailmenus.MainMenu'
WAGTAILMENUS_FLAT_MENU_MODEL = 'wagtailmenus.FlatMenu'
WAGTAILMENUS_MAIN_MENU_ITEMS_RELATED_NAME = 'menu_items'

# Rendering
WAGTAILMENUS_ACTIVE_CLASS = 'active'
WAGTAILMENUS_ACTIVE_ANCESTOR_CLASS = 'ancestor'
WAGTAILMENUS_DEFAULT_ADD_SUB_MENUS_INLINE = False
```

## Common Patterns

### Minimal Setup
```python
INSTALLED_APPS = [..., 'wagtailmenus']
TEMPLATES = [{'OPTIONS': {'context_processors': [..., 'wagtailmenus.context_processors.wagtailmenus']}}]
python manage.py migrate wagtailmenus
```

### Custom Menu Item with Image
```python
class CustomItem(AbstractMainMenuItem):
    menu = ParentalKey('wagtailmenus.MainMenu', on_delete=models.CASCADE, related_name="custom_items")
    image = models.ForeignKey(get_image_model_string(), blank=True, null=True, on_delete=models.SET_NULL)

# settings.py
WAGTAILMENUS_MAIN_MENU_ITEMS_RELATED_NAME = "custom_items"
```

### Link Page
```python
class LinkPage(AbstractLinkPage):
    pass
# Run migrations, create in admin, add to menus
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Menu tags not working | Check context processor is in TEMPLATES |
| Sub-menus empty | Check allow_subnav=True on parent items |
| Pages missing from menus | Check show_in_menus=True on pages |
| Menu not updating | Check page tree order, not CMS order |
| Flat menu empty | Check handle matches template tag argument |
| Custom model not appearing | Check related_name matches setting |
| Template not found | Check template discovery order |

## Performance Tips

| Tip | Impact |
|-----|--------|
| Use show_multiple_levels=False for flat lists | Fewer database queries |
| Cache menu output for anonymous users | Reduces per-request queries |
| Avoid use_absolute_page_urls=True | Slower URL generation |
| Use max_levels to limit depth | Controls query scope |
| Prefer flat menus for static content | Less page tree dependency |
