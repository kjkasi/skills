# Chapter 10: Permissions

## Core Idea
Wagtail extends Django's permission system with page-tree-based permissions, collection-based image/document permissions, and group-based role management. Permissions propagate down the page tree and are configured through the Groups admin interface.

## Frameworks Introduced
- **Page Tree Permissions**: Assign add, edit, publish, bulk delete, lock, or unlock permissions at any node in the page tree; they propagate to all descendant pages.
  - When to use: Controlling which users can create, edit, or publish content in specific sections of the site.
  - How: In the Groups admin, select a group, edit page permissions, choose a page node and permission type.
- **Collection Permissions**: Control access to images and documents via collection membership; root collection permissions apply globally.
  - When to use: When different teams manage different sets of images/documents, or when documents need privacy controls.
  - How: Create collections in Settings → Collections, then assign add/edit/choose permissions per collection in the Groups admin.
- **Custom Permissions**: Register model-specific permissions that appear in the Groups admin under "Other permissions".
  - When to use: When you need permissions beyond the built-in page/image/document types.
  - How: Define permissions on the model, register with `register_permissions` hook.

## Key Concepts
- **Add permission**: Create subpages under a node; edit/delete own pages; published pages require publish permission to delete.
- **Edit permission**: Edit and delete this page and all descendants, regardless of ownership.
- **Publish permission**: Publish/unpublish pages; independent of edit permission — a user with only publish cannot make edits.
- **Bulk delete permission**: Delete pages with descendants in one operation; requires add or edit permission to actually delete.
- **Lock/Unlock permission**: Lock pages to prevent editing; unlock pages locked by others (without unlock, only the lock owner and superusers can unlock).
- **Choose permission**: Controls which collections appear in the image/document chooser UI; not a security mechanism — editors can bypass it.
- **Page ownership**: The user who creates a page becomes its owner; owners can always edit their own pages with add permission.
- **Root collection**: Default collection for all images/documents; permissions here apply to all collections.

## Mental Models
- Page permissions form a tree: set a permission at any node and it flows down to all children.
- Add permission includes edit for owned pages — this is intentional because creating pages is iterative (draft → edit → publish).
- Publish permission is separate from edit — this enables moderation workflows where editors submit and publishers approve.
- Collection permissions are independent of page permissions — an image in a private collection is restricted regardless of which page it appears on.

## Anti-patterns
- **Assuming "choose" permission hides documents**: It's a UI affordance, not security — editors can bypass the chooser to reference any document.
- **Granting publish permission too broadly**: Publish permission bypasses moderation; only give it to trusted users who should make content live.
- **Forgetting page ownership**: Users with add permission can always edit their own pages — this is by design but can surprise administrators.
- **Not using bulk delete for large trees**: Without it, users must delete leaf pages one by one before deleting parents.

## Code Examples
```python
# custom permission for the Wagtail admin
from django.contrib.auth.models import Permission
from django.contrib.contenttypes.models import ContentType
from wagtail.admin.models import Admin

content_type = ContentType.objects.get_for_model(Admin)
permission = Permission.objects.create(
    content_type=content_type,
    codename="can_do_something",
    name="Can do something",
)
```
- **What it demonstrates**: Creating a custom permission visible in the Groups admin.

```python
# FieldPanel with permission restriction
from wagtail.admin.panels import FieldPanel

content_panels = Page.content_panels + [
    FieldPanel("secret_notes", permission="can_edit_secret_notes"),
]
```
- **What it demonstrates**: Restricting a field in the editor to users with a specific permission.

```python
# PanelGroup with permission restriction
from wagtail.admin.panels import ObjectList, TabbedInterface

edit_handler = TabbedInterface([
    ObjectList([FieldPanel("title")], heading="Content"),
    ObjectList([FieldPanel("analytics_code")], heading="Analytics",
               permission="can_manage_analytics"),
])
```
- **What it demonstrating**: Restricting an entire tab to a specific permission.

## Reference Tables

| Permission | Scope | Effect |
|---|---|---|
| **Add** | Page node | Create subpages; edit/delete own pages |
| **Edit** | Page node | Edit/delete this page and all descendants |
| **Publish** | Page node | Publish/unpublish pages; bypass moderation |
| **Bulk delete** | Page node | Delete pages with descendants in one operation |
| **Lock** | Page node | Lock pages to prevent editing by others |
| **Unlock** | Page node | Unlock pages locked by other users |
| **Add** (collection) | Collection | Upload images/documents to this collection |
| **Edit** (collection) | Collection | Edit images/documents in this collection |
| **Choose** | Collection | Show collection in chooser UI (not security) |

| Collection Management | Effect |
|---|---|
| **Add** | Create sub-collections |
| **Edit** | Rename, move, change privacy settings |
| **Delete** | Delete empty collections with no sub-collections |

| Permission Location | Where It Appears |
|---|---|
| Page permissions | Groups → Edit group → Page permissions |
| Image/document permissions | Groups → Edit group → Image/document permissions |
| Collection management | Groups → Edit group → Collection management permissions |
| Custom permissions | Groups → Edit group → Other permissions |
| Field-level permissions | `FieldPanel(permission="codename")` |

## Worked Example
Setting up a multi-team content structure:

```
Root/
    Marketing/
        Campaigns/
        Landing Pages/
    Engineering/
        Documentation/
        API Reference/
    Legal/
        Terms/
        Privacy Policy/
```

Configuration:
- **Marketing group**: Edit permission on Marketing node, publish on Campaigns
- **Engineering group**: Edit permission on Engineering node, no publish (submit for review)
- **Legal group**: Edit and publish on Legal node
- **Marketing images**: Separate "Marketing" collection with add/edit for Marketing group
- **All others**: Root collection choose permission for all images/documents

This structure ensures:
- Each team can only edit their section
- Engineering content goes through moderation
- Marketing images are isolated from other teams
- Everyone can use images from the root collection

## Key Takeaways
1. Page permissions propagate down the tree — set them at the highest relevant node.
2. Add permission includes edit for owned pages; edit permission covers all pages in the subtree.
3. Publish permission is independent — it controls visibility, not editing ability.
4. "Choose" permission is a UI guide, not security; use private collections for actual access control.
5. Custom permissions appear in the Groups admin when registered via the `register_permissions` hook.
6. Field-level and panel-group permissions let you restrict parts of the editor interface.

## Connects To
- **Ch 6**: Document privacy is enforced via collection permissions and the serve method.
- **Ch 7**: Search results respect page permissions — unpublished content only appears for authorized users.
- **Ch 8**: Snippets with `DraftStateMixin` or `LockableMixin` automatically get publish/lock permissions in the Groups admin.
- **Ch 9**: `{% wagtailuserbar %}` shows admin actions based on the current user's permissions.
