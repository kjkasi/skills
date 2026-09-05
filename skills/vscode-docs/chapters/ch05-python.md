# Chapter 5: Python Development

## Core Idea
VS Code provides first-class Python support through the Python extension, including linting, debugging, testing, Jupyter notebooks, and virtual environment management.

## Frameworks Introduced
- **Interpreter Selection**: Choose Python interpreter per workspace; supports virtual environments and conda
- **Test Discovery**: Automatic test discovery with pytest, unittest, or other frameworks
- **Debug Configurations**: Predefined and custom debug configs for different Python scenarios
- **Jupyter Integration**: Run notebooks inline or as separate processes; connect to remote kernels

## Key Concepts
- **Python Extension**: Microsoft's official extension; provides IntelliSense, debugging, testing, linting
- **Virtual Environments**: `venv`, `conda`, `pipenv` — VS Code auto-detects and offers to use them
- **Linting**: Pylint, flake8, pyflakes, mypy — configurable via settings
- **Formatting**: Black, autopep8, yapf — configurable as default formatter
- **Debugging**: breakpoint support, variable inspection, watch expressions, call stack
- **Jupyter Notebooks**: Interactive code execution with output display; supports markdown cells

## Mental Models
- Use **interpreter selection** when switching between Python versions or environments
- Use **test explorer** to run tests individually or in bulk; see coverage reports
- Use **Jupyter** for data exploration; export to `.py` for production code

## Code Examples
```json
// .vscode/settings.json (Python project)
{
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
  "python.linting.enabled": true,
  "python.linting.pylintEnabled": true,
  "python.formatting.provider": "black",
  "python.formatting.blackArgs": ["--line-length", "88"],
  "python.testing.pytestEnabled": true,
  "python.testing.pytestArgs": ["tests"]
}
```

```json
// .vscode/launch.json (Python debugging)
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: Current File",
      "type": "python",
      "request": "launch",
      "program": "${file}",
      "console": "integratedTerminal"
    },
    {
      "name": "Python: Attach",
      "type": "python",
      "request": "attach",
      "connect": { "host": "localhost", "port": 5678 }
    }
  ]
}
```

## Key Takeaways
1. Always set `python.defaultInterpreterPath` to your virtual environment
2. Enable type checking with mypy for better code quality
3. Use test explorer for visual test management and coverage
4. Jupyter notebooks are great for prototyping; convert to `.py` for production
5. Configure formatting on save for consistent code style

## Connects To
- **Ch 1**: Setting up Python interpreter and extensions
- **Ch 2**: Editing features work with Python IntelliSense
- **Ch 12**: Debugging Python in containers or remotely
- **Ch 13**: Data science workflows with Jupyter
