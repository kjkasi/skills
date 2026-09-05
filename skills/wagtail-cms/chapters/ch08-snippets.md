# Chapter 8: Snippets

## Core Idea
Snippets are reusable Django models that are not full pages but can be managed in the Wagtail admin. They provide editable content blocks (headers, sidebars, ads) without the overhead of the page tree, and can be extended with previews, revisions, workflows, and locking.

## Frameworks Introduced
- **Snippet Registration**: Register a model as a snippet using `register_snippet` as a decorator or function (function recommended).
  - When to use: Any non-page content that editors need to manage — ads, testimonials, team members, FAQ items.
  - How: Either `@register_snippet` on the model class, or call `register_snippet(ModelOrViewSet)` in `wagtail_hooks.py`.
- **SnippetViewSet**: A `ModelViewSet` subclass for customizing snippet admin views — listing columns, filters, menus, templates.
  - When to use: When you need custom admin behavior beyond default panels.
  - How: Subclass `SnippetViewSet`, set attributes like `list_display`, `icon`, `filterset_class`, then register the viewset.
- **Mixin-based Feature Extension**: Add features to snippets by inheriting from mixin classes (`PreviewableMixin`, `RevisionMixin`, `DraftStateMixin`, `LockableMixin`, `WorkflowMixin`).
  - When to use: When snippets need page-like capabilities (previews, version history, draft/publish, locking, moderation).
  - How: Add the mixin to the model's inheritance chain and run migrations.

## Key Concepts
- **`register_snippet`**: Tells Wagtail to treat a model as a snippet, adding it to the admin Snippets menu.
- **`SnippetViewSet`**: Customizes the admin CRUD interface for a snippet (listing, editing, icons, menus).
- **`PreviewableMixin`**: Adds live preview panel to the snippet editor; requires `get_preview_template()` or `serve_preview()`.
- **`RevisionMixin`**: Saves revisions on every edit; adds `revisions` property and `latest_revision` field.
- **`DraftStateMixin`**: Adds draft/publish workflow; changes Save to "Save draft" and adds "Publish" action.
- **`LockableMixin`**: Adds lock/unlock capability to prevent concurrent editing.
- **`WorkflowMixin`**: Adds moderation workflow support (submit for review, approve/reject).
- **`ClusterableModel`**: Required parent class for snippets with inline models or `ParentalManyToManyField`.
- **`SnippetViewSetGroup`**: Groups multiple snippet types under a single admin menu item.

## Mental Models
- Snippets are "content atoms" — small, reusable pieces that live outside the page tree and can be referenced from pages.
- The mixin pattern is compositional: each mixin adds one capability, and they compose in a specific order (WorkflowMixin → DraftStateMixin → LockableMixin → RevisionMixin).
- Registration via function in `wagtail_hooks.py` is preferred over the decorator because it separates Django model concerns from Wagtail admin concerns.

## Anti-patterns
- **Using `@register_snippet` decorator when you need customization**: Use function registration with a `SnippetViewSet` instead — it's more flexible.
- **Forgetting `ClusterableModel` with inline models**: Snippets with `InlinePanel` or `ParentalManyToManyField` must inherit from `ClusterableModel`, not `models.Model`.
- **Wrong mixin order**: `LockableMixin` must come after `DraftStateMixin` but before `RevisionMixin` — wrong order causes system check failures.
- **Assuming snippets have URLs**: Unlike pages, snippets don't have frontend URLs unless you explicitly create views for them.
- **Using `TaggableManager` with `RevisionMixin`**: Use `ClusterTaggableManager` when revisions are enabled, `TaggableManager` otherwise.

## Code Examples
```python
# basic snippet — decorator approach
from wagtail.admin.panels import FieldPanel
from wagtail.snippets.models import register_snippet

@register_snippet
class Advert(models.Model):
    url = models.URLField(null=True, blank=True)
    text = models.CharField(max_length=255)
    panels = [FieldPanel("url"), FieldPanel("text")]
    def __str__(self):
        return self.text
```
- **What it demonstrates**: Simplest snippet registration with the decorator.

```python
# recommended: function registration with SnippetViewSet
# myapp/wagtail_hooks.py
from wagtail.snippets.models import register_snippet
from wagtail.snippets.views.snippets import SnippetViewSet
from myapp.models import Advert

class AdvertViewSet(SnippetViewSet):
    model = Advert
    panels = [FieldPanel("url"), FieldPanel("text")]

register_snippet(AdvertViewSet)
```
- **What it demonstrates**: Function-based registration with a custom viewset.

```python
# previewable snippet
from wagtail.models import PreviewableMixin

class Advert(PreviewableMixin, models.Model):
    url = models.URLField(null=True, blank=True)
    text = models.CharField(max_length=255)
    panels = [FieldPanel("url"), FieldPanel("text")]

    def get_preview_template(self, request, mode_name):
        return "demo/previews/advert.html"
```
- **What it demonstrates**: Adding live preview to a snippet.

```python
# snippet with revisions and drafts
from django.contrib.contenttypes.fields import GenericRelation
from wagtail.models import DraftStateMixin, RevisionMixin
from wagtail.admin.panels import PublishingPanel

class Advert(DraftStateMixin, RevisionMixin, models.Model):
    url = models.URLField(null=True, blank=True)
    text = models.CharField(max_length=255)
    _revisions = GenericRelation("wagtailcore.Revision", related_query_name="advert")
    panels = [FieldPanel("url"), FieldPanel("text"), PublishingPanel()]

    @property
    def revisions(self):
        return self._revisions
```
- **What it demonstrates**: Full draft/publish workflow with revision history.

```python
# snippet with locking and workflows
from wagtail.models import DraftStateMixin, LockableMixin, RevisionMixin, WorkflowMixin

class Advert(WorkflowMixin, DraftStateMixin, LockableMixin, RevisionMixin, models.Model):
    url = models.URLField(null=True, blank=True)
    text = models.CharField(max_length=255)
    _revisions = GenericRelation("wagtailcore.Revision", related_query_name="advert")
    workflow_states = GenericRelation(
        "wagtailcore.WorkflowState",
        content_type_field="base_content_type",
        object_id_field="object_id",
        related_query_name="advert",
        for_concrete_model=False,
    )
    panels = [FieldPanel("url"), FieldPanel("text")]
```
- **What it demonstrates**: Complete snippet with workflow moderation, locking, and drafts.

```python
# snippet with inline models
from modelcluster.fields import ParentalKey
from modelcluster.models import ClusterableModel
from wagtail.models import Orderable

class BandMember(Orderable):
    band = ParentalKey("music.Band", related_name="members", on_delete=models.CASCADE)
    name = models.CharField(max_length=255)

@register_snippet
class Band(ClusterableModel):
    name = models.CharField(max_length=255)
    panels = [FieldPanel("name"), InlinePanel("members")]
```
- **What it demonstrates**: Snippet with child models via InlinePanel.

```python
# customizing admin views with SnippetViewSet
class MemberViewSet(SnippetViewSet):
    model = Member
    icon = "user"
    list_display = ["name", "first_name", "shirt_size", UpdatedAtColumn()]
    list_per_page = 50
    copy_view_enabled = False
    inspect_view_enabled = True
    filterset_class = MemberFilterSet
    add_to_admin_menu = True
    menu_label = "Members"
```
- **What it demonstrates**: Full admin customization with listing columns, filters, and menu placement.

```python
# grouping snippets under one menu
class MarketingViewSetGroup(SnippetViewSetGroup):
    items = (AdvertViewSet, ProductViewSet)
    menu_icon = "folder-inverse"
    menu_label = "Marketing"

register_snippet(MarketingViewSetGroup)
```
- **What it demonstrates**: Organizing multiple snippet types under a single admin menu.

```html+django
{# template tag for rendering snippets #}
{% load demo_tags %}
{% adverts %}
```
- **What it demonstrates**: Including snippets in templates via custom template tags.

## Reference Tables

| Mixin | Adds | Requires |
|---|---|---|
| `PreviewableMixin` | Live preview panel | `get_preview_template()` or `serve_preview()` |
| `RevisionMixin` | Revision history, `latest_revision` | — |
| `DraftStateMixin` | Draft/publish workflow | `RevisionMixin` |
| `LockableMixin` | Lock/unlock editing | Place after `DraftStateMixin`, before `RevisionMixin` |
| `WorkflowMixin` | Moderation workflows | `DraftStateMixin`, `RevisionMixin` |

| Registration Method | When to Use |
|---|---|
| `@register_snippet` decorator | Simple snippets with no admin customization |
| `register_snippet(Model)` function | When adding a `SnippetViewSet` later |
| `register_snippet(ViewSet)` function | Full admin customization |

| SnippetViewSet Attribute | Purpose |
|---|---|
| `model` | The Django model class |
| `icon` | Admin sidebar icon |
| `list_display` | Columns in the listing view |
| `list_per_page` | Pagination size |
| `filterset_class` | Custom filter for the listing |
| `add_to_admin_menu` | Show as top-level menu item |
| `menu_label` / `menu_name` | Menu display text and URL slug |

## Worked Example
Building a team members snippet with search and admin customization:

```python
# models.py
from django.db import models
from wagtail.search import index

class Member(index.Indexed, models.Model):
    name = models.CharField(max_length=255)
    role = models.CharField(max_length=100)
    search_fields = [
        index.SearchField("name"),
        index.AutocompleteField("name"),
        index.FilterField("role"),
    ]
    def __str__(self):
        return self.name

# wagtail_hooks.py
from wagtail.snippets.models import register_snippet
from wagtail.snippets.views.snippets import SnippetViewSet

class MemberViewSet(SnippetViewSet):
    model = Member
    icon = "user"
    list_display = ["name", "role"]
    search_fields = ["name"]

register_snippet(MemberViewSet)
```

## Key Takeaways
1. Use function-based registration in `wagtail_hooks.py` for anything beyond the simplest snippets.
2. Mixins are composable — add previews, revisions, drafts, locking, and workflows independently.
3. Inherit from `ClusterableModel` when using inline models or `ParentalManyToManyField`.
4. Mixin order matters: `WorkflowMixin → DraftStateMixin → LockableMixin → RevisionMixin`.
5. `SnippetViewSet` gives you full control over the admin interface — listing, filters, menus, templates.

## Connects To
- **Ch 7**: Snippets made searchable via `index.Indexed` get a search box in the chooser interface.
- **Ch 9**: Snippets are rendered in templates via custom template tags or ForeignKey on pages.
- **Ch 10**: Snippet permissions (add, edit, publish, lock) are managed through the Groups admin.
- **Ch 6**: Snippets can reference documents via `DocumentChooserBlock` or ForeignKey.
