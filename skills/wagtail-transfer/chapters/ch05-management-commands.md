# Management Commands

## Core Idea
`preseed_transfer_table` populates the UUID mapping table with predictable values so that content already present on both sites is treated as updates, not duplicates.

## Frameworks Introduced
- **Pre-seeding UUIDs**: For sites with existing content that need wagtail-transfer retroactively, run `preseed_transfer_table` to generate consistent UUIDs between source and destination. This ensures future imports recognize existing content.

## Key Concepts
- **model_or_app**: Can be a model label (e.g. `wagtailcore.page`) or an app label (e.g. `wagtaildocs`). App labels assign UUIDs to all models in the app.
- **--range=MIN-MAX**: Optionally limit pre-seeding to a specific ID range.
- **Multi-table inheritance**: Only the base model needs pre-seeding (e.g. `wagtailcore.page`), not specific page types. But `ParentalKey`/`InlinePanel` related models need their own UUIDs.

## Worked Example

### Example 1: Launch with wagtail-transfer
- Staging site has content. Deploy to live. Run `preseed_transfer_table wagtailcore.page` on staging. Dump staging DB and restore on live. Live now has matching UUIDs — future transfers update instead of create.

### Example 2: Retrospective install on existing sites
- Both sites have pages 1-199 in common. Run `preseed_transfer_table wagtailcore.page --range=1-199` on both. UUIDs are generated consistently — pages 1-199 are recognized as updates.

## Key Takeaways
1. `preseed_transfer_table` solves the "both sites already have content" problem.
2. Only base models need pre-seeding for multi-table inheritance; ParentalKey children need separate runs.
3. For standard Django/Wagtail models: `preseed_transfer_table auth wagtailcore wagtailimages.image wagtaildocs`.

## Connects To
- **Ch 03**: How It Works — pre-seeding populates the IDMapping table
- **Ch 01**: Setup — this is the step before you start importing
