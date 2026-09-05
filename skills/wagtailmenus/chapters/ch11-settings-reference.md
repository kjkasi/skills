# Chapter 11: Settings Reference

## Core Idea
wagtailmenus provides extensive configuration through Django settings, covering admin UI, templates, model overrides, and rendering behavior.

## Frameworks Introduced
- **Settings Hierarchy**: wagtailmenus settings follow Django conventions; override in settings files
  - When to use: Customizing any aspect of wagtailmenus behavior
  - How: Add WAGTAILMENUS_* settings to your Django settings files
- **Admin UI Customization**: Settings control icons, visibility, and admin class overrides
  - When to use: Customizing Wagtail admin interface for menu management
  - How: Use FLATMENU_MENU_ICON, *_EDITABLE_IN_WAGTAILADMIN, *_ADMIN_CLASS settings

## Key Concepts
- **Admin/UI Settings**: Control admin interface appearance and functionality
- **Template Settings**: Override default templates used by menu tags
- **Model Override Settings**: Swap out default models with custom implementations
- **Rendering Settings**: Control active classes, URL generation, and menu behavior
- **Site-Specific Settings**: Enable per-site template directories and menu behavior

## Mental Models
- Settings are organized by category: admin, templates, models, rendering
- Most settings have sensible defaults; only override what you need
- Settings are read once at startup; changes require server restart
- Model override settings use import paths, not direct class references

## Anti-patterns
- **Overriding without understanding**: Read the documentation before changing defaults
- **Hardcoding settings**: Use environment-specific settings files for deployment variations
- **Ignoring validation**: Invalid settings raise ImproperlyConfigured; test configuration changes

## Code Examples
```python
# settings/base.py - admin UI customization
WAGTAILMENUS_ADD_EDITOR_OVERRIDE_STYLES = True  # Default
WAGTAILMENUS_FLATMENU_MENU_ICON = 'list-ol'  # Default
WAGTAILMENUS_FLAT_MENUS_EDITABLE_IN_WAGTAILADMIN = True  # Default
WAGTAILMENUS_MAIN_MENUS_EDITABLE_IN_WAGTAILADMIN = True  # Default
```
- **What it demonstrates**: Admin UI settings with defaults

```python
# settings/base.py - template overrides
WAGTAILMENUS_DEFAULT_MAIN_MENU_TEMPLATE = 'menus/custom_main.html'
WAGTAILMENUS_DEFAULT_FLAT_MENU_TEMPLATE = 'menus/custom_flat.html'
WAGTAILMENUS_DEFAULT_SUB_MENU_TEMPLATE = 'menus/custom_sub.html'
WAGTAILMENUS_SITE_SPECIFIC_TEMPLATE_DIRS = True
```
- **What it demonstrations**: Template configuration

```python
# settings/base.py - model overrides
WAGTAILMENUS_MAIN_MENU_MODEL = 'myapp.CustomMainMenu'
WAGTAILMENUS_MAIN_MENU_ITEMS_RELATED_NAME = 'custom_menu_items'
WAGTAILMENUS_FLAT_MENU_MODEL = 'myapp.CustomFlatMenu'
WAGTAILMENUS_FLAT_MENU_ITEMS_RELATED_NAME = 'custom_menu_items'
```
- **What demonstrations**: Model override configuration

```python
# settings/base.py - rendering behavior
WAGTAILMENUS_ACTIVE_CLASS = 'active'
WAGTAILMENUS_ACTIVE_ANCESTOR_CLASS = 'ancestor'
WAGTAILMENUS_DEFAULT_ADD_SUB_MENUS_INLINE = False
WAGTAILMENUS_DEFAULT_SECTION_MENU_MAX_LEVELS = 2
```
- **What demonstrations**: Rendering behavior configuration

## Reference Tables

| Category | Setting | Default | Purpose |
|----------|---------|---------|---------|
| Admin | ADD_EDITOR_OVERRIDE_STYLES | True | Add custom admin styles |
| Admin | FLATMENU_MENU_ICON | 'list-ol' | Flat menu admin icon |
| Admin | FLAT_MENUS_EDITABLE_IN_WAGTAILADMIN | True | Enable flat menu editing |
| Admin | MAIN_MENUS_EDITABLE_IN_WAGTAILADMIN | True | Enable main menu editing |
| Template | DEFAULT_MAIN_MENU_TEMPLATE | menus/main_menu.html | Default main menu template |
| Template | DEFAULT_FLAT_MENU_TEMPLATE | menus/flat_menu.html | Default flat menu template |
| Template | DEFAULT_SUB_MENU_TEMPLATE | menus/sub_menu.html | Default sub-menu template |
| Template | SITE_SPECIFIC_TEMPLATE_DIRS | False | Enable per-site templates |
| Model | MAIN_MENU_MODEL | wagtailmenus.MainMenu | Custom main menu model |
| Model | FLAT_MENU_MODEL | wagtailmenus.FlatMenu | Custom flat menu model |
| Rendering | ACTIVE_CLASS | 'active' | Current page CSS class |
| Rendering | ACTIVE_ANCESTOR_CLASS | 'ancestor' | Ancestor page CSS class |
| Rendering | DEFAULT_ADD_SUB_MENUS_INLINE | False | Inline sub-menus by default |

## Worked Example
Configuring wagtailmenus for a multi-site project:

```python
# settings/base.py
WAGTAILMENUS_SITE_SPECIFIC_TEMPLATE_DIRS = True

WAGTAILMENUS_FLAT_MENUS_HANDLE_CHOICES = (
    ('header', 'Header'),
    ('footer', 'Footer'),
    ('sidebar', 'Sidebar'),
)

WAGTAILMENUS_FLAT_MENUS_ADMIN_CLASS = 'myapp.admin.CustomFlatMenuAdmin'
WAGTAILMENUS_MAIN_MENUS_ADMIN_CLASS = 'myapp.admin.CustomMainMenuAdmin'
```

Result: Each site can have unique templates, flat menu handles are constrained, and admin classes are customized.

## Key Takeaways
1. Settings follow Django conventions; add WAGTAILMENUS_* to your settings files
2. Most settings have sensible defaults; only override what you need
3. Model override settings use import paths for lazy loading
4. Settings are read once at startup; changes require server restart
5. Invalid settings raise ImproperlyConfigured; test configuration changes

## Connects To
- **Ch 2**: Installation — initial configuration
- **Ch 6**: Custom Templates — template-related settings
- **Ch 9**: Custom Menu Classes — model override settings
