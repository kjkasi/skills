# Chapter 7: Branching & Merging

## Core Idea
Branches occur when two revisions share the same parent. Use `alembic merge` to create a diamond-shaped merge point, or work with explicit branches using branch labels and `--head` syntax. The system uses topological sorting over a DAG.

## Frameworks Introduced
- **Branch Labels**: Named aliases for revision chains (e.g., `shoppingcart`) — enable `branchname@head` syntax
- **Merge Point**: A revision with multiple `down_revision` entries that joins branches back into a single head
- **depends_on**: Cross-branch dependency without merging — a revision declares it needs another branch's revision applied first

## Key Concepts
- **Multiple heads**: When branches exist, `alembic upgrade head` fails — specify `branchname@head` or merge
- **branch_labels**: String names applied to revisions, inherited by descendants up to the next branch point
- **version_locations**: Multiple directories for version files — each branch can live in its own directory
- **effective head**: A revision displayed in `alembic heads` that isn't a "real" head but is needed as a dependency

## Mental Models
- Branches create a **DAG (directed acyclic graph)** — Alembic traverses it via topological sort
- Merge points create **diamond shapes** — databases on either branch will apply the missing path before crossing the merge
- `depends_on` = "I need this other branch applied first, but we're not merging"

## Anti-patterns
- **Using `alembic upgrade head` with multiple heads**: Ambiguous — always specify which head or merge
- **Over-branching without labels**: Hard to track — use branch labels and version_locations to organize
- **Forgetting to merge before adding new revisions**: `alembic revision` fails with multiple heads unless `--head` is specified

## Code Examples
```bash
# Merge two branches
$ alembic merge -m "merge branches" ae1027 27c6a

# Create revision on a specific branch
$ alembic revision -m "add column" --head shoppingcart@head

# Upgrade to a specific branch's head
$ alembic upgrade shoppingcart@head
```
- **What it demonstrates**: Core branching and merging commands

```python
# Migration with branch label
revision = '27c6a30d7c24'
down_revision = '1975ea83b712'
branch_labels = ('shoppingcart',)
```
- **What it demonstrates**: Labeling a branch for named access

```python
# Cross-branch dependency
revision = '2a95102259be'
down_revision = '29f859a13ea'
depends_on = '55af2cb1c267'  # needs this revision from another branch
```
- **What it demonstrates**: Declaring a dependency without merging

## Key Takeaways
1. `alembic merge` creates a merge point — resolves multiple heads into one
2. Use `branch_labels` for named branches — enables `branchname@head` syntax
3. `--head shoppingcart@head` adds revisions to a specific branch
4. `depends_on` creates cross-branch dependencies without merging
5. `version_locations` lets each branch live in its own directory

## Connects To
- **Ch 2**: Revision chain is a linked list; branches turn it into a DAG
- **Ch 9**: Cookbook shows patterns for managing branch-heavy environments
