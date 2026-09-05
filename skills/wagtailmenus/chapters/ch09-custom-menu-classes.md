# Chapter 9: Custom Menu Classes and Models

## Core Idea
wagtailmenus supports deep customization through abstract models and configurable classes, allowing you to override menu and menu item models, add custom fields, and modify rendering behavior.

## Frameworks Introduced
- **Model Override Pattern**: Subclass abstract models (AbstractMainMenu, AbstractFlatMenuItem, etc.) to add custom fields and behavior
  - When to use: Adding images, descriptions, translations, or other custom fields to menu items
  - How: Create custom model, set related_name, configure WAGTAILMENUS_*_RELATED_NAME setting
- **Class Override Pattern**: Use settings to swap out menu classes (SectionMenu, ChildrenMenu, etc.) for custom rendering logic
  - When to use: Modifying how menus query pages, filter results, or generate data
  - How: Create custom class, set WAGTAILMENUS_*_CLASS setting to import path

## Key Concepts
- **AbstractMainMenu/AbstractMainMenuItem**: Base models for main menu customization
- **AbstractFlatMenu/AbstractFlatMenuItem**: Base models for flat menu customization
- **SectionMenu**: Plain Python class (not Django model) driving section_menu tag behavior
- **ChildrenMenu**: Plain Python class driving children_menu tag behavior
- **WAGTAILMENUS_*_RELATED_NAME**: Settings linking custom item models to menu models
- **WAGTAILMENUS_*_CLASS**: Settings swapping out menu classes for custom behavior

## Mental Models
- Model overrides add fields; class overrides change behavior
- Related names connect custom item models to menu models; must match ParentalKey related_name
- Menu classes are plain Python; override methods to change query behavior
- Custom models don't replace defaults in database; old tables remain intact

## Anti-patterns
- **Overriding without need**: Only customize when you have specific requirements; defaults work well
- **Mismatching related_names**: ParentalKey related_name must match WAGTAILMENUS_*_RELATED_NAME setting
- **Forgetting migrations**: Always run makemigrations and migrate after model changes

## Code Examples
```python
# models.py - custom main menu item with image
from django.db import models
from modelcluster.fields import ParentalKey
from wagtail.images import get_image_model_string
from wagtail.images.edit_handlers import ImageChooserPanel
from wagtailmenus.models import AbstractMainMenuItem

class CustomMainMenuItem(AbstractMainMenuItem):
    menu = ParentalKey(
        'wagtailmenus.MainMenu',
        on_delete=models.CASCADE,
        related_name="custom_menu_items",
    )
    image = models.ForeignKey(
        get_image_model_string(),
        blank=True,
        null=True,
        on_delete=models.SET_NULL,
    )
    hover_description = models.CharField(max_length=250, blank=True)
    
    panels = (
        PageChooserPanel('link_page'),
        FieldPanel('link_url'),
        FieldPanel('url_append'),
        FieldPanel('link_text'),
        ImageChooserPanel('image'),
        FieldPanel('hover_description'),
        FieldPanel('allow_subnav'),
    )
```
- **What it demonstrates**: Adding image and description fields to menu items

```python
# settings/base.py - register custom model
WAGTAILMENUS_MAIN_MENU_ITEMS_RELATED_NAME = "custom_menu_items"
```
- **What it demonstrates**: Linking custom item model to main menu

```python
# settings/base.py - custom section menu class
WAGTAILMENUS_SECTION_MENU_CLASS = 'myapp.menus.CustomSectionMenu'
```
- **What it demonstrates**: Swapping out menu class for custom behavior

## Reference Tables

| Setting | Default | Purpose |
|---------|---------|---------|
| MAIN_MENU_MODEL | wagtailmenus.MainMenu | Custom main menu model |
| MAIN_MENU_ITEMS_RELATED_NAME | menu_items | Related name for custom main menu items |
| FLAT_MENU_MODEL | wagtailmenus.FlatMenu | Custom flat menu model |
| FLAT_MENU_ITEMS_RELATED_NAME | menu_items | Related name for custom flat menu items |
| SECTION_MENU_CLASS | wagtailmenus.models.menus.SectionMenu | Custom section menu class |
| CHILDREN_MENU_CLASS | wagtailmenus.models.menus.ChildrenMenu | Custom children menu class |

## Worked Example
Adding multi-language support to flat menus:

1. Create custom models:
```python
class TranslatedFlatMenu(AbstractFlatMenu):
    heading_de = models.CharField(max_length=255, blank=True)
    heading_fr = models.CharField(max_length=255, blank=True)
    # ... content_panels with translated fields

class TranslatedFlatMenuItem(AbstractFlatMenuItem):
    menu = ParentalKey(TranslatedFlatMenu, on_delete=models.CASCADE)
    link_text_de = models.CharField(max_length=255, blank=True)
    link_text_fr = models.CharField(max_length=255, blank=True)
```

2. Configure settings:
```python
WAGTAILMENUS_FLAT_MENU_MODEL = 'myapp.TranslatedFlatMenu'
WAGTAILMENUS_FLAT_MENU_ITEMS_RELATED_NAME = 'custom_menu_items'
```

3. Run migrations

4. Result: Flat menus now support translated headings and link text per language

## Key Takeaways
1. Model overrides add custom fields; class overrides change behavior
2. Related names must match between ParentalKey and WAGTAILMENUS_*_RELATED_NAME settings
3. Menu classes are plain Python; override methods to change query behavior
4. Custom models don't replace defaults in database; old tables remain intact
5. Always run migrations after model changes

## Connects To
- **Ch 3**: Managing Main Menus — customizing main menu items
- **Ch 4**: Managing Flat Menus — customizing flat menu items
- **Ch 11**: Settings Reference — all override settings
