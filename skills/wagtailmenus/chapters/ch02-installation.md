# Chapter 2: Installation

## Core Idea
wagtailmenus integrates into Wagtail projects through standard Django app patterns: pip install, add to INSTALLED_APPS, configure context processor, run migrations.

## Frameworks Introduced
- **Django App Integration Pattern**: wagtailmenus follows standard Django app lifecycle (install → register → migrate → configure)
  - When to use: Adding wagtailmenus to any new or existing Wagtail project
  - How: pip install, add to INSTALLED_APPS, add context processor, run migrations

## Key Concepts
- **wagtailmenus**: Django/Wagtail app providing menu management through CMS admin
- **context_processors.wagtailmenus**: Required context processor for menu template tags to work
- **autopopulate_main_menus**: Management command to seed main menus from existing page tree
- **add-home-links**: Option for autopopulate command to include home page in main menu

## Mental Models
- Think of installation as four steps: install package → register app → add context processor → run migrations
- The autopopulate command is a one-time convenience, not a recurring sync — it only populates empty menus

## Anti-patterns
- **Skipping context processor**: Without `wagtailmenus.context_processors.wagtailmenus`, template tags won't have access to menu data
- **Re-running autopopulate**: The command only affects menus with zero items; running it repeatedly has no effect after the first run
- **Forgetting migrations**: Database tables must be created before menu functionality works

## Code Examples
```python
# settings/base.py
INSTALLED_APPS = [
    ...
    'wagtailmenus',
]

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(PROJECT_ROOT, 'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.contrib.auth.context_processors.auth',
                'django.template.context_processors.debug',
                'django.template.context_processors.i18n',
                'django.template.context_processors.media',
                'django.template.context_processors.request',
                'django.template.context_processors.static',
                'django.template.context_processors.tz',
                'django.contrib.messages.context_processors.messages',
                'wagtail.contrib.settings.context_processors.settings',
                'wagtailmenus.context_processors.wagtailmenus',
            ],
        },
    },
]
```
- **What it demonstrates**: Complete Django settings with wagtailmenus context processor

```console
pip install wagtailmenus
python manage.py migrate wagtailmenus
python manage.py autopopulate_main_menus --add-home-links
```
- **What it demonstrates**: Installation commands in order

## Reference Tables

| Step | Command/Setting | Purpose |
|------|-----------------|---------|
| 1 | `pip install wagtailmenus` | Install package |
| 2 | Add `'wagtailmenus'` to INSTALLED_APPS | Register Django app |
| 3 | Add context processor | Enable template tags |
| 4 | `python manage.py migrate wagtailmenus` | Create database tables |
| 5 (optional) | `autopopulate_main_menus` | Seed menus from page tree |

## Worked Example
Adding wagtailmenus to an existing Wagtail project with this page structure:

```
Home (root page)
├── About us
├── What we do
├── Careers
├── News & events
└── Contact us
```

Run:
```console
python manage.py autopopulate_main_menus --add-home-links
```

Creates main menu with items: Home, About us, What we do, Careers, News & events, Contact us

Note: This command is meant as "run once" to get started. It only affects menus that don't already have items defined.

## Key Takeaways
1. Installation follows standard Django patterns: pip install → INSTALLED_APPS → context processor → migrate
2. The context processor is required for template tags to access menu data
3. `autopopulate_main_menus` is a one-time convenience command for existing projects
4. Only affects empty menus — safe to run but won't overwrite existing menu items

## Connects To
- **Ch 1**: Overview — understanding what wagtailmenus provides
- **Ch 3**: Managing Main Menus — configuring the menus created by autopopulate
- **Ch 11**: Settings Reference — all available configuration options
