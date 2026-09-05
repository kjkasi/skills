# Chapter 5: Images

## Core Idea
Wagtail provides powerful image handling with automatic resizing, format conversion, focal point cropping, and responsive image support through template tags and rendition system.

## Frameworks Introduced
- **Image Tag**: Template tag for inserting resized images with attributes
  - When to use: Displaying images in any template
  - How: `{% image [image] [resize-rule] [attributes] %}`
- **Picture Tag**: Template tag for multiple format support with fallback
  - When to use: Serving modern formats (AVIF, WebP) with JPEG fallback
  - How: `{% picture [image] format-{avif,webp,jpeg} [resize-rule] %}`
- **Srcset Image**: Template tag for responsive images with srcset attribute
  - When to use: Serving different image sizes based on viewport
  - How: `{% srcset_image [image] width-{400,800} sizes="80vw" %}`

## Key Concepts
- **Rendition**: Resized/cropped version of an image generated on-demand
- **Focal Point**: Subject area of image used for intelligent cropping
- **Resize Rule**: Specification for how to transform image (width-400, fill-200x200, etc.)
- **Format Conversion**: Automatic conversion between image formats (GIF→PNG, etc.)
- **Quality Settings**: Configurable compression levels for JPEG, WebP, AVIF
- **SVG Support**: Vector images with special handling for non-rasterizable operations
- **Image Formats**: Predefined styles for rich text embedding (full-width, left/right-aligned)

## Mental Models
- Use `image` tag for simple single-format images
- Use `picture` tag when you need multiple format support (AVIF/WebP/JPEG)
- Use `srcset_image` for responsive images that adapt to viewport size
- Think of focal points as "subject markers" for intelligent cropping

## Anti-patterns
- **Upscaling small images**: Wagtail doesn't support upscaling; images max out at native dimensions
- **Ignoring format conversion**: GIF→PNG conversion loses animation; use preserve-svg for SVGs
- **Not setting quality**: Default settings may be too high/low for specific use cases
- **Forgetting alt text**: Always provide meaningful alt text for accessibility

## Code Examples
```html+django
<!-- Basic image with resize -->
{% image page.photo width-400 %}

<!-- Image with fill crop and focal point awareness -->
{% image page.photo fill-200x200 %}

<!-- Image with closer crop to focal point -->
{% image page.photo fill-200x200-c100 %}

<!-- Image with custom attributes -->
{% image page.photo width-400 class="hero-image" alt="Custom alt text" %}
```
- **What it demonstrates**: Basic image tag usage with different resize rules

```html+django
<!-- Multiple format support -->
{% picture page.photo format-{avif,webp,jpeg} width-400 %}

<!-- Responsive images with srcset -->
{% srcset_image page.photo width-{400,800} sizes="(max-width: 600px) 400px, 80vw" %}

<!-- Picture with multiple formats and sizes -->
{% picture page.photo format-{avif,webp,jpeg} width-{400,800} sizes="80vw" %}
```
- **What it demonstrates**: Modern image delivery with format and size optimization

```html+django
<!-- Accessing rendition properties -->
{% image page.photo width-400 as tmp_photo %}
<img src="{{ tmp_photo.url }}" 
     width="{{ tmp_photo.width }}" 
     height="{{ tmp_photo.height }}" 
     alt="{{ tmp_photo.alt }}" 
     class="my-custom-class" />

<!-- Using attrs shorthand -->
<img {{ tmp_photo.attrs }} class="my-custom-class" />
```
- **What it demonstrates**: Advanced image handling with individual property access

## Reference Tables
| Resize Rule | Dimensions | Behavior |
|-------------|------------|----------|
| `max-WxH` | Two dimensions | Fit within dimensions, maintain ratio |
| `min-WxH` | Two dimensions | Cover dimensions, may be larger |
| `width-W` | One dimension | Set width, maintain ratio |
| `height-H` | One dimension | Set height, maintain ratio |
| `scale-%` | Percentage | Resize to percentage |
| `fill-WxH` | Two dimensions | Crop to exact dimensions |
| `fill-WxH-c%` | Two dimensions + closeness | Crop closer to focal point |
| `original` | None | No resizing |

| Format | Quality Default | Use Case |
|--------|----------------|----------|
| JPEG | 76 | General web content |
| WebP | 80 | Modern browsers |
| AVIF | 61 | Cutting-edge compression |
| PNG | N/A | Transparency, lossless |
| GIF | N/A | Animation (note: loses animation on conversion) |

## Worked Example
Creating responsive hero image:
```html+django
{% load wagtailimages_tags %}

<!-- Desktop: 800px, Mobile: 400px -->
{% srcset_image page.hero_image width-{400,800} sizes="(max-width: 768px) 100vw, 80vw" as hero %}

<img src="{{ hero.renditions.0.url }}"
     srcset="{{ hero.renditions.0.url }} 400w, {{ hero.renditions.1.url }} 800w"
     sizes="(max-width: 768px) 100vw, 80vw"
     alt="{{ page.hero_image.alt }}"
     width="{{ hero.renditions.0.width }}"
     height="{{ hero.renditions.0.height }}"
     loading="lazy" />
```

Setting up lazy loading globally:
```python
# apps.py
from wagtail.images.apps import WagtailImagesAppConfig

class CustomImagesAppConfig(WagtailImagesAppConfig):
    default_attrs = {"decoding": "async", "loading": "lazy"}

# settings.py
INSTALLED_APPS = [
    # ...
    "myapplication.apps.CustomImagesAppConfig",
    # "wagtail.images",
]
```

## Key Takeaways
1. Always specify both image and resize rule in template tag
2. Use `fill` with focal points for intelligent cropping of subject-focused images
3. Use `picture` tag for modern format support with browser fallbacks
4. Use `srcset_image` for responsive images that adapt to viewport
5. Configure quality settings globally or per-tag based on use case
6. SVG images require special handling; use `preserve-svg` when mixing with raster operations

## Connects To
- **Ch 3**: Image fields in page models use ForeignKey to wagtailimages.Image
- **Ch 4**: ImageBlock in StreamField provides image selection in flexible content
- **Ch 2**: Tutorial demonstrates image usage in homepage and portfolio pages