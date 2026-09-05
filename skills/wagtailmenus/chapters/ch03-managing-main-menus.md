# Chapter 3: Managing Main Menus

## Core Idea
Main menus define top-level navigation items for each site; sub-levels are auto-generated from the page tree based on which items have `allow_subnav=True`.

## Frameworks Introduced
- **Top-Level Item Selection**: Editors choose pages to appear as main navigation items; the page tree provides depth automatically
  - When to use: Setting up primary site navigation
  - How: Create/edit main menu in CMS, add pages as menu items, toggle allow_subnav per item
- **Per-Site Menus**: Each Wagtail site gets its own main menu; no cross-site menu sharing for main navigation
  - When to use: Multi-site Wagtail projects where each site has different navigation
  - How: Switch sites in CMS to edit each site's main menu independently

## Key Concepts
- **MainMenu**: Wagtail model storing the main navigation menu for a specific site
- **MainMenuItem**: Individual menu items within a main menu; linked to pages or custom URLs
- **allow_subnav**: Boolean field on menu items; when True, children from page tree appear in sub-menus
- **natural order**: The page tree ordering applied to sub-menu items; pages sorted by their position in the tree
- **show_in_menus**: Wagtail field on pages; only pages with this set to True appear in menus

## Mental Models
- Main menus are "selectors" — you pick which top-level pages matter, the page tree handles everything below
- The page tree is the source of truth for sub-menu structure and ordering
- `allow_subnav=True` means "show this page's children from the page tree"
- `allow_subnav=False` means "this is a leaf in the menu, don't show children"

## Anti-patterns
- **Adding sub-menu items manually**: Don't redefine page tree hierarchy in menus; let page tree drive sub-levels
- **Ignoring show_in_menus**: Forgetting to set `show_in_menus_default=False` on listing pages clutters menus
- **Reordering in menus**: Don't try to reorder sub-menu items in CMS; use page tree reordering instead

## Code Examples
```html
{% load menu_tags %}

{# Basic main menu #}
{% main_menu %}

{# With custom template and max levels #}
{% main_menu max_levels=3 template="menus/custom_main_menu.html" sub_menu_template="menus/custom_sub_menu.html" %}

{# Single level only #}
{% main_menu show_multiple_levels=False %}
```
- **What it demonstrates**: Template tag usage for rendering main menus

## Reference Tables

| Field | Type | Default | Purpose |
|-------|------|---------|---------|
| link_page | ForeignKey(Page) | null | Page this item links to |
| link_url | URLField | '' | Custom URL (overrides link_page) |
| url_append | CharField | '' | Additional path/params added to page URL |
| link_text | CharField | '' | Custom text (overrides page title) |
| allow_subnav | BooleanField | True | Show page's children in sub-menus |

## Worked Example
Setting up main navigation for a charity website:

1. In CMS, go to Settings > Main menus
2. Create/edit the main menu for your site
3. Add menu items:
   - About Us (link_page=About page, allow_subnav=True)
   - What We Do (link_page=What We Do page, allow_subnav=True)
   - Get Involved (link_page=Get Involved page, allow_subnav=True)
   - News (link_page=News listing page, allow_subnav=False)
   - Contact (link_page=Contact page, allow_subnav=False)

4. In template:
```html
{% load menu_tags %}
<nav>
    {% main_menu max_levels=2 template="menus/nav.html" %}
</nav>
```

Result: Editors see a clean navigation with sub-menus that auto-update when pages are added/moved in the tree.

## Key Takeaways
1. Main menus define only top-level items; sub-levels come from the page tree
2. `allow_subnav=True` makes a menu item expandable with its page tree children
3. Only live pages with `show_in_menus=True` appear in rendered menus
4. Each site gets its own main menu; no cross-site sharing
5. Use `autopopulate_main_menus` command to seed initial menu from existing page tree

## Connects To
- **Ch 1**: Overview — understanding menu types
- **Ch 5**: Rendering Menus — template tags for displaying menus
- **Ch 7**: MenuPage Model — repeating pages in multi-level menus
- **Ch 11**: Settings Reference — main menu configuration options
