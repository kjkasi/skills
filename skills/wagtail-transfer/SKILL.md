---
name: wagtail-transfer
description: "Knowledge base from \"Wagtail Transfer Documentation\" by Wagtail. Use when transferring content between Wagtail CMS instances, configuring import settings, pre-seeding transfer tables, or troubleshooting sync issues."
---

<!-- argument-hint: [topic, setting name, or management command] -->

# Wagtail Transfer Documentation
**Author**: Wagtail | **Pages**: ~6 docs | **Chapters**: 5 | **Generated**: 2026-09-04

## How to Use This Skill

- **Without arguments** — load core frameworks for reference
- **With a topic** — ask about `ID Mapping`, `preseed`, `settings`, or another indexed topic; I find and read the relevant chapter
- **With chapter** — ask for `ch03`; I load that specific chapter
- **Browse** — ask "what chapters do you have?" to see the full index

When you ask about a topic not covered in Core Frameworks below, I will read
the relevant chapter file before answering.

---

## Core Frameworks & Mental Models

### ID Mapping
Wagtail Transfer uses UID-based ID Mapping to track which objects are the same across source and destination sites. Each imported object gets an IDMapping entry that maps local IDs from both sites to a shared UID. This is the backbone that enables update-vs-create on re-import.

**Use ID Mapping when**: You need to track content across multiple Wagtail instances.

**Alternative — Lookup Fields**: For models with natural unique keys (e.g. authors by name), use WAGTAILTRANSFER_LOOKUP_FIELDS to match by field values instead of UUIDs.

### Two-Layer Authentication
Each site has a global WAGTAILTRANSFER_SECRET_KEY. Each source entry in WAGTAILTRANSFER_SOURCES must share that key. This prevents unauthorized content retrieval between sites.

### Recursive Reference Resolution
When importing, Wagtail Transfer follows ForeignKeys and rich text references to recursively import related objects. Stops at models in WAGTAILTRANSFER_NO_FOLLOW_MODELS (default: Page, ContentType). References to no-follow models become broken links.

### Update vs Create Logic
- First-time import → creates new object with UUID
- Re-import → updates existing object (matched by UUID or lookup fields)
- Non-Page models on both sites → NOT updated unless in UPDATE_RELATED_MODELS

### Pre-seeding UUIDs
For sites with existing content before wagtail-transfer was installed, `preseed_transfer_table` generates consistent UUIDs between source and destination. Run on both sites before first import.

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-setup.md) | Setup | SECRET_KEY, SOURCES, URL routing |
| [ch02](chapters/ch02-basic-usage.md) | Basic Usage | Page Chooser, Model Chooser, Update vs Create |
| [ch03](chapters/ch03-how-it-works.md) | How It Works | ID Mapping, Recursive Import, Lookup Fields |
| [ch04](chapters/ch04-settings.md) | Settings and Hooks | All WAGTAILTRANSFER_* settings, hooks |
| [ch05](chapters/ch05-management-commands.md) | Management Commands | preseed_transfer_table |

## Topic Index

- **Authentication** → ch01, ch04
- **Basic Usage** → ch02
- **ID Mapping** → ch03
- **Import Flow** → ch02, ch03
- **Management Commands** → ch05
- **Pre-seeding** → ch05
- **Recursive Import** → ch03
- **Settings** → ch04
- **Setup** → ch01
- **Update Related Models** → ch04

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers the Wagtail Transfer documentation only. For hands-on implementation in your codebase, combine with project-specific tools. For topics beyond this package, check related Wagtail skills or ask the agent directly.
