# Chapter 6: Custom Templates

## Core Idea
wagtailmenus supports multiple approaches to override default templates: project-level auto-discovery, per-tag template arguments, and settings-based defaults.

## Frameworks Introduced
- **Template Auto-Discovery**: wagtailmenus searches specific paths for templates before falling back to defaults
  - When to use: Project-wide template overrides without specifying paths everywhere
  - How: Place templates in app template directories at expected paths
- **Template Argument Override**: Specify custom templates directly on template tags
  - When to use: One-off template overrides for specific menu instances
  - How: Add template="path/to/template.html" to menu tag
- **Settings-Based Defaults**: Change default templates globally via Django settings
  - When to use: Project-wide template changes that apply to all menu tags
  - How: Override DEFAULT_*_TEMPLATE settings in Django settings

## Key Concepts
- **Template Discovery Order**: wagtailmenus checks multiple locations for templates
- **Site-Specific Templates**: Multi-site projects can use different templates per site
- **sub_menu_template**: Template used for rendering sub-levels; inherited by recursive calls
- **sub_menu_templates**: Multiple templates for different levels (level_1.html, level_2.html, etc.)

## Mental Models
- Template discovery is hierarchical: tag argument → site-specific → project-level → app defaults
- Site-specific templates enable multi-site branding without code duplication
- Sub-menu templates are inherited: set once on the main tag, applies to all recursive calls
- Use settings for project-wide defaults, tag arguments for instance-specific overrides

## Anti-patterns
- **Hard-coding menu HTML in views**: Use templates instead; enables editor preview and caching
- **Ignoring template discovery order**: Unexpected template resolution can cause confusion
- **Overcomplicating template hierarchy**: Start simple, add complexity only when needed

## Code Examples
```html
{# Specify template on tag #}
{% main_menu template="menus/my_main_menu.html" sub_menu_template="menus/my_sub_menu.html" %}

{# Multiple sub-menu templates for different levels #}
{% main_menu max_levels=3 
   template="menus/level_1.html" 
   sub_menu_templates="menus/level_2.html, menus/level_3.html" %}
```
- **What it demonstrates**: Per-tag template overrides

```python
# settings/base.py - change default templates
WAGTAILMENUS_DEFAULT_MAIN_MENU_TEMPLATE = 'menus/custom_main.html'
WAGTAILMENUS_DEFAULT_FLAT_MENU_TEMPLATE = 'menus/custom_flat.html'
WAGTAILMENUS_DEFAULT_SUB_MENU_TEMPLATE = 'menus/custom_sub.html'
```
- **What it demonstrates**: Global default template overrides

```python
# settings/base.py - enable site-specific templates
WAGTAILMENUS_SITE_SPECIFIC_TEMPLATE_DIRS = True
```
- **What it demonstrates**: Enabling per-site template directories

## Reference Tables

| Setting | Default | Purpose |
|---------|---------|---------|
| DEFAULT_MAIN_MENU_TEMPLATE | menus/main_menu.html | Default template for main_menu tag |
| DEFAULT_FLAT_MENU_TEMPLATE | menus/flat_menu.html | Default template for flat_menu tag |
| DEFAULT_SECTION_MENU_TEMPLATE | menus/section_menu.html | Default template for section_menu tag |
| DEFAULT_CHILDREN_MENU_TEMPLATE | menus/children_menu.html | Default template for children_menu tag |
| DEFAULT_SUB_MENU_TEMPLATE | menus/sub_menu.html | Default template for sub_menu tag |
| SITE_SPECIFIC_TEMPLATE_DIRS | False | Enable per-site template directories |

## Worked Example
Setting up site-specific templates for a multi-site project:

1. Enable site-specific templates:
```python
WAGTAILMENUS_SITE_SPECIFIC_TEMPLATE_DIRS = True
```

2. Create template directories:
```
templates/
├── menus/
│   ├── main_menu.html          # Default for all sites
│   └── flat_menu.html
└── sites/
    ├── site1/
    │   └── menus/
    │       └── main_menu.html  # Site 1 specific
    └── site2/
        └── menus/
            └── main_menu.html  # Site 2 specific
```

3. wagtailmenus automatically picks the site-specific template when available, falls back to default otherwise.

Result: Each site can have unique menu styling while sharing the same menu data and logic.

## Key Takeaways
1. Template discovery checks: tag argument → site-specific → project-level → app defaults
2. Use template arguments for one-off overrides; settings for project-wide changes
3. Site-specific templates enable multi-site branding without code duplication
4. Sub-menu templates are inherited by recursive calls; set once on the main tag
5. Place custom templates in app template directories for auto-discovery

## Connects To
- **Ch 5**: Rendering Menus — template tag usage
- **Ch 11**: Settings Reference — all template-related settings
