# Cheatsheet

## Quick Decision Rules

| Situation | Action | Why |
|-----------|--------|-----|
| Content is freeform (blog, news) | Use StreamField, not RichTextField | JSON storage preserves structure; rich text is just HTML |
| Need repeated related objects | Use InlinePanel + Orderable | Versioned with parent page; reordable in admin |
| Reusing inline across pages | Abstract model + ParentalKey on base Page | Avoids duplication; still editable only where InlinePanel defined |
| Need page only under specific parent | Set `parent_page_types` | Enforces site structure; empty list hides from editor |
| Prevent page creation in editor | Set `parent_page_types = []` | Doesn't block programmatic creation |
| Need field visible only to some users | FieldPanel `permission="codename"` | Or `permission="superuser"` for superusers only |
| Need columns of fields | FieldRowPanel with `col*` classes | Reduces snow-blindness; groups related fields |
| Collapsible section | MultiFieldPanel with `classname="collapsed"` | Saves vertical space for advanced options |
| Multiple chooser selection | MultipleChooserPanel with `chooser_field_name` | Opens multi-select instead of individual add buttons |
| Fields hidden from admin | Set field `editable=False` or omit from panels | Omitted fields stay hidden from form |
| Want avif/webp fallback | Use `{% picture %}` tag | Browser picks best supported format |
| Responsive images | `{% srcset_image %}` with brace-expansion + `sizes` | Browser selects appropriate resolution |
| Cropped thumbnail | `{% image fill-WxH %}` | Crops to exact dimensions; uses focal point |
| Tight crop to focal point | `fill-WxH-c100` | Crops as close as possible to focal point |
| Image URL for meta tags | `{% image_url %}` instead of `{% image %}` | Offloads rendition creation to separate request |
| Page with many images | Prefetch renditions on queryset | Single extra query vs N+1 |
| Cache page fragments | Use `{% wagtailcache %}` or `{% wagtailpagecache %}` | Safe for preview; Django `{% cache %}` leaks drafts |
| Production search at scale | Elasticsearch/OpenSearch backend | Database backend fine for dev/small sites |
| High traffic site | Redis cache + frontend CDN with purge hooks | Wagtail supports Cloudflare/CloudFront invalidation |
| Multi-site shared search | Set `INDEX_PREFIX` on Elasticsearch backend | Prevents index collisions |
| Want search auto-update disabled | Set `AUTO_UPDATE: False` + run `update_index` | Useful for external ES with bulk indexing |
| Images stored on S3 | Set `WAGTAILIMAGES_RENDITION_STORAGE` | Renditions on CDN, not Django server |
| SVG with format filter | Add `preserve-svg` to image tag | Prevents rasterization error on SVGs |
| Need page cache key | Use `page.cache_key` | Changes when page state changes; override `get_cache_key_components` for custom fields |
| Private collection for docs | Set up private collection + `WAGTAILDOCS_SERVE_METHOD` | `'redirect'` for S3; `'serve_view'` for local |
| Prevent bulk delete accidents | Keep `WAGTAILADMIN_UNSAFE_PAGE_DELETION_LIMIT` at 10 | Forces site name confirmation |
| Want custom page base | Set `WAGTAIL_PAGE_MODEL` | All pages inherit from your model |
| Want autosave disabled | Set `WAGTAIL_AUTOSAVE_INTERVAL = 0` | Prevents unexpected saves |
| Want preview disabled | Set `preview_modes = []` on model | Removes preview panel entirely |

## Decision Trees

### Page Model Structure
- Fixed layout with known fields → Django model fields + FieldPanel
- Freeform mixed content → StreamField with block types
  - Need headings + paragraphs + images → StreamField with CharBlock + RichTextBlock + ImageBlock
  - Need structured repeatable sections → StreamField with StructBlock
  - Need repeating list of items → StreamField with ListBlock
  - Need mixed content in sub-section → StreamField with StreamBlock

### Block Type Selection
- Single field, no sub-structure → Built-in block (CharBlock, RichTextBlock, etc.)
- Group of related fields → StructBlock (subclass if reusable)
- Repeat same type → ListBlock
- Mix of types in sequence → StreamBlock

### Image Resize Rule
- Need exact dimensions with crop → `fill-WxH`
- Need to fit within bounds → `max-WxH`
- Need to cover at least bounds → `min-WxH`
- Need specific width only → `width-W`
- Need specific height only → `height-H`
- Need percentage → `scale-N`

### Search Backend Choice
- Dev/small site → Database backend (`wagtail.search.backends.database`)
- Production with ES → Elasticsearch 7/8/9 backend
- AWS-hosted → OpenSearch with `requests-aws4auth`
- Need full-text + faceted → Elasticsearch (fast, powerful)
- Need simple text search → Database (zero config)

### Permission Model
- Page editing → Tree-based: add/edit/publish propagate down
- Image/document access → Collection-based: root collection = global
- Hide fields from role → `FieldPanel(permission="codename")`
- Restrict panel group → `MultiFieldPanel(permission="codename")`
- Superuser-only field → `FieldPanel(permission="superuser")`

## Trade-off Matrices

### RichTextField vs StreamField

| Dimension | RichTextField | StreamField |
|-----------|--------------|-------------|
| Editor freedom | WYSIWYG only | Mixed structured + freeform |
| Data storage | HTML string | JSON with block structure |
| Migration effort | Low | High (data migrations needed) |
| Template rendering | Direct output | `{% include_block %}` required |
| Search indexing | Single SearchField | Per-block control |
| Performance | Simpler queries | More complex but indexed |

### Search Backends

| Dimension | Database | Elasticsearch | OpenSearch |
|-----------|----------|---------------|------------|
| Setup complexity | Zero | Medium | Medium |
| Performance at scale | Limited | Fast | Fast |
| Full-text features | Basic FTS | Advanced | Advanced |
| Faceted search | No | Yes | Yes |
| Hosting requirement | Existing DB | Separate service | Separate service |
| Auto-update | Automatic | Automatic | Automatic |
| Cost | Free | Free/self-host or paid | Free/self-host or paid |

### Image Serving Methods

| Dimension | Direct | Redirect | Serve View |
|-----------|--------|----------|------------|
| Permission check | Bypassed | Check then redirect | Check then serve |
| Performance | Fastest | Fast | Slowest |
| S3 compatible | Yes | Yes | No |
| Security risk | High | Medium | Low |
| Use case | Static sites | S3/CDN | Local storage |

## Thresholds & Defaults

### Limits
- `WAGTAILIMAGES_MAX_IMAGE_PIXELS`: 128 megapixels (128,000,000)
- `WAGTAILIMAGES_MAX_UPLOAD_SIZE`: 10MB default
- `WAGTAILDOCS_MAX_UPLOAD_SIZE`: No limit default
- `WAGTAILADMIN_UNSAFE_PAGE_DELETION_LIMIT`: 10 pages
- `WAGTAILSEARCH_HITS_MAX_AGE`: 7 days
- `WAGTAILAPI_LIMIT_MAX`: 20 results
- `WAGTAIL_AUTOSAVE_INTERVAL`: 500ms
- `WAGTAIL_AUTO_UPDATE_PREVIEW_INTERVAL`: 500ms
- `WAGTAIL_EDITING_SESSION_PING_INTERVAL`: 10000ms (10s)
- `WAGTAILIMAGES_INDEX_PAGE_SIZE`: 30
- `WAGTAILIMAGES_CHOOSER_PAGE_SIZE`: 12
- `WAGTAIL_TAG_LIMIT`: None (unlimited)

### Image Quality Defaults
| Format | Default Quality | Thumbnail (64px) | Large Thumb (256px) | Full Photo |
|--------|----------------|-------------------|---------------------|------------|
| JPEG | 76 | 55 | 65 | 85 |
| WebP | 80 | 57 | 70 | 87 |
| AVIF | 61 | 49 | 54 | 73 |

### Elasticsearch N-gram Settings
- `min_gram`: 3 (ngram), 2 (edgengram)
- `max_gram`: 15
- `max_ngram_diff`: 12

### Panel CSS Classes
- `title`: Bigger font for title fields
- `collapsed`: Panel starts collapsed
- `col1`–`col12`: FieldRowPanel column width (out of 12)

## Tells & Smells

### Red Flags
- `Meta.ordering` on Page model → Won't work; use `.order_by()` on QuerySet
- `{{ block }}` instead of `{% include_block block %}` → Loses request context; use include_block
- Rich text body with images/quotes/video → Should be StreamField instead
- Permission check failing for images → Check collection hierarchy, not page tree
- Renditions not appearing → Check `WAGTAILIMAGES_EXTENSIONS` allows your format
- SVG with format filter error → Add `preserve-svg` argument
- Preview showing drafts to public → Use `{% wagtailcache %}` not `{% cache %}`
- Slow page with many images → Prefetch renditions; use `{% image_url %}` for meta tags
- Search returning stale results → Run `update_index` if `AUTO_UPDATE: False`
- Field visible to wrong users → Check `permission` kwarg on FieldPanel
- Bulk delete blocked → User needs `bulk delete` permission + underlying add/edit
- Page URL missing request → Use `{% pageurl %}` tag which auto-passes request
- StreamField empty in search → Add field to `search_fields` with `index.SearchField`
- Block not indexed → Set `search_index=True` explicitly if needed

### Quick Checks
- Page template not found? → Check `app_label/model_name.html` naming
- Inline panel empty? → Check `related_name` matches `InlinePanel` first arg
- Image tag not rendering? → Both image object AND resize rule required
- Settings not taking effect? → Check if using `WAGTAILSEARCH_BACKENDS` not `MODELSEARCH_BACKENDS`
- Forms showing in admin? → Check `WAGTAILSNIPPETS_MENU_SHOW_ALL` setting
- Password fields missing? → Check `WAGTAILUSERS_PASSWORD_ENABLED`
- Update notifications wrong level? → Set `WAGTAIL_ENABLE_UPDATE_CHECK = "lts"` for LTS only
