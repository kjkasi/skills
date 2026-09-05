# Chapter 1: Getting Started

## Core Idea
Wagtail is a Django-based CMS that provides a powerful editing interface while maintaining full developer control through Python code, not drag-and-drop interfaces.

## Frameworks Introduced
- **Wagtail Project Template**: Generate a complete Wagtail project with `wagtail start mysite`
  - When to use: Starting a new Wagtail project from scratch
  - How: Run `pip install wagtail`, then `wagtail start mysite`, then follow Django setup steps
- **Integration into Existing Django**: Add Wagtail to an existing Django project
  - When to use: You have an established Django project and want to add CMS capabilities
  - How: Install wagtail, add apps to INSTALLED_APPS, configure settings, add URL patterns

## Key Concepts
- **Wagtail**: Open-source CMS built on Django with powerful editing interface and modular architecture
- **Page Model**: All page types inherit from `wagtail.models.Page` and are Django models
- **Admin Interface**: Wagtail's separate admin backend (not Django admin) for content management
- **The Zen of Wagtail**: Design principles emphasizing clear role separation between content authors, site administrators, developers, and designers

## Mental Models
- Use `wagtail start` when building a new standalone Wagtail site from scratch
- Use manual integration when adding CMS features to an existing Django project
- Think of Wagtail as providing the editing interface while you write the code for structure and behavior

## Anti-patterns
- **Pushing too much power to content authors**: Don't let editors make design and layout decisions within the content editing interface
- **Using Rich Text for structured data**: Avoid putting event dates, locations, etc. as styled text; use dedicated fields instead
- **Making everything configurable through admin**: Some features are better maintained in code than through the admin interface

## Code Examples
```python
# Settings configuration for Wagtail integration
INSTALLED_APPS = [
    # ...
    "wagtail.contrib.forms",
    "wagtail.contrib.redirects",
    "wagtail.embeds",
    "wagtail.sites",
    "wagtail.users",
    "wagtail.snippets",
    "wagtail.documents",
    "wagtail.images",
    "wagtail.search",
    "wagtail.admin",
    "wagtail",
    "modelcluster",
    "taggit",
]

MIDDLEWARE = [
    # ...
    "wagtail.contrib.redirects.middleware.RedirectMiddleware",
]
```
- **What it demonstrates**: Required Wagtail apps and middleware for integration into existing Django project

## Reference Tables
| Setting | Purpose | Recommended Value |
|---------|---------|-------------------|
| `WAGTAIL_SITE_NAME` | Displayed on admin dashboard | Your site name |
| `WAGTAILADMIN_BASE_URL` | Base URL for admin emails | https://example.com |
| `DATA_UPLOAD_MAX_NUMBER_FIELDS` | Max form fields per submission | 10,000 |
| `STATIC_ROOT` | Static files directory | BASE_DIR / "static" |
| `MEDIA_ROOT` | User-uploaded files directory | BASE_DIR / "media" |

## Worked Example
Starting a new Wagtail project:
1. Create virtual environment
2. Run `pip install wagtail`
3. Run `wagtail start mysite`
4. Navigate to project: `cd mysite`
5. Install dependencies: `pip install -r requirements.txt`
6. Run migrations: `python manage.py migrate`
7. Create superuser: `python manage.py createsuperuser`
8. Start server: `python manage.py runserver`
9. Access site at http://localhost:8000, admin at http://localhost:8000/admin/

## Key Takeaways
1. Wagtail is not an instant website in a box; expect to write code
2. Always wear the right hat: distinguish between content author, site administrator, developer, and designer roles
3. A CMS should get information out of an editor's head into a database, not dictate how that information should look
4. The best user interface for a programmer is usually a programming language

## Connects To
- **Ch 2**: Building a portfolio site applies these setup concepts in a practical project
- **Ch 3**: Page models introduced here are explored in depth in the pages chapter