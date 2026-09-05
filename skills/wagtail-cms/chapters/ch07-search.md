# Chapter 7: Search

## Core Idea
Wagtail provides a pluggable search system with support for database full-text search, Elasticsearch, and OpenSearch backends. Content is made searchable by declaring `search_fields` on models, and queries are performed via QuerySet methods or the search backend API.

## Frameworks Introduced
- **Search Indexing**: Declare which fields on a model are searchable (`SearchField`), autocomplete-enabled (`AutocompleteField`), or filterable (`FilterField`) via `search_fields`.
  - When to use: Any model you want users to find via search — Pages, Images, Documents, or custom models.
  - How: Add `search_fields` to the model class; use `index.SearchField` for full-text, `index.AutocompleteField` for partial matching, `index.FilterField` for faceting.
- **Search Backend Configuration**: Configure which search engine to use via `WAGTAILSEARCH_BACKENDS`.
  - When to use: Always — even the default database backend needs to be explicitly configured in some setups.
  - How: Set `WAGTAILSEARCH_BACKENDS` in settings with the desired backend class and options.

## Key Concepts
- **`SearchField`**: Indexes a field for full-text search; supports `boost` for ranking and `es_extra` for Elasticsearch mapping overrides.
- **`AutocompleteField`**: Indexes a field for partial-word matching (e.g., "Hel" matches "Hello World").
- **`FilterField`**: Indexes a field for filtering results but not full-text search (e.g., date ranges, categories).
- **`RelatedFields`**: Indexes fields from related models (ForeignKey, ManyToMany, reverse relations).
- **`AUTO_UPDATE`**: Per-backend setting to enable/disable automatic index updates on save/delete.
- **`update_index` management command**: Rebuilds the search index from scratch; run after model changes or bulk imports.
- **Search operators**: `"or"` (match any term, default for Elasticsearch) vs `"and"` (match all terms, default for database).
- **Phrase/Fuzzy/Boost queries**: Advanced query types via `wagtail.search.query` classes.
- **`parse_query_string`**: Utility to parse user input into query objects with colon-syntax filters.

## Mental Models
- Search is decoupled from the database — the index can live in PostgreSQL FTS, Elasticsearch, or OpenSearch, and the same QuerySet API works across all backends.
- Indexing is opt-in for fields: only fields declared in `search_fields` are searchable; everything else is ignored.
- Signal handlers keep the index in sync automatically, but `update_index` is the safety net for rebuilding from scratch.

## Anti-patterns
- **Forgetting to index custom fields**: Extra fields on Page/Image subclasses won't appear in search results unless added to `search_fields`.
- **Relying solely on `AUTO_UPDATE` in production**: Signal handlers can miss edge cases — run `update_index` periodically as a safety net.
- **Using `autocomplete()` for regular search**: Autocomplete returns partial matches that dilute relevance; use `search()` for normal queries.
- **Setting `AUTO_UPDATE = False` without a cron job**: The index will drift from the database; always schedule `update_index`.

## Code Examples
```python
# settings.py — configure search backend
WAGTAILSEARCH_BACKENDS = {
    "default": {
        "BACKEND": "wagtail.search.backends.database",
    }
}
```
- **What it demonstrates**: Setting up the default database search backend.

```python
# settings.py — Elasticsearch backend
WAGTAILSEARCH_BACKENDS = {
    "default": {
        "BACKEND": "wagtail.search.backends.elasticsearch9",
        "URLS": ["https://localhost:9200"],
        "INDEX_PREFIX": "mysite_",
        "TIMEOUT": 5,
    }
}
```
- **What it demonstrates**: Configuring Elasticsearch 9 with index prefixing.

```python
# models.py — indexing extra fields on a Page
from wagtail.search import index

class EventPage(Page):
    description = models.TextField()
    date = models.DateField()

    search_fields = Page.search_fields + [
        index.SearchField("description"),
        index.FilterField("date"),
    ]
```
- **What it demonstrates**: Adding searchable and filterable fields to a Page subclass.

```python
# models.py — indexing related fields
class Book(models.Model, index.Indexed):
    search_fields = [
        index.SearchField("title", boost=10),
        index.AutocompleteField("title"),
        index.FilterField("author"),
        index.RelatedFields("author", [
            index.SearchField("name"),
        ]),
    ]
```
- **What it demonstrates**: Indexing across relationships and boosting title relevance.

```python
# models.py — indexing callables
class EventPage(Page):
    IS_PRIVATE_CHOICES = ((False, "Public"), (True, "Private"))
    is_private = models.BooleanField(choices=IS_PRIVATE_CHOICES)

    search_fields = Page.search_fields + [
        index.SearchField("get_is_private_display"),
        index.FilterField("is_private"),
    ]
```
- **What it demonstrates**: Indexing the human-readable display value of a choices field.

```python
# querying — basic search
>>> EventPage.objects.filter(date__gt=timezone.now()).search("Christmas")

# autocomplete
>>> EventPage.objects.live().autocomplete("Eve")

# search specific fields
>>> EventPage.objects.search("Event", fields=["title"])

# custom model search
>>> from wagtail.search.backends import get_search_backend
>>> s = get_search_backend()
>>> s.search("Great", Book.objects.filter(published_date__year__lt=1900))
```
- **What it demonstrates**: Various search QuerySet patterns.

```python
# advanced queries
from wagtail.search.query import PlainText, Phrase, Boost, Fuzzy

# phrase search
>>> Page.objects.search(Phrase("Hello world"))

# fuzzy matching (Elasticsearch only)
>>> Page.objects.search(Fuzzy("Hallo"))

# boolean composition
>>> Page.objects.search(PlainText("Hello") & (PlainText("world") | PlainText("earth")))

# boost specific queries
>>> Page.objects.search(Boost(Phrase("Hello world"), 10.0) | Phrase("Hello earth"))
```
- **What it demonstrates**: Composing complex search queries.

```python
# parse_query_string — user input with filters
from wagtail.search.utils import parse_query_string

filters, query = parse_query_string(
    'my query "this is a phrase" published:yes', operator='and'
)
# filters = {'published': ['yes']}
# query = And([PlainText('my query'), Phrase('this is a phrase')])
```
- **What it demonstrates**: Parsing user search input with colon-syntax filters.

```python
# faceted search
>>> Page.objects.search("Test").facet("content_type_id")
OrderedDict([('2', 4), ('1', 2)])
```
- **What it demonstrates**: Getting facet counts for taxonomy fields.

```html+django
{# frontend search view template #}
<form action="{% url 'search' %}" method="get">
    <input type="text" name="query" value="{{ search_query }}">
    <input type="submit" value="Search">
</form>
{% for result in search_results %}
    <h4><a href="{% pageurl result %}">{{ result }}</a></h4>
    {{ result.search_description|safe }}
{% endfor %}
```
- **What it demonstrates**: A complete frontend search form and results listing.

## Reference Tables

| Field Type | Purpose | Use Case |
|---|---|---|
| `SearchField` | Full-text search indexing | Text fields users search against |
| `AutocompleteField` | Partial-word matching | Search-as-you-type inputs |
| `FilterField` | Filtering/faceting | Date ranges, categories, booleans |
| `RelatedFields` | Index related model fields | ForeignKey/ManyToMany content |

| Backend | Module | When to Use |
|---|---|---|
| Database (default) | `wagtail.search.backends.database` | Development, small-to-medium sites |
| Elasticsearch 7/8/9 | `wagtail.search.backends.elasticsearch{7,8,9}` | Large sites, advanced search features |
| OpenSearch 2/3 | `wagtail.search.backends.opensearch{2,3}` | AWS-hosted or self-managed OpenSearch |

| Search Method | Behavior |
|---|---|
| `.search("term")` | Full-word matching (default) |
| `.autocomplete("term")` | Partial-word matching |
| `.facet("field")` | Returns OrderedDict of ID → count |
| `operator="or"` | Match any term |
| `operator="and"` | Match all terms |

## Worked Example
Building a search page with filters:

```python
# views.py
from django.shortcuts import render
from wagtail.models import Page
from wagtail.contrib.search_promotions.models import Query

def search(request):
    search_query = request.GET.get("query", None)
    if search_query:
        search_results = Page.objects.live().search(search_query)
        Query.get(search_query).add_hit()
    else:
        search_results = Page.objects.none()
    return render(request, "search_results.html", {
        "search_query": search_query,
        "search_results": search_results,
    })
```

## Key Takeaways
1. Pages, Images, and Documents are indexed automatically — only add `search_fields` for extra fields.
2. Use `SearchField` for full-text, `AutocompleteField` for typeahead, `FilterField` for facets.
3. The database backend is good enough for most sites; Elasticsearch/OpenSearch for advanced needs.
4. Run `update_index` periodically even with `AUTO_UPDATE = True` as a safety net.
5. Use `parse_query_string` to support phrase and filter syntax in user search input.

## Connects To
- **Ch 8**: Snippets can be made searchable by inheriting `index.Indexed` — a search box appears in the chooser.
- **Ch 9**: Search results are rendered with standard Django templates; use `{% pageurl %}` for result links.
- **Ch 10**: Search results respect page-level permissions — unpublished pages only appear for users with edit/publish permission.
