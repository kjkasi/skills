# Chapter 12: Contributing

## Core Idea
wagtailmenus welcomes contributions through GitHub; the project follows standard open-source practices for issues, pull requests, and releases.

## Frameworks Introduced
- **GitHub Contribution Workflow**: Standard fork → branch → PR workflow for open-source contributions
  - When to use: Contributing code, documentation, or bug reports
  - How: Fork repo, create feature branch, make changes, submit PR
- **Release Process**: Structured release cycle with version numbering and changelog management
  - When to use: Understanding release cadence and version compatibility
  - How: Follow semantic versioning; check CHANGES.rst for release notes

## Key Concepts
- **GitHub Repository**: Source code and issue tracking at github.com/jazzband/wagtailmenus
- **Jazzband**: Community organization hosting wagtailmenus and other Django packages
- **Semantic Versioning**: Major.Minor.Patch version numbering
- **CHANGES.rst**: Release notes file documenting changes in each version
- **Django Jazzband Policy**: Contribution guidelines and code of conduct

## Mental Models
- Contributing follows standard open-source patterns: fork, branch, PR, review
- Releases follow semantic versioning; check compatibility with Wagtail/Django versions
- Documentation contributions are welcome; docs use Sphinx/RST format
- Bug reports should include reproduction steps and environment details

## Anti-patterns
- **Skipping issue discussion**: For significant changes, open an issue first to discuss approach
- **Ignoring code style**: Follow existing code patterns; run linting before submitting
- **Forgetting tests**: Add tests for new functionality; maintain test coverage

## Code Examples
```bash
# Fork and clone the repository
git clone https://github.com/your-username/wagtailmenus.git
cd wagtailmenus

# Create a feature branch
git checkout -b feature/my-feature

# Install development dependencies
pip install -e .
pip install -r docs/requirements.txt

# Run tests
python -m django test wagtailmenus --settings=tests.settings

# Build documentation
cd docs
make html
```
- **What demonstrations**: Development setup and contribution workflow

```bash
# Submit a pull request
git add .
git commit -m "Add feature: description of changes"
git push origin feature/my-feature
# Then create PR on GitHub
```
- **What demonstrations**: Commit and PR workflow

## Reference Tables

| Task | Command/Location | Notes |
|------|------------------|-------|
| Report bug | GitHub Issues | Include reproduction steps |
| Suggest feature | GitHub Issues | Discuss approach first |
| Submit code | GitHub Pull Requests | Fork → branch → PR |
| Build docs | `cd docs && make html` | Sphinx/RST format |
| Run tests | `python -m django test wagtailmenus` | Uses Django test framework |
| Check releases | CHANGES.rst | Version history and notes |

## Worked Example
Contributing a new template tag:

1. Fork repository on GitHub
2. Clone and set up development environment:
```bash
git clone https://github.com/your-username/wagtailmenus.git
cd wagtailmenus
pip install -e .
```

3. Create feature branch:
```bash
git checkout -b feature/my-template-tag
```

4. Implement changes in wagtagmenus/templatetags/menu_tags.py

5. Add tests in tests/

6. Update documentation in docs/

7. Run tests:
```bash
python -m django test wagtailmenus --settings=tests.settings
```

8. Commit and push:
```bash
git add .
git commit -m "Add template_tag: my new tag"
git push origin feature/my-template-tag
```

9. Create pull request on GitHub with description of changes

## Key Takeaways
1. Contributions follow standard fork → branch → PR workflow
2. Open an issue first for significant changes to discuss approach
3. Include tests for new functionality; maintain test coverage
4. Documentation uses Sphinx/RST format; contributions welcome
5. Follow existing code style and patterns

## Connects To
- **Ch 1**: Overview — understanding wagtailmenus architecture
- **Ch 11**: Settings Reference — adding new configuration options
