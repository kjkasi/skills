# Patterns

## Import Flow Pattern
**When to use**: Importing content between Wagtail instances.
**How**: Open admin → Import → Select source → Choose content (page tree or snippet) → Import.
**Trade-offs**: Page Chooser imports trees; Model Chooser imports individual snippets. Choose based on granularity needed.

## Pre-seeding Pattern
**When to use**: Existing sites with content that need wagtail-transfer retroactively.
**How**: Run `preseed_transfer_table` on both source and destination with matching parameters.
**Trade-offs**: Must run before first import. Only needed once per site pair.

## Identity Matching Pattern
**When to use**: Models with natural unique keys (e.g. authors by name, tags by slug).
**How**: Add entry to WAGTAILTRANSFER_LOOKUP_FIELDS with the model label and field list.
**Trade-offs**: UUID matching is default and works universally; lookup fields are more human-readable but require careful field selection.

## Recursive Import Control Pattern
**When to use**: Preventing unwanted cascading imports (e.g. all linked pages).
**How**: Add models to WAGTAILTRANSFER_NO_FOLLOW_MODELS. References become broken if possible.
**Trade-offs**: Stops unwanted cascading but creates broken references that may need manual fixing.

## Reverse Relation Following Pattern
**When to use**: Importing models that have reverse relations (e.g. tags on images).
**How**: Add (model, relation_name, track_deletions) to WAGTAILTRANSFER_FOLLOWED_REVERSE_RELATIONS.
**Trade-offs**: track_deletions=true deletes destination objects not in source — use only for true child models.
