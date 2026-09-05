# How It Works

## Core Idea
Wagtail Transfer uses ID Mapping (UID-based) to track imported content across sites, and recursively follows references to import related objects — unless they're in the no-follow list.

## Frameworks Introduced
- **ID Mapping**: Each imported model gets an IDMapping entry that maps local IDs from source and destination to a shared UID. This enables re-imports to identify and update the same object across sites.
- **Finding References**: When importing, Wagtail Transfer recursively follows ForeignKeys and rich text references to import related models. Stops at WAGTAILTRANSFER_NO_FOLLOW_MODELS (default: Page).

## Key Concepts
- **IDMapping**: Model that maps source/destination local IDs to a shared UUID.
- **Lookup Fields**: Alternative to IDMapping — use model fields (e.g. first_name + surname) for matching instead of UUIDs.
- **Recursive import**: Referenced objects are imported first, then the referencing object.
- **No-follow models**: Models in this list are not recursively imported; references become broken if possible.
- **Update related models**: Non-Page models already on both sites are NOT updated unless listed in UPDATE_RELATED_MODELS.

## Mental Models
- Think of IDMapping as a "shared address book" — both sites agree on the same UID for the same logical object.
- Recursive import is like a dependency resolver: it pulls in what's needed before the main object.

## Anti-patterns
- **Assuming local IDs match**: Source and destination local IDs are never guaranteed to be the same — always rely on UID or lookup fields.
- **Not pre-seeding for existing sites**: If you have existing content on both sites, you must preseed the transfer table or get duplicates.

## Key Takeaways
1. IDMapping is the backbone — it tracks which objects are the same across sites.
2. Lookup fields provide an alternative for models with natural unique keys.
3. Recursive import stops at no-follow models; references become broken links.
4. Non-Page models are NOT updated on re-import unless in UPDATE_RELATED_MODELS.

## Connects To
- **Ch 02**: Basic Usage — the import flows rely on ID Mapping
- **Ch 04**: Settings — WAGTAILTRANSFER_NO_FOLLOW_MODELS, UPDATE_RELATED_MODELS, LOOKUP_FIELDS
