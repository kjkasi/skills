# Chapter 7: MenuPage Model

## Core Idea
MenuPage extends Wagtail's Page model to allow pages to appear again alongside their children in multi-level menus, solving the problem of important pages becoming just "toggles" in dropdown navigation.

## Frameworks Introduced
- **Page Repetition Pattern**: MenuPage allows a page to appear both as a parent and as an item in its own sub-menu
  - When to use: When a page should be clickable AND show its children in multi-level menus
  - How: Subclass MenuPage instead of Page; configure repeat_in_menu and repeat_menu_text per page instance
- **MenuPageMixin**: Alternative to full MenuPage model; can be added to existing page types
  - When to use: When you want repetition behavior without replacing your page model base class
  - How: Add MenuPageMixin to your page model's inheritance chain

## Key Concepts
- **MenuPage**: Abstract model extending wagtail.core.models.Page with menu repetition fields
- **repeat_in_menu**: Boolean field; when True, page appears in its own sub-menu alongside children
- **repeat_menu_text**: CharField; custom text for the repeated item (defaults to page title)
- **MenuPageMixin**: Mixin class providing repetition behavior without full model replacement
- **MenuPageMenuItem**: Internal model linking MenuPage instances to menu items

## Mental Models
- MenuPage solves the "clickable parent" problem: pages can be both navigation containers and navigation targets
- Think of repeat_in_menu as "show this page again in its own children's list"
- The repeated item is a separate menu item with its own link, not a copy of the parent
- MenuPageMixin is for existing projects; MenuPage is for new projects starting fresh

## Anti-patterns
- **Using MenuPage for all pages**: Only use for pages that need to appear in sub-menus; regular pages work fine with default Page
- **Ignoring repeat_menu_text**: Default repeats the page title; customize for clearer navigation labels
- **Confusing repeat_in_menu with show_in_menus**: show_in_menus controls visibility everywhere; repeat_in_menu only affects sub-menu appearance

## Code Examples
```python
# models.py - using MenuPage
from wagtailmenus.models import MenuPage

class SectionPage(MenuPage):
    """A page that can repeat in its own sub-menu"""
    intro = models.TextField(blank=True)
    
    content_panels = MenuPage.content_panels + [
        FieldPanel('intro'),
    ]
```
- **What it demonstrates**: Creating a page type that supports menu repetition

```python
# models.py - using MenuPageMixin for existing page types
from wagtailmenus.models import MenuPageMixin

class ArticlePage(Page, MenuPageMixin):
    """Existing page type with menu repetition added"""
    body = RichTextField(blank=True)
    
    # MenuPageMixin adds repeat_in_menu and repeat_menu_text fields
```
- **What it demonstrates**: Adding repetition behavior to existing page types

```html
{# In menu template - checking for repeated items #}
{% for item in menu_items %}
    <li class="{{ item.active_class }}">
        <a href="{{ item.href }}">{{ item.text }}</a>
        {% if item.has_children_in_menu %}
            {% sub_menu item %}
        {% endif %}
    </li>
{% endfor %}
```
- **What it demonstrates**: Menu templates work the same with MenuPage items

## Reference Tables

| Field | Type | Default | Purpose |
|-------|------|---------|---------|
| repeat_in_menu | BooleanField | False | Show page in its own sub-menu |
| repeat_menu_text | CharField | '' | Custom text for repeated item |
| show_in_default_menu | BooleanField | True | Include in default menu generation |

## Worked Example
A product catalog where category pages should be clickable AND show their products:

1. Create SectionPage extending MenuPage:
```python
class ProductCategory(MenuPage):
    description = models.TextField(blank=True)
```

2. Create product pages under categories in the page tree

3. In CMS, edit the ProductCategory page:
   - Check "Repeat in menu"
   - Set "Repeat menu text" to "View All Products"

4. Result in menu:
```
Products (main menu item)
├── View All Products (repeated item - links to category page)
├── Product A
├── Product B
└── Product C
```

Users can click "Products" to see the category page, or click individual products. "View All Products" provides a quick link back to the category overview.

## Key Takeaways
1. MenuPage solves the "clickable parent" problem in multi-level menus
2. repeat_in_menu makes a page appear in its own sub-menu alongside children
3. repeat_menu_text customizes the label for the repeated item
4. MenuPageMixin adds repetition behavior to existing page types without replacing Page
5. Only use MenuPage for pages that need sub-menu appearance; regular pages work fine with Page

## Connects To
- **Ch 1**: Overview — understanding menu repetition concept
- **Ch 3**: Managing Main Menus — how repeat_in_menu affects main navigation
- **Ch 5**: Rendering Menus — template tags handle MenuPage items automatically
