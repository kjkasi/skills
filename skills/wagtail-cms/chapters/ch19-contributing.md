# Chapter 19: Contributing to Wagtail

## Core Idea
Contributing to Wagtail involves setting up a development environment with Node.js and Python, following PEP8/ESLint coding standards, writing tests for all changes, and submitting focused pull requests with clear descriptions. The project uses pre-commit hooks, Ruff for Python linting, and has a structured first-contribution guide.

## Frameworks Introduced
- **Development Environment Setup**: Fork-based workflow with virtualenv, npm toolchain, and compiled static assets
  - When to use: Setting up Wagtail core development for the first time
  - How: Fork repo, `pip install -e ".[testing,docs]"`, `npm ci`, `npm run build`, use bakerydemo as test site
- **Linting and Formatting Pipeline**: Multi-tool consistency enforcement across Python, JS, CSS, and templates
  - When to use: Before every commit to avoid CI failures
  - How: Install pre-commit (`pre-commit install`), run `make lint` and `make format`
- **Test Suite Architecture**: pytest-based with parallel execution, keepdb support, and multiple database backends
  - When to use: Validating changes before submitting PRs
  - How: `python runtests.py --parallel --keepdb` for speed, `--postgres` or `--elasticsearch8` for specific backends

## Key Concepts
- **Pre-commit Hooks**: Automatic linting/formatting on every git commit via `.pre-commit-config.yaml`
- **Ruff**: Combined Python linter and formatter replacing flake8, isort, and black
- **ESLint / Prettier / Stylelint**: JavaScript and CSS formatting and linting tools
- **djhtml / Curlylint**: HTML template formatting and linting tools
- **fnm (Fast Node Manager)**: Preferred way to install correct Node.js version via `.nvmrc`
- **Bakerydemo**: Official Wagtail demo site used as development testing ground
- **Good First Issue**: GitHub label for beginner-friendly tasks
- **Draft PRs**: GitHub feature for running CI without requesting review
- **RFC Process**: Large changes go through wagtail/rfcs repository for community discussion

## Mental Models
1. **Branch-per-Issue**: Always create a new branch off `main` with descriptive name like `feature/1234-add-unit-tests`
2. **Focused Changes**: Each PR should solve one problem — separate unrelated fixes into different branches/PRs
3. **Test-First Mindset**: Writing tests often takes 5-10x longer than the fix itself but ensures long-lived solutions
4. **CI Feedback Loop**: Push to same branch for CI re-runs — never open new PRs for fixes to avoid losing review context

## Anti-patterns
- **Committing compiled assets**: Static assets are compiled before releases, not in PRs — run `npm run build` locally but don't commit output
- **Asking "please assign me this issue"**: Wagtail doesn't use issue assignment for community contributors — just start working and add a comment
- **Fixing multiple unrelated issues in one PR**: Creates review complexity — create separate branches and PRs for each fix
- **Ignoring CI failures**: Read each error, fix locally, push to same branch — never ignore failing checks
- **Using `try/except` for Django version checks**: Always use explicit `django.VERSION >= (x, y)` comparisons

## Code Examples
```sh
# Development environment setup
git clone https://github.com/username/wagtail.git
cd wagtail
pip install -e ".[testing,docs]" --config-settings editable-mode=strict -U
npm ci
npm run build
```
- **What it demonstrates**: Complete Wagtail development environment setup

```sh
# Running tests efficiently
python runtests.py --parallel --keepdb --exclude-tag=transaction
```
- **What it demonstrates**: Fast test execution with parallel runs and database reuse

```sh
# Running specific test modules
python runtests.py wagtail.admin
python runtests.py -- wagtail.tests.test_blocks.TestIntegerBlock
```
- **What it demonstrates**: Running targeted tests during development

```python
# Django compatibility pattern
from django import VERSION as DJANGO_VERSION

if DJANGO_VERSION >= (1, 9):
    related_field = field.rel
else:
    related_field = field.related
```
- **What it demonstrates**: Proper Django version checking without try/except

```sh
# Linting and formatting
make lint          # Run all linting
make format        # Run all formatting and fix linting issues
pre-commit install # Set up automatic checks on commits
```
- **What it demonstrates**: Code quality enforcement tools

## Reference Tables

| Tool | Purpose | Command |
|------|---------|---------|
| Ruff | Python linting + formatting | `make lint-server` / `make format-server` |
| ESLint | JavaScript linting | `make lint-client` |
| Prettier | JS/CSS formatting | `make format-client` |
| Stylelint | CSS linting | `make lint-client` |
| djhtml | HTML template formatting | Included in `make lint-server` |
| Curlylint | HTML template linting | Included in `make lint-server` |
| pre-commit | Git hook automation | `pre-commit install` |

| Test Command | Purpose |
|-------------|---------|
| `python runtests.py` | Run all Python tests |
| `--parallel` | Run tests in parallel |
| `--keepdb` | Reuse test database between runs |
| `--exclude-tag=transaction` | Skip TransactionTestCase tests |
| `--postgres` | Test against PostgreSQL |
| `--elasticsearch8` | Test against Elasticsearch 8 |
| `npm run test:unit` | JavaScript unit tests (Jest) |
| `npm run test:integration` | Playwright integration tests |

## Worked Example
First contribution workflow:

1. Read Zen of Wagtail, Django overview, and Wagtail Guide
2. Join Wagtail Slack (`#new-contributors`), set up GitHub profile
3. Complete the Wagtail getting started tutorial
4. Fork wagtail, clone, set up dev environment with `pip install -e ".[testing,docs]"` and `npm ci`
5. Verify setup by editing `wagtailadmin/home.html` template and `client/src/entrypoints/admin/wagtailadmin.js`
6. Find an issue (check `good-first-issue` label), read all comments and linked PRs
7. Create branch: `git checkout -b fix/1234-update-docs-dark-mode`
8. Make focused changes, write unit tests
9. Run `make lint` and `make format` before committing
10. Create PR with descriptive title, link to issue, screenshots if visual
11. Fix CI failures by pushing to same branch, never open new PRs

## Key Takeaways
1. Use `fnm install` for correct Node.js version, `pip install -e ".[testing,docs]"` for development mode, `npm ci && npm run build` for assets
2. Install pre-commit hooks with `pre-commit install` — they run automatically on every commit
3. Run `make lint` and `make format` before pushing to avoid CI failures from formatting issues
4. Never ask "please assign me" — just start working and add a comment describing your approach
5. Each PR should solve one problem with tests — use draft PRs for WIP work, push to same branch for CI fixes

## Connects To
- **Ch 18**: Testing (running Wagtail's test suite during development)
- **Ch 20**: Advanced topics (contributing translations via Transifex)
- **Ch 16**: Customization (understanding admin structure for UI contributions)
- **Ch 17**: Performance (testing performance-related changes)
