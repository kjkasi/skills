# Patterns

## Main Menu Setup Pattern
**When to use**: Setting up primary site navigation in a new Wagtail project
**How**: 
1. Install wagtailmenus and add to INSTALLED_APPS
2. Add context processor to TEMPLATES setting
3. Run migrations
4. Use autopopulate_main_menus command to seed initial menu
5. Edit main menu in CMS to add/reorder top-level items
**Trade-offs**: Editors gain autonomy but must understand page tree impact on sub-menus

## Flat Menu Pattern
**When to use**: Creating CMS-managed menus for footer, sidebar, or secondary navigation
**How**:
1. Create flat menu in Wagtail admin under Settings > Flat menus
2. Assign unique handle (e.g., 'footer', 'sidebar')
3. Add menu items with page links or custom URLs
4. Reference in template with {% flat_menu 'handle' %}
**Trade-offs**: Flexible but requires CMS access; custom URLs need manual maintenance

## MenuPage Repetition Pattern
**When to use**: Making pages appear both as parents and clickable items in sub-menus
**How**:
1. Subclass MenuPage instead of Page for your page type
2. Enable repeat_in_menu on page instances in CMS
3. Optionally set repeat_menu_text for custom labels
**Trade-offs**: Solves "clickable parent" problem but adds model complexity

## AbstractLinkPage Pattern
**When to use**: Adding menu links without creating full page content (external links, anchors)
**How**:
1. Subclass AbstractLinkPage to create LinkPage model
2. Run migrations
3. Create link pages in Wagtail admin
4. Add to menus like regular pages
**Trade-offs**: Lightweight but limited to links; no page content or children

## Custom Menu Item Fields Pattern
**When to use**: Adding images, descriptions, or translations to menu items
**How**:
1. Subclass AbstractMainMenuItem or AbstractFlatMenuItem
2. Add custom fields (image, description, translated fields)
3. Set related_name on ParentalKey
4. Configure WAGTAILMENUS_*_RELATED_NAME setting
5. Run migrations
**Trade-offs**: Full customization but requires model changes and migrations

## Template Auto-Discovery Pattern
**When to use**: Project-wide menu template overrides without specifying paths everywhere
**How**:
1. Create templates in app template directories at expected paths
2. wagtailmenus automatically discovers and uses them
3. Fall back to defaults if not found
**Trade-offs**: Convenient but template resolution order can be confusing

## Site-Specific Templates Pattern
**When to use**: Multi-site projects with different menu styling per site
**How**:
1. Enable WAGTAILMENUS_SITE_SPECIFIC_TEMPLATE_DIRS = True
2. Create site-specific template directories
3. Place templates in site-specific paths
**Trade-offs**: Flexible but requires template organization

## Active Class Styling Pattern
**When to use**: Visual feedback for current page and ancestors in navigation
**How**:
1. Configure ACTIVE_CLASS and ACTIVE_ANCESTOR_CLASS settings
2. Use CSS to style active states
3. Template tags automatically apply classes to menu items
**Trade-offs**: Improves UX but requires CSS work

## Cross-Site Menu Reuse Pattern
**When to use**: Sharing flat menus across multiple sites in a project
**How**:
1. Create flat menu on default site
2. Use fall_back_to_default_site_menus=True in template tag
3. Secondary sites automatically use default site's menu
**Trade-offs**: Reduces duplication but limits per-site customization

## Section Menu Pattern
**When to use**: Showing current page's section tree in sidebar or sub-navigation
**How**:
1. Use {% section_menu %} tag in templates
2. Automatically renders current page's branch of the page tree
3. Configure max_levels for depth control
**Trade-offs**: Automatic but limited to current page's tree position

## Hook-Based Modification Pattern
**When to use**: Global menu modifications (filtering, analytics, A/B testing)
**How**:
1. Connect to before_render_menu signal
2. Modify menu_items in signal handler
3. Return modified items
**Trade-offs**: Powerful but runs on every request; consider caching

## Multi-Language Menu Pattern
**When to use**: Supporting translated menu items in multi-language projects
**How**:
1. Create custom menu models with translated fields
2. Use TranslatedField descriptor for language switching
3. Override menu_text property to return translated content
**Trade-offs**: Full translation support but requires custom models

## Template Argument Override Pattern
**When to use**: One-off template overrides for specific menu instances
**How**:
1. Add template="path/to/template.html" to menu tag
2. Override applies only to that tag instance
3. Use for context-specific template variations
**Trade-offs**: Flexible but can lead to template sprawl

## Settings-Based Defaults Pattern
**When to use**: Project-wide template changes that apply to all menu tags
**How**:
1. Override DEFAULT_*_TEMPLATE settings in Django settings
2. All menu tags use new defaults unless overridden
3. Centralized configuration
**Trade-offs**: Consistent but requires understanding of settings hierarchy
