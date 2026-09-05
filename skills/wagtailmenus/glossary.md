# Glossary

**AbstractLinkPage** — Abstract model for creating link-only pages that appear in menus without full page content (Ch 8)

**AbstractMainMenu** — Abstract model for customizing the main menu model (Ch 9)

**AbstractMainMenuItem** — Abstract model for customizing main menu item fields and behavior (Ch 9)

**AbstractFlatMenu** — Abstract model for customizing the flat menu model (Ch 9)

**AbstractFlatMenuItem** — Abstract model for customizing flat menu item fields and behavior (Ch 9)

**Active Class** — CSS class applied to the current page's menu item (default: 'active') (Ch 5)

**Active Ancestor Class** — CSS class applied to ancestors of the current page in menus (default: 'ancestor') (Ch 5)

**allow_repeating_parents** — Template tag argument controlling whether MenuPage repetition is respected (Ch 7)

**allow_subnav** — Boolean field on menu items controlling whether children from page tree appear in sub-menus (Ch 3)

**ancestors** — Pages higher in the page tree than the current page; receive ancestor_class in menus (Ch 5)

**autopopulate_main_menus** — Management command to seed main menus from existing page tree structure (Ch 2)

**ChildrenMenu** — Python class driving children_menu tag behavior for rendering page children (Ch 9)

**children_menu** — Template tag that renders children of a specific page (Ch 5)

**context_processors.wagtailmenus** — Required Django context processor for menu template tags to access menu data (Ch 2)

**CustomMenuClasses** — System for overriding menu and menu item models through abstract models and settings (Ch 9)

**DEFAULT_ADD_SUB_MENUS_INLINE** — Setting controlling whether sub-menus are added inline by default (Ch 11)

**DEFAULT_CHILDREN_MENU_TEMPLATE** — Setting for default children_menu tag template (Ch 11)

**DEFAULT_FLAT_MENU_TEMPLATE** — Setting for default flat_menu tag template (Ch 11)

**DEFAULT_MAIN_MENU_TEMPLATE** — Setting for default main_menu tag template (Ch 11)

**DEFAULT_SECTION_MENU_TEMPLATE** — Setting for default section_menu tag template (Ch 11)

**DEFAULT_SUB_MENU_TEMPLATE** — Setting for default sub_menu tag template (Ch 11)

**fall_back_to_default_site_menus** — Template tag argument enabling cross-site menu reuse for flat menus (Ch 4)

**FlatMenu** — CMS-managed menu not tied to page tree; can link to pages or custom URLs (Ch 4)

**FlatMenuItem** — Individual items within a flat menu; can link to pages or custom URLs (Ch 4)

**flat_menu** — Template tag that renders a named flat menu by handle (Ch 5)

**FLATMENU_MENU_ICON** — Setting for flat menu admin icon in Wagtail CMS (Ch 11)

**FLAT_MENUS_ADMIN_CLASS** — Setting for custom flat menu admin class (Ch 11)

**FLAT_MENUS_EDITABLE_IN_WAGTAILADMIN** — Setting controlling flat menu editing in Wagtail admin (Ch 11)

**FLAT_MENUS_HANDLE_CHOICES** — Setting constraining flat menu handle choices in admin (Ch 11)

**handle** — Unique identifier for a flat menu (e.g., 'footer', 'sidebar') (Ch 4)

**hooks** — Django signals fired at specific points in menu rendering lifecycle (Ch 10)

**link_page** — ForeignKey field on menu items pointing to a target page (Ch 8)

**link_text** — CharField for custom display text on menu items (Ch 8)

**link_url** — URLField for custom URLs on menu items; supports anchors like #signup (Ch 8)

**main_menu** — Template tag that renders the main navigation menu for the current site (Ch 5)

**MAIN_MENU_ITEMS_RELATED_NAME** — Setting linking custom main menu item model to main menu (Ch 9)

**MAIN_MENU_MODEL** — Setting for custom main menu model (Ch 9)

**MainMenu** — Site-specific model storing the main navigation menu (Ch 3)

**MainMenuItem** — Individual items within a main menu; linked to pages or custom URLs (Ch 3)

**max_levels** — Template tag argument controlling how many levels deep a menu renders (Ch 5)

**MenuPage** — Abstract model extending Page to allow pages to repeat in their own sub-menus (Ch 7)

**MenuPageMixin** — Mixin class providing menu repetition behavior without full model replacement (Ch 7)

**menu_items** — List of menu item objects available in templates and signals (Ch 5)

**natural order** — The page tree ordering applied to sub-menu items (Ch 3)

**Page** — Wagtail's base model for website content; parent class for MenuPage (Ch 7)

**repeat_in_menu** — Boolean field on MenuPage controlling whether page appears in its own sub-menu (Ch 7)

**repeat_menu_text** — CharField for custom text on repeated MenuPage items (Ch 7)

**section_menu** — Template tag that renders a menu for the current page's section of the tree (Ch 5)

**SectionMenu** — Plain Python class driving section_menu tag behavior (Ch 9)

**SECTION_MENU_CLASS** — Setting for custom section menu class (Ch 9)

**show_in_menus** — Wagtail field controlling page visibility in all menus (default: True) (Ch 1)

**show_menu_heading** — Template tag argument controlling heading display above flat menus (Ch 5)

**show_multiple_levels** — Template tag argument controlling multi-level output (Ch 5)

**SITE_SPECIFIC_TEMPLATE_DIRS** — Setting enabling per-site template directories (Ch 11)

**sub_menu** — Recursive template tag for rendering sub-levels within a menu (Ch 5)

**sub_menu_template** — Template used for rendering sub-levels; inherited by recursive calls (Ch 6)

**sub_menu_templates** — Multiple templates for different sub-menu levels (Ch 6)

**url_append** — CharField for additional path/params added to page URLs (Ch 8)

**use_absolute_page_urls** — Template tag argument controlling relative vs absolute URLs (Ch 5)

**wagtailmenus** — Django/Wagtail app providing menu management through CMS admin (Ch 1)
