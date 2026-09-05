# Chapter 15: Headless Wagtail

## Core Idea
Wagtail can serve as a headless CMS backend, exposing content via REST or GraphQL APIs while the frontend is built with frameworks like Next.js, Nuxt.js, or Gatsby. While well-supported for many use cases, some features (page preview, form submissions, password-protected pages) have limitations that require workarounds.

## Frameworks Introduced
- **REST API (built-in)**: Wagtail's native API for serving content to external frontends.  - When to use: Default choice for headless projects; simple, well-documented, no extra dependencies.  - How: Enable `wagtail.api.v2` or `wagtail.api.v3` in `INSTALLED_APPS`.
- **GraphQL (wagtail-grapple)**: GraphQL API layer for Wagtail, enabling flexible querying.  - When to use: Complex data requirements, multiple related resources in one query.  - How: Install `wagtail-grapple`, configure types for your models.
- **strawberry-wagtail**: Alternative GraphQL library for Wagtail.  - When to use: Preference for Strawberry over Graphene.  - How: Install and configure alongside Wagtail.
- **wagtail-headless-preview**: Third-party package for previewing draft content in headless frontends.  - When to use: When editors need to preview unpublished changes before publishing.  - How: Install the package, configure preview URL in frontend.

## Key Concepts
- **Access tiers**: The API serves different content based on authentication — anonymous sees live/public pages only; authenticated sees draft and restricted pages.
- **Content checker**: Accessibility checker that works in headless mode when the user bar is loaded in the frontend.
- **User bar**: Cross-domain widget enabling live preview scroll restoration, content checking, and metrics in the page editor.
- **Image rendition**: Pre-defined image sizes served via `ImageRenditionField` or the dynamic image serve view.
- **Internal rich text format**: Wagtail stores rich text with custom tags (`<a linktype="page">`, `<embed embedtype="image">`); the API returns this unprocessed by default.
- **Site record mismatch**: In headless mode, the Wagtail site record must match the API domain, not the frontend domain.
- **Page URL routing**: Headless projects configure routing in the frontend framework, not Wagtail; "View Live" links may resolve incorrectly.

## Mental Models
1. **Backend/frontend separation**: Wagtail handles content management and storage; the frontend framework handles rendering and routing. The API is the bridge.
2. **Two-site domain problem**: Wagtail's site record must be set to the API domain (e.g., `api.wagtail.org`), but URLs need to generate for the frontend domain (e.g., `www.wagtail.org`). This mismatch is a core headless challenge.
3. **Preview as a workaround**: Since the API can't serve draft content to anonymous requests, preview requires either a proxy, a shared secret, or a dedicated preview package.
4. **Rich text as two formats**: Internal format (stored in DB) vs. rendered format (HTML). The API returns internal format; the frontend must either parse it or the backend must pre-render it.

## Anti-patterns
- **Ignoring the site record mismatch**: If the Wagtail site domain doesn't match the API domain, URL generation will produce incorrect links.
- **Building custom routing in Wagtail**: Stick with Wagtail's default slug-based routing for headless projects. Custom routes require more maintenance.
- **Assuming the API pre-renders rich text**: By default, the API returns internal format. Use `expand_db_html()` or a GraphQL library that pre-renders.
- **Not loading the user bar in preview routes**: The content checker, scroll restoration, and metrics won't work without it.
- **Using password-protected pages headlessly**: There's currently no API support for viewing password-protected content from a headless frontend.

## Code Examples
```python
# User bar view for cross-domain headless frontend
from django.views.generic import TemplateView
from wagtail.admin.userbar import Userbar

class UserbarView(TemplateView):
    template_name = Userbar.template_name
    http_method_names = ["get"]

    def dispatch(self, request, *args, **kwargs):
        response = super().dispatch(request, *args, **kwargs)
        response["Access-Control-Allow-Origin"] = "https://my.headless.site"
        return response

    def get_context_data(self, **kwargs):
        return Userbar(object=None, position="bottom-right").get_context_data(
            super().get_context_data(request=self.request, **kwargs)
        )
```
- **What it demonstrates**: Serving the Wagtail user bar from a Django view for cross-domain headless frontends.

```tsx
// React component loading the user bar in Next.js
'use client';
import Script from 'next/script';
import { useEffect, useRef } from 'react';

export default function Userbar({ hidden = false }) {
  const userbarRef = useRef(null);
  const apiHost = process.env.NEXT_PUBLIC_WAGTAIL_API_HOST;

  useEffect(() => {
    fetch(`${apiHost}/userbar/`)
      .then((res) => res.text())
      .then((userbar) => {
        if (!userbarRef.current || userbarRef.current.querySelector('wagtail-userbar')) return;
        userbarRef.current.innerHTML = userbar;
      });
  }, [apiHost]);

  return (
    <>
      <div hidden={hidden} ref={userbarRef} />
      <Script src={`${apiHost}/static/wagtailadmin/js/vendor.js`} />
      <Script src={`${apiHost}/static/wagtailadmin/js/userbar.js`} />
    </>
  );
}
```
- **What it demonstrates**: Loading the Wagtail user bar in a Next.js headless frontend.

```python
# Content checker with cross-domain allowed origins
from wagtail.admin.utils import get_admin_base_url

class HeadlessContentCheckerItem(ContentCheckerItem):
    def get_axe_spec(self, request):
        spec = super().get_axe_spec(request)
        spec["allowedOrigins"] = [
            "https://my.headless.site"
            if self.in_editor
            else get_admin_base_url()
        ]
        return spec
```
- **What it demonstrates**: Configuring the content checker for cross-domain headless deployments.

## Reference Tables

| Feature | Support Level | Notes |
|---|---|---|
| REST API | ✅ Full | Native v2/v3 support |
| GraphQL | ✅ Full | Via wagtail-grapple or strawberry-wagtail |
| User bar | ✅ Full | Requires cross-domain view setup |
| Content checker | ✅ Full | Needs user bar + Axe origin config |
| Page preview | ⚠️ Workaround | Use wagtail-headless-preview package |
| Images | ⚠️ Workaround | ImageRenditionField or dynamic serve view |
| Page URL routing | ⚠️ Workaround | Stick to Wagtail's default routing |
| Rich text | ⚠️ Workaround | Pre-render on backend or parse on frontend |
| Multi-site | 🛑 Limited | API checks host header; site must match API domain |
| Form submissions | 🛑 None | No official API for headless form handling |
| Password-protected pages | 🛑 None | API excludes these from queries |

| Frontend Framework | Support | Notes |
|---|---|---|
| Next.js | ⚠️ | Community examples; no official plugin |
| Nuxt.js | ⚠️ | Used by NASA JPL; no official plugin |
| Gatsby | ⚠️ | gatsby-source-wagtail plugin (requires GraphQL) |

## Worked Example
Set up a headless Wagtail site with Next.js:
1. Enable `wagtail.api.v3` (or v2) in `INSTALLED_APPS`.
2. Set the Wagtail site record domain to the API domain (e.g., `api.example.com`).
3. Create a `UserbarView` Django view serving the user bar template with CORS headers.
4. In Next.js, create a `Userbar` component that fetches from `/userbar/` and renders the response.
5. For page preview, install `wagtail-headless-preview` and configure a preview route in Next.js that passes the preview token.
6. Use `expand_db_html()` in a custom API serializer to pre-render rich text for frontend consumption.

## Key Takeaways
1. Wagtail has good headless support for REST and GraphQL APIs, with workarounds needed for preview, images, and routing.
2. The site record domain must match the API domain; frontend URLs are a separate concern.
3. Rich text requires explicit handling — either pre-render on the backend or parse on the frontend.
4. The user bar enables editor features (content checking, scroll restoration) in cross-domain frontends.
5. Form submissions and password-protected pages lack official headless API support.

## Connects To
- **Ch 14 (API)**: The API (v2/v3) is the primary interface for headless Wagtail architectures.
- **Ch 11 (Deployment)**: Headless deployments add a separate frontend hosting layer (Vercel, Netlify) alongside the Wagtail backend.
- **Ch 13 (Frontend)**: Rich text internal format and rewrite handlers are the same concepts the API and headless frontend must handle.
