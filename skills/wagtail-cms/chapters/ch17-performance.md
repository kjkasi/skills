# Chapter 17: Performance Optimization

## Core Idea
Wagtail provides multiple performance optimization layers including Redis caching, deferred image serving, template fragment caching, and frontend cache purging integration. Most optimizations are configuration-based with minimal code changes required.

## Frameworks Introduced
- **Redis Cache Backend**: High-performance persistent caching using django-redis
  - When to use: Production deployments handling significant traffic
  - How: Install Redis, add `django-redis` to requirements, configure `CACHES` setting with Redis backend
- **Template Fragment Caching**: Cache specific portions of templates using Wagtail-safe tags
  - When to use: Repeating expensive template calculations across requests
  - How: Use `{% wagtailcache %}` or `{% wagtailpagecache %}` tags instead of Django's `{% cache %}`
- **Image Serve View**: Offload image rendition creation to a separate request
  - when to use: Pages with many images or when image processing errors shouldn't break page loads
  - How: Use `{% image_url %}` tag or `generate_image_url` instead of `{% image %}` for meta tags

## Key Concepts
- **Renditions Cache Backend**: Separate cache backend for image renditions (default: Memcached on port 11211)
- **Page Cache Key**: Unique value per page state via `page.cache_key` for cache invalidation
- **Cache Key Components**: Override `get_cache_key_components()` to include custom fields in cache key computation
- **Image Serve View**: Deferred image processing that creates renditions on first request rather than during page load
- **Prefetch Image Renditions**: Single-query prefetching for lists of objects with images
- **Frontend Cache Purging**: Automatic CDN cache invalidation when pages change
- **Lazy Loading**: `loading='lazy'` and `decoding='async'` attributes for deferred image loading

## Mental Models
1. **Cache Layer Separation**: Use separate cache backends for general data vs. image renditions to optimize eviction policies
2. **Deferred Processing**: Image serve view moves rendition creation from page request to image request, preventing page failures from image processing errors
3. **Cache Key as Page State**: `page.cache_key` changes whenever page content changes — use it as part of custom cache keys for automatic invalidation
4. **Request Context Propagation**: Passing `request` or `current_site` to `page.get_url()` enables cache reuse across navigation and content links

## Anti-patterns
- **Using Django's `{% cache %}` tag directly**: Can leak preview content to end users — always use `{% wagtailcache %}` or `{% wagtailpagecache %}` instead
- **Not prefetching image renditions in list views**: Each item's rendition triggers a separate query — use `prefetch_image_renditions()` for significant performance gains
- **Hardcoding page URLs in templates**: Always use `{% pageurl %}` or `{% fullpageurl %}` which pass `request` automatically, avoiding redundant site lookups
- **Ignoring `HOSTNAMES` for multi-frontend setups**: Without it, purge requests are sent to all backends unnecessarily

## Code Examples
```python
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/dbname",
        "OPTIONS": {"CLIENT_CLASS": "django_redis.client.DefaultClient"},
    }
}
```
- **What it demonstrates**: Configuring Redis as the primary cache backend

```html+django
<meta property="og:image" content="{% image_url page.hero_image width-600 %}" />
```
- **What it demonstrates**: Using image serve view for deferred rendition creation in meta tags

```python
from django.core.cache import cache

result = page.expensive_operation()
cache.set("expensive_result_" + page.cache_key, result, 3600)
```
- **What it demonstrates**: Using `page.cache_key` for automatic cache invalidation when page changes

```python
def get_cache_key_components(self):
    components = super().get_cache_key_components()
    components.append(self.external_slug)
    return components
```
- **What it demonstrates**: Customizing cache key to include additional model fields

```html+django
{% wagtailcache 3600 "my-cache-key" %}
    {# expensive template content #}
{% endwagtailcache %}
```
- **What it demonstrates**: Using Wagtail-safe template fragment caching

## Reference Tables

| Optimization | Impact | Setup Effort |
|-------------|--------|--------------|
| Redis cache backend | High | Low (config only) |
| Image serve view (`{% image_url %}`) | Medium-High | Low (template change) |
| Prefetch image renditions | Medium | Low (queryset change) |
| Template fragment caching | Medium | Low (template tags) |
| Page cache key | Medium | Low (config + code) |
| Frontend cache purging | High for CDN sites | Medium (CDN setup) |
| Elasticsearch backend | High for search | Medium (install + config) |

## Worked Example
Optimizing a blog index page with many posts and images:

1. Configure Redis cache backend in settings
2. Add renditions cache backend pointing to Memcached
3. In `BlogIndexPage.get_context()`, use `prefetch_image_renditions()` on the queryset
4. In templates, use `{% wagtailcache %}` around the post list
5. For social sharing, use `{% image_url %}` instead of `{% image %}` for og:image meta tags
6. In navigation templates, always use `{% pageurl %}` which auto-passes request context
7. Configure frontend cache purging with `HOSTNAMES` list for the specific CDN backend

## Key Takeaways
1. Redis is the recommended cache backend for production Wagtail installations
2. Always use `{% wagtailcache %}` or `{% wagtailpagecache %}` instead of Django's `{% cache %}` to avoid leaking preview content
3. Use `{% image_url %}` for image meta tags to defer rendition creation and prevent page errors from image processing failures
4. `page.cache_key` provides automatic invalidation — override `get_cache_key_components()` to include custom fields
5. Prefetching image renditions with `prefetch_image_renditions()` eliminates N+1 queries in list views

## Connects To
- **Ch 6**: Images (renditions and image serving)
- **Ch 9**: Templates (template fragment caching integration)
- **Ch 16**: Customization (cache key components in custom page models)
- **Ch 18**: Testing (performance testing considerations)
