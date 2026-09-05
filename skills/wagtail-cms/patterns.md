# Wagtail CMS Patterns

## Page Model Inheritance
**When to use**: Creating new content types for Wagtail sites
**How**: Subclass `wagtail.models.Page`, define database fields, search index, and editor panels
**Trade-offs**: Each page type creates a database table; deep inheritance can cause performance issues

## StreamField for Mixed Content
**When to use**: Pages with variable content layouts (blog posts, articles)
**How**: Define `StreamField` with block types as `(name, block_type)` tuples
**Trade-offs**: More flexible than RichTextField but requires custom templates

## Reusable StructBlock
**When to use**: Common content patterns across multiple page types
**How**: Subclass `blocks.StructBlock` with child blocks as attributes
**Trade-offs**: Improves readability but adds Python classes to maintain

## Inline Models with ParentalKey
**When to use**: Repeating fields like related links, galleries, team members
**How**: Create `Orderable` model with `ParentalKey` to parent page, add `InlinePanel`
**Trade-offs**: Enables inline editing but increases database complexity

## Abstract Model for Reusable Inlines
**When to use**: Same inline pattern needed on multiple page types
**How**: Create abstract model with common fields, subclass for each parent
**Trade-offs**: Reduces code duplication but requires separate concrete models

## Custom Page Templates
**When to use**: Different rendering for page instances or AJAX requests
**How**: Override `get_template()` or set `ajax_template` attribute
**Trade-offs**: Enables dynamic rendering but adds template complexity

## Search Index Configuration
**When to use**: Making content searchable and filterable
**How**: Add `SearchField` and `FilterField` to `search_fields`
**Trade-offs**: Improves search but increases index size and update time

## Block Per-Instance Templates
**When to use**: Custom rendering for specific block types
**How**: Pass `template` argument to block or override `get_template()` method
**Trade-offs**: Full rendering control but requires separate template files

## StreamField Block Limits
**When to use**: Restricting content structure or ensuring required blocks
**How**: Set `min_num`, `max_num`, or `block_counts` on StreamField/StreamBlock
**Trade-offs**: Enforces content rules but reduces editorial flexibility

## Search Query Composition
**When to use**: Complex search requirements with multiple terms or fields
**How**: Use `PlainText`, `Phrase`, `Boost` classes with operators
**Trade-offs**: Powerful queries but backend-dependent (Elasticsearch recommended)

## Faceted Search
**When to use**: Taxonomy-based filtering of search results
**How**: Call `.facet(field_name)` on search results
**Trade-offs**: Useful analytics but requires indexed taxonomy fields

## ModelViewSet for CRUD
**When to use**: Admin interfaces for non-page Django models
**How**: Subclass `ModelViewSet`, register with `register_admin_viewset` hook
**Trade-offs**: Rapid development but less custom than manual views

## ChooserViewSet for Selection
**When to use**: Modal-based selection for ForeignKey fields
**How**: Subclass `ChooserViewSet`, register with hooks
**Trade-offs**: Consistent UX but requires viewset registration

## Linked Fields for Chooser Filtering
**When to use**: Dependent dropdowns or filtered chooser results
**How**: Use `linked_fields` dictionary with CSS selectors or regex patterns
**Trade-offs**: Dynamic filtering but complex configuration

## Admin View Registration
**When to use**: Custom admin views beyond snippets/pages
**How**: Create Django view, register with `register_admin_urls` hook
**Trade-offs**: Full control but requires manual URL and menu registration

## ViewSet Grouping
**When to use**: Related admin views under single menu item
**How**: Create `ViewSet` subclass with `get_urlpatterns()`, register with hooks
**Trade-offs**: Organized navigation but additional abstraction layer

## Redis Cache Backend
**When to use**: Production deployments needing fast caching
**How**: Install `django-redis`, configure `CACHES` setting
**Trade-offs**: High performance but requires Redis server

## Image Serve View
**When to use**: Pages with many images needing lazy loading
**How**: Use `{% image_url %}` tag instead of direct rendition URLs
**Trade-offs**: Faster page loads but additional HTTP requests

## Page URL Optimization
**When to use**: Navigation menus, page content with many internal links
**How**: Pass `request` to `get_url()` or use `{% pageurl %}` tag
**Trade-offs**: Reduces queries but requires request context

## Template Fragment Caching
**When to use**: Expensive template sections with stable content
**How**: Use `{% wagtailcache %}` or `{% wagtailpagecache %}` tags
**Trade-offs**: Improves performance but invalidation complexity

## WagtailPageTestCase
**When to use**: Testing page creation, routing, and rendering
**How**: Extend `WagtailPageTestCase`, use assertion methods like `assertCanCreate`
**Trade-offs**: Comprehensive testing but requires test setup

## StreamField Programmatic Access
**When to use**: Modifying content in migrations or scripts
**How**: Use `.blocks_by_name()`, `.first_block_by_name()`, or list operations
**Trade-offs**: Powerful manipulation but JSON storage limitations