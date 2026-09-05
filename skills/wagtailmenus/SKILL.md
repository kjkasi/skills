---
name: wagtailmenus
description: "Knowledge base from \"wagtailmenus documentation\" by Jazzband. Use when implementing menus in Wagtail CMS projects, configuring menu templates, customizing menu models, or troubleshooting menu rendering."
---

<!-- argument-hint: [menu type, template tag, setting name, or model name] -->

# wagtailmenus
**Author**: Jazzband | **Source**: github.com/jazzband/wagtailmenus | **Generated**: 2026-09-03

## How to Use This Skill

- **Without arguments** — load core frameworks for reference
- **With a topic** — ask about `main_menu`, `flat_menu`, `MenuPage`, or another indexed topic; I find and read the relevant chapter
- **With chapter** — ask for `ch01`; I load that specific chapter
- **Browse** — ask "what chapters do you have?" to see the full index

When you ask about a topic not covered in Core Frameworks below, I will read
the relevant chapter file before answering.

---

## Core Frameworks & Mental Models

### Menu Architecture
- **Main Menus**: Site-specific top-level navigation; editors select pages, page tree provides depth. Use `{% main_menu %}` tag.
- **Flat Menus**: CMS-managed menus for any context (footer, sidebar); identified by handle. Use `{% flat_menu 'handle' %}` tag.
- **Section Menus**: Auto-generated from current page's tree branch. Use `{% section_menu %}` tag.

### Menu Item Model
- **link_page**: ForeignKey to target page (optional)
- **link_url**: Custom URL field (optional, supports anchors like #signup)
- **url_append**: Additional path/params added to page URL
- **link_text**: Custom display text (overrides page title)
- **allow_subnav**: Controls whether children from page tree appear in sub-menus

### Page Tree Integration
- **Natural order**: Page tree ordering drives sub-menu structure and sequence
- **show_in_menus**: Only pages with this=True appear in rendered menus at any level
- **allow_subnav=True**: "Show this page's children from the page tree"
- **allow_subnav=False**: "This is a leaf in the menu, don't show children"

### Template Tag System
- **main_menu**: Renders site main navigation with active classes
- **flat_menu**: Renders named flat menu by handle
- **section_menu**: Renders current page's section tree
- **children_menu**: Renders children of specific page
- **sub_menu**: Recursive tag for sub-levels (inherited by recursive calls)

### Active Class System
- **active_class**: CSS class for current page's menu item (default: 'active')
- **ancestor_class**: CSS class for ancestor pages (default: 'ancestor')
- **apply_active_classes**: Tag argument to enable/disable (default: True)

### Menu Customization
- **MenuPage**: Abstract model for pages that repeat in their own sub-menus (repeat_in_menu)
- **MenuPageMixin**: Mixin for adding repetition to existing page types
- **AbstractLinkPage**: Lightweight link-only pages for menus
- **Custom models**: Subclass abstract models, set related_name, configure WAGTAILMENUS_*_RELATED_NAME

### Template Discovery
- **Auto-discovery**: wagtailmenus searches: tag argument → site-specific → project-level → app defaults
- **Site-specific templates**: Enable WAGTAILMENUS_SITE_SPECIFIC_TEMPLATE_DIRS for multi-site projects
- **Sub-menu inheritance**: Set template once on main tag; applies to all recursive calls

### Settings Hierarchy
- **Admin settings**: FLATMENU_MENU_ICON, *_EDITABLE_IN_WAGTAILADMIN, *_ADMIN_CLASS
- **Template settings**: DEFAULT_*_TEMPLATE, SITE_SPECIFIC_TEMPLATE_DIRS
- **Model settings**: *_MODEL, *_RELATED_NAME for custom implementations
- **Rendering settings**: ACTIVE_CLASS, ACTIVE_ANCESTOR_CLASS, DEFAULT_ADD_SUB_MENUS_INLINE

### Hook System
- **before_render_menu**: Signal fired after items fetched, before template render
- **Usage**: Global filtering, analytics, A/B testing, permission-based visibility
- **Pattern**: Connect handler, modify menu_items, return modified list

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-overview.md) | Overview and Key Concepts | Top-Level Item Selection, Flat Menus, MenuPage Model |
| [ch02](chapters/ch02-installation.md) | Installation | Django App Integration, autopopulate_main_menus |
| [ch03](chapters/ch03-managing-main-menus.md) | Managing Main Menus | MainMenu, MainMenuItem, allow_subnav, Per-Site Menus |
| [ch04](chapters/ch04-managing-flat-menus.md) | Managing Flat Menus | FlatMenu, FlatMenuItem, handle, fall_back_to_default_site_menus |
| [ch05](chapters/ch05-rendering-menus.md) | Rendering Menus with Template Tags | main_menu, flat_menu, section_menu, children_menu, sub_menu |
| [ch06](chapters/ch06-custom-templates.md) | Custom Templates | Template Auto-Discovery, Site-Specific Templates, Sub-menu Inheritance |
| [ch07](chapters/ch07-menupage-model.md) | MenuPage Model | MenuPage, repeat_in_menu, repeat_menu_text, MenuPageMixin |
| [ch08](chapters/ch08-abstractlinkpage.md) | AbstractLinkPage Model | AbstractLinkPage, link_page, link_url, Automatic Link Hiding |
| [ch09](chapters/ch09-custom-menu-classes.md) | Custom Menu Classes and Models | Model Override Pattern, Class Override Pattern, related_name |
| [ch10](chapters/ch10-hooks.md) | Hooks | before_render_menu signal, Runtime Modification |
| [ch11](chapters/ch11-settings-reference.md) | Settings Reference | Admin Settings, Template Settings, Model Settings, Rendering Settings |
| [ch12](chapters/ch12-contributing.md) | Contributing | GitHub Workflow, Release Process, Semantic Versioning |

## Topic Index

- **AbstractLinkPage** → ch08
- **AbstractMainMenu** → ch09
- **AbstractMainMenuItem** → ch09
- **AbstractFlatMenu** → ch09
- **AbstractFlatMenuItem** → ch09
- **Active Class** → ch05, ch11
- **allow_subnav** → ch03, ch07
- **autopopulate_main_menus** → ch02
- **ChildrenMenu** → ch09
- **children_menu** → ch05
- **context_processors.wagtailmenus** → ch02
- **Custom Menu Classes** → ch09
- **Custom Templates** → ch06
- **DEFAULT_*_TEMPLATE** → ch06, ch11
- **fall_back_to_default_site_menus** → ch04
- **FlatMenu** → ch04
- **FlatMenuItem** → ch04
- **flat_menu** → ch05
- **FLATMENU_MENU_ICON** → ch11
- **handle** → ch04
- **Hooks** → ch10
- **link_page** → ch08
- **link_text** → ch08
- **link_url** → ch08
- **main_menu** → ch05
- **MainMenu** → ch03
- **MainMenuItem** → ch03
- **max_levels** → ch05
- **MenuPage** → ch07
- **MenuPageMixin** → ch07
- **Page Tree** → ch01, ch03
- **repeat_in_menu** → ch07
- **repeat_menu_text** → ch07
- **section_menu** → ch05
- **SectionMenu** → ch09
- **show_in_menus** → ch01, ch03
- **SITE_SPECIFIC_TEMPLATE_DIRS** → ch06, ch11
- **sub_menu** → ch05
- **Template Discovery** → ch06
- **url_append** → ch08
- **wagtailmenus** → ch01

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers wagtailmenus documentation only. For hands-on implementation in your codebase, combine with project-specific tools and Wagtail documentation. For topics beyond this skill, check related skills or ask the agent directly.
