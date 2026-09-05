# Chapter 13: Client-Side Customization

## Core Idea
Wagtail's admin can be extended with custom JavaScript (Stimulus, React, or vanilla JS), CSS, Draftail rich text editor plugins, and client-side panel APIs. The approach ranges from simple DOM event hooks to full React component integration.

## Frameworks Introduced
- **Stimulus**: Lightweight JS framework for server-rendered HTML. Wagtail's recommended approach for new client-side interactivity.  - When to use: Custom widgets, dynamic form behavior, interactive UI elements.  - How: Create a controller class extending `window.StimulusModule.Controller`, register via `window.wagtail.app.register()`.
- **React**: Used for complex admin components (sidebar, comments, Draftail).  - When to use: Extending or customizing existing React-based admin components.  - How: Access globals `window.React`, `window.ReactDOM`, `window.wagtail.components.*`.
- **Draftail**: Wagtail's rich text editor built on Draft.js.  - When to use: Adding custom formatting, entities, or toolbar controls to rich text.  - How: Register features via `register_rich_text_features` hook.

## Key Concepts
- **`insert_global_admin_js` hook**: Injects JavaScript into every admin page.
- **`insert_editor_js` hook**: Injects JavaScript into pages with rich text editors.
- **`window.wagtail.app`**: The core Stimulus application instance for registering controllers.
- **`window.StimulusModule`**: Exposed Stimulus library for extending without a build system.
- **`data-controller`**: Stimulus attribute that binds an HTML element to a controller.
- **`data-action`**: Stimulus attribute defining event-to-method mappings (e.g., `click->my-controller#handle`).
- **`data-*-value`**: Stimulus attributes for passing configuration values to controllers.
- **`window.wagtail.editHandler`**: Client-side Panel object for accessing form contents programmatically.
- **`getBoundWidget()`**: Returns the widget instance managed by a FieldPanel, giving access to form field values.
- **Inline styles / Blocks / Entities**: Draftail's three extension types for formatting content.
- **Rewrite handlers**: Server-side classes that convert rich text internal format (`<a linktype="page">`) to HTML.

## Mental Models
1. **Progressive enhancement tiers**: Start with vanilla JS + DOM events → use Stimulus for dynamic widgets → use React only for complex components like Draftail extensions.
2. **Controller-HTML contract**: Stimulus controllers are bound to HTML via `data-*` attributes. The HTML declares behavior; the JS implements it. No manual initialization needed.
3. **Feature registry pipeline**: Rich text features flow through: `register_rich_text_features` hook → editor plugin registration → converter rule registration → editor widget rendering.
4. **Panel as a stable API**: `window.wagtail.editHandler` provides a structured way to access form data without depending on HTML structure, which may change between versions.

## Anti-patterns
- **Using jQuery**: Deprecated and will be removed. Use vanilla JS or Stimulus instead.
- **Hardcoding DOM selectors**: HTML structure may change between Wagtail releases. Use `window.wagtail.editHandler` or Stimulus data attributes.
- **Building React components without understanding Draft.js**: Entity extensions require knowledge of Draft.js APIs. Consider StreamField for block-level content instead.
- **Loading JS before admin core files**: Use hooks (`insert_global_admin_js`, `insert_editor_js`) to ensure proper load order.

## Code Examples
```javascript
// Simple Stimulus controller
class MyController extends window.StimulusModule.Controller {
    static targets = ['label'];
    connect() {
        console.log('Connected:', this.element.innerText);
    }
}
window.wagtail.app.register('my-controller', MyController);
```
- **What it demonstrates**: Basic Stimulus controller registration with the Wagtail admin app.

```python
# Loading the controller via hook
# wagtail_hooks.py
from django.templatetags.static import static
from django.utils.html import format_html
from wagtail import hooks

@hooks.register("insert_global_admin_js")
def global_admin_js():
    return format_html('<script src="{}"></script>', static("js/example.js"))
```
- **What it demonstrates**: Injecting custom JavaScript into all admin pages.

```javascript
// Word count controller with dynamic values
class WordCountController extends window.StimulusModule.Controller {
    static values = { max: { default: 10, type: Number } };
    connect() { this.updateCount(); }
    updateCount(event) {
        const value = event ? event.target.value : this.element.value;
        const words = (value || '').split(' ');
        this.output.textContent = `${words.length} / ${this.maxValue} words`;
    }
}
window.wagtail.app.register('word-count', WordCountController);
```
- **What it demonstrates**: Stimulus controller with configurable values and event-driven updates.

```python
# Attaching Stimulus data attributes to a form widget
from django.forms import TextInput

class ColorWidget(TextInput):
    def build_attrs(self, *args, **kwargs):
        attrs = super().build_attrs(*args, **kwargs)
        attrs["data-controller"] = "color"
        attrs["data-color-theme-value"] = self.theme
        return attrs
```
- **What it demonstrates**: Building Stimulus-compatible widgets using Django's `build_attrs`.

```python
# Draftail inline style feature (mark)
@hooks.register("register_rich_text_features")
def register_mark_feature(features):
    features.register_editor_plugin(
        "draftail", "mark",
        draftail_features.InlineStyleFeature({"type": "MARK", "label": "☆"})
    )
    features.register_converter_rule("contentstate", "mark", {
        "from_database_format": {"mark": InlineStyleElementHandler("MARK")},
        "to_database_format": {"style_map": {"MARK": "mark"}},
    })
    features.default_features.append("mark")
```
- **What it demonstrates**: Adding a custom inline style to the Draftail editor.

## Reference Tables

| Hook | Scope | Use Case |
|---|---|---|
| `insert_global_admin_js` | All admin pages | Global utilities, Stimulus controllers |
| `insert_editor_js` | Rich text editor pages | Draftail plugins, editor-specific JS |
| `register_rich_text_features` | Rich text configuration | Formatting, blocks, entities |

| Draftail Extension Type | Description | Complexity |
|---|---|---|
| InlineStyleFeature | Bold, italic, custom inline formatting | Low |
| BlockFeature | Block-level elements (blockquote, custom blocks) | Low |
| EntityFeature | Rich data (links, images, custom embeds) | High (requires React) |
| ControlFeature | Custom toolbar UI elements | Medium |
| DecoratorFeature | Text decoration/highlighting | Medium |
| PluginFeature | Full Draft.js plugin API access | High |

| Client Panel Object | Method | Description |
|---|---|---|
| `Panel` | `getPanelByName(name)` | Find descendant panel by field name |
| `FieldPanel` | `getBoundWidget()` | Get widget instance with form field value |
| `FieldPanel` | `getErrorMessage()` | Get current error message |
| `FieldPanel` | `setErrorMessage(msg)` | Set or clear error message |
| `InlinePanel` | `addForm()` | Append a new blank form |

## Worked Example
Add a word count indicator to a rich text field:
1. Create `WordCountController` extending `window.StimulusModule.Controller`.
2. Register it: `window.wagtail.app.register('word-count', WordCountController)`.
3. Load via `insert_editor_js` hook.
4. On a `FieldPanel`, set `widget=forms.TextInput(attrs={"data-controller": "word-count", "data-word-count-max-value": 5, "data-action": "input->word-count#updateCount"})`.
5. The controller auto-connects when the element appears, including in dynamically added InlinePanels.

## Key Takeaways
1. Start simple — vanilla JS + DOM events may be sufficient. Use Stimulus for dynamic widgets.
2. Use hooks to inject JS/CSS at the right scope (global admin vs. editor pages).
3. Draftail inline styles and blocks are straightforward; entities require React knowledge.
4. `window.wagtail.editHandler` provides a stable API for accessing panel/form data without depending on HTML structure.
5. Stimulus controllers auto-initialize for dynamically added elements (modals, InlinePanel, StreamField).

## Connects To
- **Ch 12 (Admin Views)**: Server-side admin extensions pair with client-side customization for complete admin experiences.
- **Ch 14 (API)**: Rich text internal format (rewrite handlers) is the same format the API serves and consumes.
