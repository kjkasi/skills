# Chapter 20: Advanced Wagtail Topics

## Core Idea
Wagtail supports advanced content management patterns including tagging (via django-taggit), internationalization (separate page trees per locale), content personalization (segment-based blocks), privacy controls (page/collection restrictions), and reference indexing (tracking cross-object references). These features build on Django's foundation while providing CMS-specific abstractions.

## Frameworks Introduced
- **Tagging System**: django-taggit + django-modelcluster for in-memory tag management during previews
  - When to use: Adding categorization/tagging to pages or snippets
  - How: Create `TaggedItemBase` subclass, add `ClusterTaggableManager` field, use `ParentalKey` for through model
- **Internationalization (i18n)**: Separate page trees per locale with `Locale` model and `translation_key` UUID
  - When to use: Multi-language content sites
  - How: Set `WAGTAIL_I18N_ENABLED=True`, configure `WAGTAIL_CONTENT_LANGUAGES`, use `i18n_patterns` in URLs
- **Content Personalization**: StreamField blocks with segment-based rendering and preview modes
  - When to use: Members-only content, campaign targeting, time-aware variants
  - How: Create `StructBlock` with `ChoiceBlock` segment, use `get_context` to read request, add preview modes
- **Privacy System**: Page and collection-level access restrictions (login, password, groups)
  - When to use: Protecting content from public access
  - How: Use Privacy control in admin, or override `get_default_privacy_setting` for automatic restrictions

## Key Concepts
- **ClusterTaggableManager**: In-memory tag management compatible with django-modelcluster for previews/revisions
- **TaggedItemBase**: Through model for many-to-many tag relationships
- **Custom Tag Models**: Inherit from `TagBase` + `ItemBase` for independent tag pools per model
- **free_tagging = False**: Disables automatic tag creation, requiring tags to exist in database first
- **Locale Model**: Stores BCP-47 language codes, linked to pages via `locale` foreign key
- **translation_key**: UUID shared across translations of the same content for cross-locale lookups
- **TranslatableMixin**: Adds `locale` and `translation_key` fields to any model for translation support
- **BootstrapTranslatableMixin**: Migration helper for adding translation fields to existing data
- **PageViewRestriction**: Model storing access restrictions (LOGIN, PASSWORD, GROUPS, NONE)
- **ReferenceIndex**: Tracks references between objects (pages, images, documents, snippets) for usage reporting
- **wagtail_reference_index_ignore**: Attribute to exclude models/fields from reference tracking
- **PersonalizationPreviewMixin**: Mixin storing selected preview segment on request for block-level access

## Mental Models
1. **Tag Pool Isolation**: Default tags are shared across all models — custom tag models create independent pools per content type
2. **Page Tree per Locale**: Each language gets its own complete page tree, not field-level translations — translations are separate pages sharing a `translation_key`
3. **Segment as Request State**: Content personalization stores segment selection on `request` object, blocks read it in `get_context`
4. **Reference Index as Best-Effort**: The index tracks common references but doesn't enforce referential integrity — use hooks for deletion blocking

## Anti-patterns
- **Using default `Tag` model for all content types**: Creates mixed autocomplete suggestions — use custom tag models for independent pools
- **Directly adding `TranslatableMixin` to models with existing data**: Requires bootstrap migration sequence — use `BootstrapTranslatableMixin` first
- **Relying on reference index for deletion prevention**: It's best-effort — implement `before_delete_page` hooks for strict enforcement
- **Not using `{% include_block %}` for StreamField**: Blocks won't receive parent context including `request` needed for personalization

## Code Examples
```python
# Adding tags to a page model
from modelcluster.fields import ParentalKey
from modelcluster.contrib.taggit import ClusterTaggableManager
from taggit.models import TaggedItemBase

class BlogPageTag(TaggedItemBase):
    content_object = ParentalKey('demo.BlogPage', on_delete=models.CASCADE, related_name='tagged_items')

class BlogPage(Page):
    tags = ClusterTaggableManager(through=BlogPageTag, blank=True)
    promote_panels = Page.promote_panels + [FieldPanel('tags')]
```
- **What it demonstrates**: Adding tagging capability to a page model with in-memory tag support

```python
# Custom tag model with restricted tagging
class BlogTag(TagBase):
    free_tagging = False
    class Meta:
        verbose_name = "blog tag"
        verbose_name_plural = "blog tags"

class TaggedBlog(ItemBase):
    tag = models.ForeignKey(BlogTag, related_name="tagged_blogs", on_delete=models.CASCADE)
    content_object = ParentalKey("demo.BlogPage", on_delete=models.CASCADE, related_name="tagged_items")
```
- **What it demonstrates**: Independent tag pool with disabled free tagging

```python
# Enabling internationalization
# settings.py
USE_I18N = True
WAGTAIL_I18N_ENABLED = True
WAGTAIL_CONTENT_LANGUAGES = LANGUAGES = [
    ("en", "English"),
    ("fr", "French"),
    ("es", "Spanish"),
]
```
- **What it demonstrates**: Basic i18n configuration for multi-language content

```python
# Translatable snippet with bootstrap migration
class Advert(TranslatableMixin, models.Model):
    name = models.CharField(max_length=255)

# For existing data: use BootstrapTranslatableMixin first
class Advert(BootstrapTranslatableMixin, models.Model):
    name = models.CharField(max_length=255)
```
- **What it demonstrates**: Making snippets translatable with proper migration support

```python
# Content personalization block
class SegmentedContentBlock(StructBlock):
    content = RichTextBlock()
    segment = ChoiceBlock(
        choices=[("all", "All visitors"), ("logged_in", "Logged-in users"), ("anonymous", "Anonymous")],
        default="all",
    )

    def get_context(self, value, parent_context=None):
        context = super().get_context(value, parent_context)
        request = parent_context.get("request") if parent_context else None
        preview = getattr(request, "personalization_preview_segment", None)
        context["is_authenticated"] = (
            preview == "logged_in" if preview
            else (request.user.is_authenticated if request else False)
        )
        return context
```
- **What it demonstrates**: Segment-based content block with preview support

```python
# Blocking deletion of referenced pages
@register("before_delete_page")
def prevent_deleting_referenced_pages(request, page):
    references = ReferenceIndex.get_references_for_object(page)
    if references.exists():
        raise PermissionDenied("This page is referenced by other content and cannot be deleted.")
```
- **What it demonstrates**: Using reference index to enforce deletion rules

## Reference Tables

| Privacy Restriction Type | Description | Configuration |
|-------------------------|-------------|---------------|
| `NONE` | No restrictions | Default |
| `LOGIN` | Any logged-in user | Privacy control in admin |
| `PASSWORD` | Shared password required | Privacy control in admin |
| `GROUPS` | Specific group membership | Privacy control in admin |

| i18n Setting | Purpose |
|-------------|---------|
| `USE_I18N` | Enable Django i18n |
| `WAGTAIL_I18N_ENABLED` | Enable Wagtail multi-language content |
| `LANGUAGES` | Frontend available languages |
| `WAGTAIL_CONTENT_LANGUAGES` | Content authoring languages |
| `LANGUAGE_CODE` | Default language code |
| `WAGTAIL_FRONTEND_LOGIN_URL` | Custom login URL for private pages |

| Reference Index Commands | Purpose |
|------------------------|---------|
| `rebuild_references_index` | Repopulate references table |
| `show_references_index` | Show indexed object counts |

## Worked Example
Setting up a multi-language blog with tagging and privacy:

1. Configure i18n: `WAGTAIL_I18N_ENABLED=True`, `WAGTAIL_CONTENT_LANGUAGES=[("en","English"),("fr","French")]`
2. Wrap URLs with `i18n_patterns` and add `LocaleMiddleware`
3. Create `BlogTag(TagBase)` with `free_tagging=False` and `TaggedBlog(ItemBase)` through model
4. Add `ClusterTaggableManager(through=TaggedBlog)` to `BlogPage`
5. Register `BlogTag` as snippet for admin management
6. Create language selector template using `page.get_translations.live` and `{% pageurl translation %}`
7. Set default privacy on sensitive pages via `get_default_privacy_setting` returning `BaseViewRestriction.GROUPS`
8. Add `wagtail.locales` to `INSTALLED_APPS` for locale management UI

## Key Takeaways
1. Default tags are shared across all models — use custom `TagBase` subclasses for independent tag pools per content type
2. Wagtail i18n creates separate page trees per locale — translations are separate pages sharing a `translation_key` UUID
3. Content personalization works via `get_context` reading request state — always use `{% include_block %}` to pass request context
4. Privacy restrictions inherit down the page tree — aliases copy their source page's restrictions
5. Reference index is best-effort — implement `before_delete_page` hooks for strict deletion prevention

## Connects To
- **Ch 6**: Images (reference index tracks image usage)
- **Ch 10**: Page models (locale and translation_key fields on all pages)
- **Ch 12**: Snippets (translatable snippets via TranslatableMixin)
- **Ch 16**: Customization (custom tag models, admin locale management)
- **Ch 17**: Performance (cache key components for translated pages)
