# Basic Usage

## Core Idea
Two import methods: Page Chooser (imports entire page trees) and Model Chooser (imports individual snippets or entire models). Both follow the same pattern: open admin → select Import → choose source → select content → import.

## Frameworks Introduced
- **Page Chooser flow**: Import entire page trees by selecting a parent page on the source site. Descendants are imported together. First-time imports require selecting a destination parent; re-imports update existing pages.
- **Model Chooser flow**: Import individual snippet objects or entire model registrations. Works similarly but scoped to Snippet models.

## Key Concepts
- **Parent page selection**: When importing pages, select a parent page on the source site — all descendants follow.
- **Update vs Create**: If the page/object was previously imported, it updates; otherwise creates new.
- **Associated objects**: Images, documents, and other referenced objects are also imported depending on configuration.

## Anti-patterns
- **Importing without understanding ID mapping**: If pages exist on both sites without pre-seeded IDs, you'll get duplicates instead of updates.

## Key Takeaways
1. Page Chooser imports entire page trees; Model Chooser imports snippets.
2. Previously imported content is updated, not duplicated.
3. Referenced objects (images, docs) are imported based on WAGTAILTRANSFER settings.

## Connects To
- **Ch 03**: How It Works — explains the ID Mapping that enables update-vs-create
- **Ch 05**: Management Commands — preseed_transfer_table for setup scenarios
