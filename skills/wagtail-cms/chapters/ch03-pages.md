# Chapter 3: Page Models

## Core Idea
Every page type in Wagtail is a Django model inheriting from `wagtail.models.Page`, forming a hierarchical tree structure where pages can have parent/child relationships and custom fields.

## Frameworks Introduced
- **Page Model**: Django model representing a page type with fields, panels, and business rules
  - When to use: Creating any new page type in Wagtail
  - How: Inherit from Page, define fields, configure panels, set parent/subpage types
- **Inline Models**: Related models nested within a page using ParentalKey
  - When to use: Repeating content like related links, carousel items, tags
  - how: Create model with ParentalKey to page, add InlinePanel to content_panels

## Key Concepts
- **Page**: Base class for all page types; each page is stored in both Page table and its specific model table
- **Multi-table Inheritance**: Django feature allowing multiple page models in same tree
- **specific**: Property converting Page instance to its specific model type (may cause DB lookup)
- **parent_page_types**: List of page types where this page can be created as child
- **subpage_types**: List of page types that can be created as children of this page
- **content_panels**: Fields shown in main content editing tab
- **promote_panels**: Fields shown in metadata/SEO tab
- **settings_panels**: Fields shown in settings tab

## Mental Models
- Use `parent_page_types` to enforce site structure (e.g., blog posts only under blog index)
- Use `subpage_types` to prevent unwanted child pages (e.g., no children allowed)
- Think of `specific` property as "downcasting" from generic Page to specific type
- Use `get_context` to add extra template variables without extra database queries

## Anti-patterns
- **Field names matching class names**: Causes Django relation errors; append "Page" to model names
- **Using Meta.ordering on Page models**: Won't work; apply ordering in QuerySet explicitly
- **Not using specific when needed**: Accessing custom fields requires downcasting to specific type
- **Overusing ForeignKey without related_name**: Can cause reverse relation conflicts

## Code Examples
```python
# Complete page model example
class BlogPage(Page):
    body = RichTextField()
    date = models.DateField("Post date")
    feed_image = models.ForeignKey(
        "wagtailimages.Image",
        null=True,
        blank=True,
        on_delete=models.SET_NULL,
        related_name="+",
    )
    
    search_fields = Page.search_fields + [
        index.SearchField("body"),
        index.FilterField("date"),
    ]
    
    content_panels = Page.content_panels + [
        FieldPanel("date"),
        FieldPanel("body"),
        InlinePanel("related_links"),
    ]
    
    promote_panels = [
        MultiFieldPanel(Page.promote_panels, "Common page configuration"),
        FieldPanel("feed_image"),
    ]
    
    parent_page_types = ["blog.BlogIndex"]
    subpage_types = []
```
- **What it demonstrates**: Complete page model with fields, search indexing, panels, and type restrictions

```python
# Inline model for related links
class BlogPageRelatedLink(Orderable):
    page = ParentalKey(BlogPage, on_delete=models.CASCADE, related_name="related_links")
    name = models.CharField(max_length=255)
    url = models.URLField()
    
    panels = [
        FieldPanel("name"),
        FieldPanel("url"),
    ]
```
- **What it demonstrates**: Creating reusable inline content with ParentalKey relationship

## Reference Tables
| Panel Type | Purpose | Example Use |
|------------|---------|-------------|
| FieldPanel | Edit single model field | title, body, date |
| MultiFieldPanel | Group related fields | Hero section fields |
| InlinePanel | Show inline model instances | Related links, carousel items |
| FieldRowPanel | Arrange fields in row | Side-by-side fields |
| PageChooserPanel | Link to pages with type filtering | Related pages |

## Worked Example
Converting Page to specific type:
```python
# Get all pages (generic Page objects)
>>> Page.objects.all()
[<Page: Homepage>, <Page: About us>, <Page: Blog>]

# Convert to specific type to access custom fields
>>> page = Page.objects.get(title="A Blog post")
>>> page.specific
<BlogPage: A Blog post>
>>> page.specific.body  # Now accessible
```

Customizing template context:
```python
class BlogIndexPage(Page):
    def get_context(self, request, *args, **kwargs):
        context = super().get_context(request, *args, **kwargs)
        context["blog_entries"] = BlogPage.objects.child_of(self).live()
        return context
```

## Key Takeaways
1. Each page type is a separate database table through Django's multi-table inheritance
2. Use `specific` property when you need access to custom fields from generic Page queryset
3. Set `parent_page_types` and `subpage_types` to enforce site structure rules
4. Inline models use ParentalKey from django-modelcluster, not Django's ForeignKey
5. Page URLs can be customized by overriding `get_url_parts()` method

## Connects To
- **Ch 1**: Getting started concepts applied when creating first page models
- **Ch 4**: StreamField is often used as a field type within page models
- **Ch 5**: Image fields in pages use the image handling system covered in images chapter