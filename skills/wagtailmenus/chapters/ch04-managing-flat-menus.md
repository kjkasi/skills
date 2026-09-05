# Chapter 4: Managing Flat Menus

## Core Idea
Flat menus provide CMS-managed navigation for any context (footer, sidebar, secondary nav) with support for custom URLs, multi-level output, and cross-site reuse.

## Frameworks Introduced
- **Flat Menu Pattern**: CMS-managed menus not tied to page tree; editors define items directly with page links or custom URLs
  - When to use: Footer navigation, sidebar menus, help menus, any non-primary navigation
  - How: Create via Wagtail admin under Settings > Flat menus, assign unique handle
- **Cross-Site Menu Reuse**: Flat menus can be shared across sites or site-specific using `fall_back_to_default_site_menus`
  - When to use: Multi-site projects where some menus are identical across sites
  - How: Set handle and site in CMS; use `fall_back_to_default_site_menus=True` in template tag

## Key Concepts
- **FlatMenu**: Wagtail model storing a named menu with items; identified by handle and site
- **FlatMenuItem**: Individual items within a flat menu; can link to pages or custom URLs
- **handle**: Unique identifier for a flat menu (e.g., 'footer', 'sidebar', 'help')
- **fall_back_to_default_site_menus**: When True, uses default site's menu if current site has none
- **max_levels**: Controls how many levels deep a flat menu renders (can be set per-menu in CMS)

## Mental Models
- Flat menus are "named containers" — give them a handle, add items, reference by handle in templates
- Think of handles as menu identifiers: 'footer', 'header', 'sidebar', 'help'
- Flat menus can be flat lists or multi-level trees depending on max_levels setting
- Cross-site reuse is opt-in: each site gets its own menu unless you explicitly share

## Anti-patterns
- **Hard-coding footer links**: Use flat menus instead; enables editor changes without code deploys
- **Ignoring handle naming**: Use descriptive handles (footer, sidebar) not generic ones (menu1, menu2)
- **Forgetting fall_back_to_default_site_menus**: In multi-site projects, this prevents empty menus on secondary sites

## Code Examples
```html
{% load menu_tags %}

{# Basic flat menu #}
{% flat_menu 'footer' %}

{# With custom template and options #}
{% flat_menu 'footer' template="menus/custom_footer.html" show_menu_heading=False %}

{# Multi-level flat menu #}
{% flat_menu 'sidebar' max_levels=2 %}

{# Cross-site fallback #}
{% flat_menu 'footer' fall_back_to_default_site_menus=True %}
```
- **What it demonstrates**: Template tag usage for rendering flat menus

```python
# settings/base.py - constrain flat menu handles
WAGTAILMENUS_FLAT_MENUS_HANDLE_CHOICES = (
    ('header', 'Header'),
    ('footer', 'Footer'),
    ('sidebar', 'Sidebar'),
    ('help', 'Help'),
)
```
- **What it demonstrates**: Restricting flat menu handles to predefined choices

## Reference Tables

| Argument | Type | Default | Purpose |
|----------|------|---------|---------|
| handle | str | required | Unique menu identifier |
| show_menu_heading | bool | True | Display heading above menu |
| show_multiple_levels | bool | True | Allow multi-level output |
| max_levels | int | None | Override menu's max_levels field |
| fall_back_to_default_site_menus | bool | False | Use default site menu if none for current site |
| template | str | '' | Custom template path |
| sub_menu_template | str | '' | Custom sub-menu template |

## Worked Example
Setting up a footer menu for a multi-site project:

1. In CMS, go to Settings > Flat menus
2. Create flat menu:
   - Title: "Footer Navigation"
   - Handle: "footer"
   - Site: default site
   - Menu items: Quick Links (link_url='#'), Contact (link_page=Contact), Privacy Policy (link_page=Privacy)

3. In template:
```html
{% load menu_tags %}
<footer>
    {% flat_menu 'footer' 
       template="menus/footer.html" 
       show_menu_heading=False 
       fall_back_to_default_site_menus=True %}
</footer>
```

Result: Footer menu appears on all sites; secondary sites use default site's menu if they don't have their own.

## Key Takeaways
1. Flat menus are CMS-managed menus for any non-primary navigation context
2. Each flat menu is identified by a unique handle within a site
3. Flat menus support both flat lists and multi-level trees (configurable via max_levels)
4. Use `fall_back_to_default_site_menus=True` for multi-site projects
5. Editors can create, edit, and copy flat menus without touching code

## Connects To
- **Ch 1**: Overview — understanding menu types
- **Ch 5**: Rendering Menus — template tags for displaying flat menus
- **Ch 11**: Settings Reference — flat menu configuration options
