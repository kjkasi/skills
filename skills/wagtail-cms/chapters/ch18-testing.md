# Chapter 18: Testing Wagtail Sites

## Core Idea
Wagtail provides specialized test utilities including `WagtailPageTestCase` with page-specific assertions, form data helpers for StreamField/RichText, and fixture creation patterns for the page tree hierarchy. Tests must account for Wagtail's tree structure and revision system.

## Frameworks Introduced
- **WagtailPageTestCase**: Extended Django TestCase with page-specific assertions for creation, rendering, editing, and previewing
  - When to use: Testing any page type behavior, routing, rendering, or admin functionality
  - How: Extend `WagtailPageTestCase`, use `assertCanCreate`, `assertPageIsRenderable`, etc.
- **Form Data Helpers**: Functions for constructing POST data for StreamField and nested forms
  - When to use: Submitting page creation/edit forms via test client
  - How: Import from `wagtail.test.utils.form_data`, use `nested_form_data()`, `streamfield()`, `rich_text()`

## Key Concepts
- **assertPageIsRoutable**: Verifies a page can be routed to without Http404 for a given route path
- **assertPageIsRenderable**: Verifies a page renders without fatal errors, with options for `accept_404` and `accept_redirect`
- **assertPageIsEditable**: Tests the page edit view works (GET + POST cycle) with auto-extracted or provided post data
- **assertPageIsPreviewable**: Tests preview mode loading including custom preview modes
- **assertCanCreateAt**: Asserts a child page type can be created under a parent type
- **assertCanCreate**: Full creation test with POST data, including StreamField content
- **assertAllowedParentPageTypes / assertAllowedSubpageTypes**: Tests page hierarchy constraints
- **nested_form_data**: Wraps form data for inline panels and StreamField structures
- **RichText**: Wrapper for RichTextBlock content in test data
- **Page Tree Setup**: `Page.get_first_root_node()` + `Site.objects.create()` + `parent.add_child(instance=page)`

## Mental Models
1. **Page Creation Chain**: Pages can't use `objects.create()` — they must be added via `parent.add_child(instance=child)`, starting from the root node
2. **Fixture Dual Records**: Custom page models need both `wagtailcore.page` and `app.model` entries in fixtures for proper content type resolution
3. **Test Data Extraction**: `assertPageIsEditable` automatically extracts form data from GET response HTML when `post_data` isn't provided
4. **Preview Mode as Request Attribute**: Preview segments are stored on `request.personalization_preview_segment` for block-level `get_context` access

## Anti-patterns
- **Using `MyPage.objects.create()`**: Pages require tree operations — always use `parent.add_child(instance=page)`
- **Missing Site setup in tests**: Every test needs a `Site` object with `hostname="testserver"` and a `root_page`
- **Using `--keepdb` with TransactionTestCase tests**: Django's known issue causes initial data loss — exclude with `--exclude-tag=transaction`
- **Hardcoding fixture primary keys**: Use `--natural-foreign` with `dumpdata` to use `["app", "model"]` instead of PKs

## Code Examples
```python
from wagtail.test.utils import WagtailPageTestCase
from wagtail.models import Page, Site
from home.models import HomePage, MyPage

class MyPageTest(WagtailPageTestCase):
    @classmethod
    def setUpTestData(cls):
        root = Page.get_first_root_node()
        Site.objects.create(
            hostname="testserver",
            root_page=root,
            is_default_site=True,
            site_name="testserver",
        )
        home = HomePage(title="Home")
        root.add_child(instance=home)
        cls.page = MyPage(title="My Page", slug="mypage")
        home.add_child(instance=cls.page)

    def test_get(self):
        response = self.client.get(self.page.url)
        self.assertEqual(response.status_code, 200)
```
- **What it demonstrates**: Basic Wagtail page test with proper tree and site setup

```python
from wagtail.test.utils.form_data import nested_form_data, streamfield

def test_can_create_content_page(self):
    root_page = HomePage.objects.get(pk=2)
    self.assertCanCreate(
        root_page,
        ContentPage,
        nested_form_data({
            "title": "About us",
            "body": streamfield([("text", "Lorem ipsum dolor sit amet")]),
        }),
    )
```
- **What it demonstrates**: Testing page creation with StreamField content via form data helpers

```python
from wagtail.rich_text import RichText

class MyPageTest(WagtailPageTestCase):
    @classmethod
    def setUpTestData(cls):
        # ... create page
        cls.page.body.extend([
            ("heading", "Just a CharField Heading"),
            ("paragraph", RichText("<p>First paragraph</p>")),
        ])
        cls.page.save()

    def test_page_content(self):
        response = self.client.get(self.page.url)
        self.assertContains(response, "Just a CharField Heading")
        self.assertContains(response, "<p>First paragraph</p>")
```
- **What it demonstrates**: Testing StreamField content with RichText wrapper

```python
def test_editability(self):
    self.assertPageIsEditable(self.page)

def test_editability_on_post(self):
    self.assertPageIsEditable(
        self.page,
        post_data={
            "title": "Fabulous events",
            "slug": "events",
            "show_featured": True,
            "action-publish": "",
        },
    )
```
- **What it demonstrates**: Testing page edit view with custom POST data

```python
def test_default_route(self):
    self.assertPageIsRoutable(self.page)

def test_year_archive_route(self):
    self.assertPageIsRoutable(self.page, "archive/year/1984/")
```
- **What it demonstrating**: Testing custom page routing logic

## Reference Tables

| Assertion | Purpose | Key Parameters |
|-----------|---------|----------------|
| `assertPageIsRoutable` | Page routing works | `route_path`, `msg` |
| `assertPageIsRenderable` | Page renders without error | `route_path`, `accept_404`, `accept_redirect`, `user` |
| `assertPageIsEditable` | Edit view GET+POST cycle | `post_data`, `user` |
| `assertPageIsPreviewable` | Preview mode loads | `mode`, `post_data`, `user` |
| `assertCanCreateAt` | Child type allowed under parent | `parent_model`, `child_model` |
| `assertCanCreate` | Full creation with POST data | `parent`, `child_model`, `data`, `publish` |
| `assertAllowedParentPageTypes` | Parent type constraints | `child_model`, `parent_models` |
| `assertAllowedSubpageTypes` | Child type constraints | `parent_model`, `child_models` |

| Form Data Helper | Purpose |
|-----------------|---------|
| `nested_form_data()` | Wraps data for InlinePanel and StreamField |
| `streamfield()` | Creates StreamField block list |
| `rich_text()` | Wraps content in `RichText()` for RichTextBlock |
| `inline_formset()` | Creates inline formset data |

## Worked Example
Testing a blog with tag filtering:

```python
from wagtail.test.utils import WagtailPageTestCase
from wagtail.models import Page, Site
from wagtail.test.utils.form_data import nested_form_data

class BlogPageTest(WagtailPageTestCase):
    @classmethod
    def setUpTestData(cls):
        root = Page.get_first_root_node()
        Site.objects.create(hostname="testserver", root_page=root, is_default_site=True)
        home = HomePage(title="Home")
        root.add_child(instance=home)
        cls.blog_index = BlogIndexPage(title="Blog")
        home.add_child(instance=cls.blog_index)
        cls.blog_page = BlogPage(title="First Post", slug="first-post")
        cls.blog_index.add_child(instance=cls.blog_page)

    def test_page_is_renderable(self):
        self.assertPageIsRenderable(self.blog_page)

    def test_page_is_editable(self):
        self.assertPageIsEditable(self.blog_page)

    def test_can_create_blog_page(self):
        self.assertCanCreate(
            self.blog_index,
            BlogPage,
            nested_form_data({"title": "New Post", "body": streamfield([("text", "Hello")])}),
        )

    def test_allowed_parents(self):
        self.assertAllowedParentPageTypes(BlogPage, {BlogIndexPage})
```

## Key Takeaways
1. Pages must be created via `parent.add_child(instance=page)` — not `objects.create()` — and require a `Site` object in tests
2. `WagtailPageTestCase` provides 8+ page-specific assertions covering routing, rendering, editing, previewing, and hierarchy validation
3. Form data helpers (`nested_form_data`, `streamfield`, `rich_text`) are essential for testing StreamField and InlinePanel submissions
4. Use `--parallel --keepdb --exclude-tag=transaction` for fast test runs, excluding TransactionTestCase tests
5. Custom page models need dual fixture entries (`wagtailcore.page` + `app.model`) with `--natural-foreign` for consistent content types

## Connects To
- **Ch 5**: Page tree structure (understanding root nodes and `add_child`)
- **Ch 10**: Page models (testing page type creation and hierarchy)
- **Ch 16**: Customization (testing custom edit handlers and forms)
- **Ch 19**: Contributing (running Wagtail's own test suite)
