# Chapter 5: Rendering Menus with Template Tags

## Core Idea
wagtailmenus provides template tags that handle all menu rendering logic: fetching data, applying CSS classes, generating multi-level output, and supporting custom templates.

## Frameworks Introduced
- **Template Tag Architecture**: Each menu type has a dedicated template tag that fetches data, applies active classes, and renders using configurable templates
  - When to use: Displaying any menu in Wagtail templates
  - How: Load menu_tags, use {% main_menu %}, {% flat_menu %}, {% section_menu %}, {% children_menu %}
- **Active Class System**: Automatic CSS classes applied to menu items indicating current page and ancestors
  - When to use: Styling menus to show active state
  - How: Configure ACTIVE_CLASS and ACTIVE_ANCESTOR_CLASS settings; disable with apply_active_classes=False

## Key Concepts
- **main_menu tag**: Renders the main navigation menu for the current site
- **flat_menu tag**: Renders a named flat menu by handle
- **section_menu tag**: Renders a menu for the current page's section (branch of page tree)
- **children_menu tag**: Renders children of a specific page
- **sub_menu tag**: Recursive tag for rendering sub-levels within a menu
- **active_class**: CSS class applied to the current page's menu item (default: 'active')
- **ancestor_class**: CSS class applied to ancestors of the current page (default: 'ancestor')

## Mental Models
- Template tags are "menu renderers" — they fetch data, apply logic, and output HTML
- Active classes provide visual feedback: 'active' for current page, 'ancestor' for parent pages
- Sub-menus are rendered recursively; each level calls {% sub_menu %} for the next
- Template arguments override defaults; use for per-tag customization

## Anti-patterns
- **Hard-coding menu HTML**: Use template tags instead; they handle active classes, multi-level output, and page tree integration
- **Ignoring apply_active_classes**: Disabling active classes removes important UX feedback
- **Overriding max_levels globally**: Use per-tag arguments for context-specific depth control

## Code Examples
```html
{% load menu_tags %}

{# Main menu with options #}
{% main_menu max_levels=3 
   template="menus/main_nav.html" 
   sub_menu_template="menus/sub_nav.html" 
   apply_active_classes=True %}

{# Flat menu by handle #}
{% flat_menu 'footer' 
   show_menu_heading=False 
   fall_back_to_default_site_menus=True %}

{# Section menu for current page #}
{% section_menu max_levels=2 %}

{# Children of specific page #}
{% children_menu page=some_page max_levels=1 %}
```
- **What it demonstrates**: Core template tag usage patterns

```html
{# Inline sub-menus (no recursive tag call needed) #}
{% main_menu add_sub_menus_inline=True %}

{# In template, access sub-menus directly #}
{% for item in menu_items %}
    <li>
        <a href="{{ item.href }}">{{ item.text }}</a>
        {% if item.sub_menu %}
            <ul>
            {% for sub_item in item.sub_menu.items %}
                <li><a href="{{ sub_item.href }}">{{ sub_item.text }}</a></li>
            {% endfor %}
            </ul>
        {% endif %}
    </li>
{% endfor %}
```
- **What it demonstrates**: Using add_sub_menus_inline for simpler templates

## Reference Tables

| Tag | Purpose | Required Arguments | Key Optional Arguments |
|-----|---------|-------------------|----------------------|
| main_menu | Site main navigation | none | max_levels, template, sub_menu_template, apply_active_classes |
| flat_menu | Named flat menu | handle | show_menu_heading, max_levels, fall_back_to_default_site_menus |
| section_menu | Current section tree | none | max_levels, allow_repeating_parents |
| children_menu | Children of page | page | max_levels, template |
| sub_menu | Recursive sub-level | item | max_levels, template |

## Worked Example
Building a responsive navigation with dropdowns:

```html
{# base.html #}
{% load menu_tags %}
<nav class="main-nav">
    {% main_menu max_levels=2 
       template="menus/dropdown_nav.html" 
       sub_menu_template="menus/dropdown_sub.html" 
       apply_active_classes=True %}
</nav>

{# menus/dropdown_nav.html #}
<ul class="nav-list">
{% for item in menu_items %}
    <li class="nav-item {{ item.active_class }}">
        <a href="{{ item.href }}">{{ item.text }}</a>
        {% if item.has_children_in_menu %}
            {% sub_menu item template="menus/dropdown_sub.html" %}
        {% endif %}
    </li>
{% endfor %}
</ul>

{# menus/dropdown_sub.html #}
<ul class="dropdown">
{% for item in menu_items %}
    <li class="{{ item.active_class }}">
        <a href="{{ item.href }}">{{ item.text }}</a>
    </li>
{% endfor %}
</ul>
```

Result: Multi-level navigation with automatic active state styling and clean template separation.

## Key Takeaways
1. Template tags handle all menu logic: data fetching, active classes, multi-level rendering
2. Use {% main_menu %} for primary nav, {% flat_menu 'handle' %} for named menus
3. Active classes ('active', 'ancestor') provide visual feedback for current page
4. Template arguments override defaults; use for per-tag customization
5. add_sub_menus_inline=True simplifies templates by avoiding recursive {% sub_menu %} calls

## Connects To
- **Ch 6**: Custom Templates — overriding default menu templates
- **Ch 3**: Managing Main Menus — configuring main menu items
- **Ch 4**: Managing Flat Menus — creating flat menus
- **Ch 11**: Settings Reference — default template settings
