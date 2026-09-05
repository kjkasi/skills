# Chapter 2: Tutorial - Building a Portfolio Site

## Core Idea
This tutorial transforms a basic Wagtail blog into a fully functional portfolio site, teaching StreamField, forms, snippets, search, and deployment through hands-on implementation.

## Frameworks Introduced
- **StreamField**: Flexible content modeling with blocks for mixed content types
  - When to use: Pages with varied content layouts (paragraphs, images, embeds, etc.)
  - How: Define block types in StreamField, create templates for each block
- **Form Builder**: Create data collection forms without writing Django form code
  - When to use: Contact forms, surveys, registration forms
  - How: Use AbstractFormField and AbstractEmailForm, create form templates
- **Snippets**: Reusable content models that aren't full pages
  - When to use: Footer text, navigation settings, sidebar content
  - How: Register with @register_snippet, add mixins for drafts/revisions

## Key Concepts
- **StreamField**: Block-based content field allowing flexible layouts with semantic meaning
- **StructBlock**: Groups child blocks into a single compound block
- **ListBlock**: Repeating block allowing multiple instances of same block type
- **StreamBlock**: Defines set of child block types that can be mixed in any sequence
- **Snippet**: Non-page Django model registered with Wagtail for reusable content
- **Template Tag**: Custom Django template tag for rendering snippets across pages

## Mental Models
- Use StreamField when content structure varies between pages but needs semantic meaning
- Use RichTextField when content is mostly text with basic formatting
- Think of snippets as "content atoms" that can appear across multiple pages
- Use settings models for site-wide configuration (navigation, footer text)

## Anti-patterns
- **Overusing RichText for structured data**: Loss of semantic value makes content hard to query and reuse
- **Not using search index fields**: Without indexing fields in search_fields, content won't be searchable
- **Hardcoding footer text**: Use snippets or settings so content editors can update without code changes

## Code Examples
```python
# StreamField with custom blocks
class PortfolioPage(Page):
    parent_page_types = ["home.HomePage"]
    
    body = StreamField(
        PortfolioStreamBlock(),
        blank=True,
        use_json_field=True,
        help_text="Use this section to list your projects and skills.",
    )
    
    content_panels = Page.content_panels + [
        FieldPanel("body"),
    ]
```
- **What it demonstrates**: StreamField usage with custom block types and parent page restrictions

```python
# Form page with email capability
class FormPage(AbstractEmailForm):
    intro = RichTextField(blank=True)
    thank_you_text = RichTextField(blank=True)
    
    content_panels = AbstractEmailForm.content_panels + [
        FormSubmissionsPanel(),
        FieldPanel("intro"),
        InlinePanel("form_fields"),
        FieldPanel("thank_you_text"),
        MultiFieldPanel(
            [
                FieldRowPanel(
                    [
                        FieldPanel("from_address"),
                        FieldPanel("to_address"),
                    ]
                ),
                FieldPanel("subject"),
            ],
            "Email",
        ),
    ]
```
- **What it demonstrates**: Creating a form page that sends submissions via email

## Reference Tables
| Block Type | Purpose | Use Case |
|------------|---------|----------|
| StructBlock | Group child blocks together | Person block, image with caption |
| ListBlock | Repeat same block type multiple times | Gallery of images |
| StreamBlock | Mix different block types in any sequence | Carousel with images/videos |
| CharBlock | Single-line text input | Names, titles |
| RichTextBlock | WYSIWYG text editor | Body text, descriptions |
| ImageBlock | Image selector/uploader | Photos, illustrations |
| EmbedBlock | External content embedder | YouTube videos, tweets |
| PageChooserBlock | Link to other pages | Related content, navigation |

## Worked Example
Building a portfolio page with StreamField:
1. Create `base/blocks.py` with reusable custom blocks
2. Define `CaptionedImageBlock` (StructBlock) for images with captions
3. Define `HeadingBlock` (StructBlock) for sized headings
4. Create `BaseStreamBlock` (StreamBlock) with common blocks
5. Create `portfolio/blocks.py` inheriting from BaseStreamBlock
6. Add CardBlock and FeaturedPostsBlock to portfolio blocks
7. Create templates for each custom block
8. Use blocks in PortfolioPage model with StreamField
9. Create portfolio_page.html template rendering {{ page.body }}

## Key Takeaways
1. StreamField balances developer need for structured data with editor need for flexibility
2. Custom blocks should be defined in a blocks.py file for reusability
3. Each custom block type needs a template for rendering
4. Use `parent_page_types` to control where pages can be created in the tree
5. Search functionality requires adding fields to search_fields and running update_index

## Connects To
- **Ch 3**: Page models and tree structure concepts are applied when setting parent_page_types
- **Ch 4**: StreamField blocks explored here are covered in depth in the StreamField chapter
- **Ch 5**: Image handling in templates uses the image tag demonstrated in the images chapter