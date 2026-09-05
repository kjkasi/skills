# Glossary

- **IDMapping** — Model that maps local IDs from source and destination sites to a shared UID for tracking imported content (Ch 3)
- **UID** — Unique identifier shared across sites to identify the same logical object regardless of local ID differences (Ch 3)
- **Lookup Fields** — Configuration that uses model field values (e.g. first_name + surname) instead of UUIDs for object matching (Ch 4)
- **No-Follow Models** — Models that are not recursively imported when referenced from imported content (default: Page, ContentType) (Ch 4)
- **Update Related Models** — List of models that should be updated to the latest version from source on re-import (Ch 4)
- **Followed Reverse Relations** — Configuration for following reverse relationships to import related models like tags (Ch 4)
- **Chooser API Proxy Timeout** — Timeout in seconds for API calls to browse the source page tree (default: 5) (Ch 4)
- **Field Adapters** — Classes that serialize/deserialize field data during export/import; extensible via hooks (Ch 4)
- **Custom Serializer** — Override of PageSerializer to control which fields are exported for a model (Ch 4)
- **Preseed Transfer Table** — Management command to populate UUID mappings for sites with existing content (Ch 5)
- **Multi-Table Inheritance** — Django inheritance where only the base model needs UUID pre-seeding (Ch 5)
