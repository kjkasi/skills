# Chapter 4: StreamField

## Core Idea
StreamField provides flexible content modeling by treating content as a sequence of typed blocks, balancing developer need for structured data with editor need for layout flexibility.

## Frameworks Introduced
- **StructBlock**: Groups child blocks into compound blocks with named fields
  - When to use: Complex content types like person profiles, image captions, events
  - How: Define child blocks as attributes or list of tuples, subclass for reuse
- **ListBlock**: Repeating block allowing multiple instances of same type
  - When to use: Galleries, lists of items, repeated content sections
  - How: Wrap child block type in ListBlock
- **StreamBlock**: Defines set of block types that can be mixed in any sequence
  - When to use: Common content sets across multiple page types
  - How: Subclass StreamBlock, define block types as attributes

## Key Concepts
- **StreamField**: Model field storing content as JSON sequence of typed blocks
- **Block Types**: Built-in (CharBlock, RichTextBlock) and custom (StructBlock subclasses)
- **Block Template**: HTML template for rendering individual block types
- **Block Icons**: Visual representations in admin block picker menu
- **Block Groups**: Organize blocks in admin menu under headings
- **Block Counts**: Limit min/max instances of specific block types
- **Block Previews**: Show preview in block picker when adding blocks

## Mental Models
- Use StreamField when content structure varies between pages but needs semantic meaning
- Use RichTextField when content is mostly text with basic formatting only
- Think of blocks as "content atoms" with type safety and rendering control
- Use StreamBlock for common content sets across multiple page types

## Anti-patterns
- **Using StreamField for simple fields**: Overkill for basic text/number fields
- **Not indexing blocks for search**: Content won't be searchable without search_fields configuration
- **Missing block templates**: Blocks render with minimal HTML without custom templates
- **Hardcoding block types**: Use StreamBlock subclasses for reuse across projects

## Code Examples
```python
# Basic StreamField usage
class BlogPage(Page):
    body = StreamField(
        [
            ("heading", blocks.CharBlock(form_classname="title")),
            ("paragraph", blocks.RichTextBlock()),
            ("image", ImageBlock()),
        ]
    )
```
- **What it demonstrates**: Simple StreamField with three basic block types

```python
# Custom StructBlock subclass
class PersonBlock(blocks.StructBlock):
    first_name = blocks.CharBlock()
    surname = blocks.CharBlock()
    photo = ImageBlock(required=False)
    biography = blocks.RichTextBlock()
    
    class Meta:
        icon = "user"
        template = "myapp/blocks/person.html"
```
- **What it demonstrates**: Creating reusable custom block with icon and template

```python
# StreamBlock for common content across pages
class CommonContentBlock(blocks.StreamBlock):
    heading = blocks.CharBlock(form_classname="title")
    paragraph = blocks.RichTextBlock()
    image = ImageBlock()

class BlogPage(Page):
    body = StreamField(CommonContentBlock())
```
- **What it demonstrates**: Reusable StreamBlock for consistent content across page types

```python
# Limiting block counts
body = StreamField(
    [
        ("heading", blocks.CharBlock(form_classname="title")),
        ("paragraph", blocks.RichTextBlock()),
        ("image", ImageBlock()),
    ],
    block_counts={
        "heading": {"min_num": 1, "max_num": 3},
    },
)
```
- **What it demonstrates**: Enforcing business rules on block instances

## Reference Tables
| Block Type | Purpose | Template Variable |
|------------|---------|-------------------|
| CharBlock | Single-line text | `{{ value }}` |
| TextBlock | Multi-line text | `{{ value }}` |
| RichTextBlock | WYSIWYG editor | `{{ value|richtext }}` |
| IntegerBlock | Whole numbers | `{{ value }}` |
| BooleanBlock | True/false | `{{ value }}` |
| DateBlock | Date picker | `{{ value }}` |
| DateTimeBlock | Date/time picker | `{{ value }}` |
| TimeBlock | Time picker | `{{ value }}` |
| URLBlock | URL input | `{{ value }}` |
| EmailBlock | Email input | `{{ value }}` |
| ChoiceBlock | Dropdown select | `{{ value }}` |
| ImageBlock | Image selector | `{% image value %}` |
| EmbedBlock | External embed | `{{ value }}` |
| PageChooserBlock | Page link | `{% pageurl value %}` |
| StructBlock | Grouped fields | `{{ value.field_name }}` |
| ListBlock | Repeating blocks | `{% for item in value %}` |
| StreamBlock | Mixed blocks | `{% for block in value %}` |

## Worked Example
Creating a captioned image block:
```python
# Block definition
class CaptionedImageBlock(StructBlock):
    image = ImageBlock(required=True)
    caption = CharBlock(required=False)
    attribution = CharBlock(required=False)
    
    class Meta:
        icon = "image"
        template = "base/blocks/captioned_image_block.html"
```

Template rendering:
```html+django
{% load wagtailimages_tags %}
<figure>
    {% image self.image fill-600x338 loading="lazy" %}
    <figcaption>{{ self.caption }} - {{ self.attribution }}</figcaption>
</figure>
```

Using in page model:
```python
class PortfolioPage(Page):
    body = StreamField(
        [
            ("image", CaptionedImageBlock()),
            ("heading", HeadingBlock()),
            ("paragraph", blocks.RichTextBlock()),
        ]
    )
```

## Key Takeaways
1. StreamField stores content as JSON, preserving semantic structure unlike RichTextField
2. Custom blocks should be defined in separate files for reusability
3. Each block type needs a template for proper rendering
4. Use `{% include_block %}` tag to render blocks in templates
5. Block previews help editors choose the right block type
6. Search indexing requires explicit configuration in search_fields

## Connects To
- **Ch 3**: StreamField is commonly used as a field type within page models
- **Ch 2**: Tutorial demonstrates practical StreamField usage for portfolio content
- **Ch 5**: ImageBlock within StreamField uses image handling system