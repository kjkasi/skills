# Chapter 1: Overview and Key Concepts

## Core Idea
wagtailmenus gives editors explicit control over top-level menu items while dynamically generating sub-menus from the page tree, solving the fragility of tree-driven navigation.

## Frameworks Introduced
- **Top-Level Item Selection**: Editors choose which pages appear as top-level menu items; sub-menus are auto-generated from the page tree structure
  - When to use: Any Wagtail project where main navigation needs editorial control
  - How: Define top-level items in CMS, set `allow_subnav=True` for items that should show children
- **Flat Menus**: CMS-managed menus for any context (footer, sidebar, secondary nav) that can link to pages or custom URLs
  - When to use: Non-primary navigation areas, footer links, or any menu not derived from the page tree
  - How: Create via Wagtail admin under Settings > Flat menus, assign a unique handle
- **MenuPage Model**: Extends Wagtail's Page to allow pages to repeat alongside their children in multi-level menus
  - When to use: When a parent page should appear again in sub-menus alongside its children
  - How: Subclass `wagtailmenus.models.MenuPage` instead of `wagtail.core.models.Page`

## Key Concepts
- **Main Menu**: Site-specific menu defining top-level navigation items; sub-levels generated from page tree
- **Flat Menu**: Reusable menu not tied to page tree; can be shared across sites or site-specific
- **Menu Item**: Individual link within a menu; can point to a page, custom URL, or both
- **allow_subnav**: Field on menu items controlling whether children from page tree appear beneath
- **show_in_menus**: Wagtail field controlling page visibility in all menus (default True)
- **max_levels**: Controls how many levels deep a menu renders (default from menu settings)

## Mental Models
- Use flat menus when you need editorial control over non-primary navigation (footer, sidebar, help menus)
- Think of main menus as "top-level selectors" — you pick the surface, the page tree provides the depth
- Menu items are connectors: they link pages or URLs into the menu structure, not content containers

## Anti-patterns
- **Hard-coding menu links**: Changes require code deploys; use CMS-managed flat menus instead
- **Overriding sub-menu structure**: Don't redefine page tree hierarchy in menus; let page tree drive sub-levels
- **Ignoring show_in_menus**: Forgetting to set `show_in_menus_default=False` on listing pages (news, events) clutters menus with unintended items

## Code Examples
```python
# settings/base.py - minimal wagtailmenus setup
INSTALLED_APPS = [
    ...
    'wagtailmenus',
]

TEMPLATES = [{
    'OPTIONS': {
        'context_processors': [
            ...
            'wagtailmenus.context_processors.wagtailmenus',
        ],
    },
}]
```
- **What it demonstrates**: Essential settings for enabling wagtailmenus in a Django project

## Reference Tables

| Menu Type | Source of Items | Multi-level Support | Site Scope |
|-----------|----------------|---------------------|------------|
| Main Menu | CMS-selected pages | Auto-generated from page tree | Per-site |
| Flat Menu | CMS-defined items | Configurable (max_levels) | Per-site or shared |
| Section Menu | Current page tree branch | Auto-generated | Per-site |

## Worked Example
A charity site needs:
- **Main nav**: About, What We Do, Get Involved, News, Contact (selected by editors)
- **Footer**: Quick Links, Contact Info, Social (flat menu with custom URLs)
- **Sidebar on "What We Do" pages**: Sub-pages of current section (section_menu)

Setup:
1. Create main menu items in CMS: About, What We Do, Get Involved, News, Contact
2. Set `allow_subnav=True` on About, What We Do, Get Involved
3. Create flat menu with handle `footer`, add Quick Links, Contact Info items
4. In template: `{% main_menu %}` for nav, `{% flat_menu 'footer' %}` for footer, `{% section_menu %}` for sidebar

Result: Editors can reorder main nav items and add footer links without touching code. Sub-menus auto-update when pages are moved.

## Key Takeaways
1. Main menus let editors pick top-level items; sub-levels come from the page tree automatically
2. Flat menus handle any non-primary navigation and support custom URLs
3. Only pages with `show_in_menus=True` appear in rendered menus at any level
4. The page tree is the source of truth for sub-menu structure and ordering
5. CMS-managed menus eliminate hard-coded navigation and enable editor autonomy

## Connects To
- **Ch 3**: Managing Main Menus — detailed setup for main navigation
- **Ch 4**: Managing Flat Menus — creating and configuring flat menus
- **Ch 5**: Rendering Menus — template tags for displaying menus
- **Ch 7**: MenuPage Model — repeating pages in multi-level menus
