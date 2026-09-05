# Chapter 16: Deep Admin Customization

## Core Idea
Wagtail's admin interface is fully customizable through Django's template inheritance, custom page models, custom user models, and viewset overrides. This allows developers to tailor the editing experience to match exact project requirements while maintaining upgrade compatibility.

## Frameworks Introduced
- **Custom Base Page Models**: Define application-specific fields available across all page types without multi-level inheritance overhead
  - When to use: When all page types need shared fields (category, review_date) that are queryable across types
  - How: Inherit from `AbstractPage`, set `WAGTAIL_PAGE_MODEL`, create bootstrap migrations, then extend `BasePage` in child models
- **Admin Template Overriding**: Use Django template inheritance to replace branding, login forms, and UI elements
  - When to use: Customizing logos, colors, fonts, login forms, or adding custom admin UI elements
  - How: Create `templates/wagtailadmin/` in an app registered before `wagtail.admin`, override template blocks
- **Custom User Models**: Extend Django's auth system with Wagtail-compatible user forms and viewsets
  - When to use: Adding profile fields (country, status) that appear in admin user management
  - How: Extend `AbstractUser`, create custom forms, extend templates, configure `UserViewSet` subclass
- **Tabbed Interface Customization**: Override default Content/Promote tabs with custom panel layouts
  - When to use: Adding sidebar content panels, permission-restricted tabs, or reorganizing the edit interface
  - How: Set `edit_handler = TabbedInterface([ObjectList(...), ...])` on the page model

## Key Concepts
- **AbstractPage**: Base class providing standard page fields (minus `show_in_menus`, `seo_title`, `search_description`) for custom base page models
- **ShowInMenusMixin**: Mixin to add `show_in_menus` support back when using `AbstractPage`
- **DefaultBasePageMixin**: Mixin inheriting from both `AbstractPage` and `ShowInMenusMixin` for all default Page fields
- **ClusterTaggableManager**: In-memory tag management for previews and revisions via django-modelcluster
- **SlugInput**: Locale-aware widget for slug generation with custom formatters and transliteration
- **NoFutureDateValidator**: Built-in validator preventing future dates on model fields
- **WagtailAdminPageForm**: Base form class for custom page form logic including deferred validation
- **CopyForm**: Customizable form class for page copy operations with auto-increment slug support

## Mental Models
1. **Template Block Override Hierarchy**: Admin templates use Django's inheritance chain — override specific blocks in `wagtailadmin/base.html` while keeping the rest intact
2. **Base Page Model Bootstrap**: Custom base page models require a 3-migration sequence: initial (without wagtailcore deps), bootstrap (locale/translation_key), finalize (remaining fields)
3. **Form Class Chain**: Page forms generate subclasses automatically — custom `base_form_class` provides the base, Wagtail adds panel-defined fields on top
4. **Viewset Registration**: Custom admin views are registered via hooks (`register_admin_viewset`) and subclass existing viewsets for incremental customization

## Anti-patterns
- **Registering custom app after wagtail.admin**: The custom app must be in `INSTALLED_APPS` before `wagtail.admin` for template overrides to work
- **Using try/except for Django version checks**: Always use explicit `django.VERSION >= (x, y)` comparisons instead of try/except for Django compatibility
- **Ignoring migration dependency ordering**: Custom base page model migrations must run before Wagtail's — remove `wagtailcore` dependency and delete bootstrap fields from initial migration
- **Not using `@register_form_field_override` for global widget changes**: Prefer `wagtail_hooks.py` registration over per-model overrides for site-wide SlugInput changes

## Code Examples
```python
# Custom base page model
from wagtail.models import AbstractPage

class BasePage(AbstractPage):
    category = models.CharField(max_length=100, blank=True)
    review_date = models.DateField(blank=True, null=True)
    promote_panels = AbstractPage.promote_panels + ["category", "review_date"]

# Settings
WAGTAIL_PAGE_MODEL = "basepage.BasePage"
```
- **What it demonstrates**: Defining a custom base page model with shared fields available to all page types

```python
# Custom tabbed interface with permissions
from wagtail.admin.panels import TabbedInterface, TitleFieldPanel, ObjectList

class FundingPage(Page):
    shared_panels = [TitleFieldPanel("title"), FieldPanel("body")]
    private_panels = [FieldPanel("approval")]

    edit_handler = TabbedInterface([
        ObjectList(shared_panels, heading="Details"),
        ObjectList(private_panels, heading="Admin only", permission="superuser"),
        ObjectList(Page.promote_panels, heading="Promote"),
        ObjectList(Page.settings_panels, heading="Settings"),
    ])
```
- **What it demonstrates**: Custom tabbed editing interface with permission-restricted panels

```html+django
{% extends "wagtailadmin/base.html" %}
{% load static %}
{% block branding_logo %}
    <img src="{% static 'images/custom-logo.svg' %}" alt="Custom Project" width="80" />
{% endblock %}
```
- **What it demonstrates**: Replacing the admin logo via template block override

```python
# Custom user model with forms
class User(AbstractUser):
    country = models.CharField(max_length=255)
    status = models.ForeignKey(MembershipStatus, on_delete=models.SET_NULL, null=True)

class CustomUserEditForm(UserEditForm):
    status = forms.ModelChoiceField(queryset=MembershipStatus.objects, required=True)
    class Meta(UserEditForm.Meta):
        fields = UserEditForm.Meta.fields | {"country", "status"}
```
- **What it demonstrates**: Extending the Wagtail user model with custom fields and edit forms

```python
# Custom page listing viewset
class BlogPageViewSet(PageViewSet):
    model = BlogPage
    parent_models = [BlogIndexPage]
    columns = PageViewSet.columns + [
        Column("blog_category", label="Category", sort_key="blog_category"),
    ]
    filterset_class = BlogPageFilterSet
```
- **What it demonstrates**: Customizing the page explorer listing with additional columns and filters

## Reference Tables

| Admin Template Block | Override File | Purpose |
|---------------------|---------------|---------|
| `branding_logo` | `wagtailadmin/base.html` | Replace admin logo |
| `branding_favicon` | `wagtailadmin/admin_base.html` | Replace favicon |
| `branding_title` | `wagtailadmin/admin_base.html` | Change title prefix |
| `branding_login` | `wagtailadmin/login.html` | Custom login message |
| `branding_welcome` | `wagtailadmin/home.html` | Dashboard welcome message |
| `above_login` / `below_login` | `wagtailadmin/login.html` | Add content around login form |
| `fields` | `wagtailadmin/login.html` | Add extra login form fields |

| Rich Text Feature IDs | Description |
|----------------------|-------------|
| `h2`, `h3`, `h4` | Heading elements |
| `bold`, `italic` | Text formatting |
| `ol`, `ul` | Lists |
| `link` | Page/external/email links |
| `document-link` | Document links |
| `image` | Embedded images |
| `embed` | Embedded media |
| `h1`, `h5`, `h6` | Extra headings (not default) |
| `code`, `blockquote` | Code/quote blocks (not default) |

## Worked Example
Setting up a custom user model with admin integration:

1. Create `myapp/models.py` with `User(AbstractUser)` adding `country` and `status` fields
2. Set `AUTH_USER_MODEL = "myapp.User"` and add app before `wagtail.users` in `INSTALLED_APPS`
3. Create `CustomUserEditForm` and `CustomUserCreationForm` extending Wagtail's forms
4. Create templates `wagtailusers/users/create.html` and `edit.html` extending Wagtail's templates, adding fields in `extra_fields` block
5. Create `UserViewSet` subclass calling `get_form_class()` to return custom forms
6. Create `CustomUsersAppConfig` subclass setting `user_viewset = "myapp.viewsets.UserViewSet"`
7. Replace `wagtail.users` with `myproject.apps.CustomUsersAppConfig` in `INSTALLED_APPS`

## Key Takeaways
1. Custom base page models require a specific 3-migration bootstrap sequence before Wagtail's own migrations run
2. Admin template overrides use Django's template inheritance — create files in `templates/wagtailadmin/` in apps registered before `wagtail.admin`
3. The `edit_handler` attribute with `TabbedInterface` gives full control over page/snippet editing layout and permissions
4. Custom user models require forms, template overrides, a `UserViewSet` subclass, and a custom `AppConfig` — four coordinated pieces
5. Page listing customization uses `PageViewSet` and `PageListingViewSet` subclasses registered via the `register_admin_viewset` hook

## Connects To
- **Ch 10**: Page models and StreamField (foundation for understanding page model inheritance)
- **Ch 12**: Snippets (custom page listings apply to snippets via viewsets)
- **Ch 15**: Forms and form builder (custom form classes integrate with Wagtail's form system)
- **Ch 17**: Performance (custom base page models improve query performance across page types)
