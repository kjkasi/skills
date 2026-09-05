# Chapter 15: Data Science

## Core Idea
VS Code supports data science workflows through Jupyter notebooks, Python interactive development, and data visualization tools.

## Frameworks Introduced
- **Jupyter Notebooks**: Interactive code execution with output display and markdown cells
- **Python Interactive**: Run Python code in interactive windows
- **Data Wrangler**: Explore and clean tabular data
- **Notebook Renderer**: Rich output display for notebooks

## Key Concepts
- **Jupyter Kernels**: Execute code in different Python environments
- **Notebook Cells**: Code and markdown blocks executed independently
- **Kernel Management**: Switch between kernels, install new ones
- **Variable Explorer**: Inspect variables during notebook execution
- **Data Wrangler**: GUI for data cleaning and transformation

## Mental Models
- Use **Jupyter notebooks** for data exploration and presentation
- Use **Python Interactive** for quick code execution without full notebook
- Use **Data Wrangler** for visual data cleaning

## Code Examples
```json
// .vscode/settings.json (Jupyter configuration)
{
  "jupyter.askForKernelRestart": false,
  "jupyter.interactiveWindow.textEditor.executeSelection": true,
  "notebook.formatOnSave.enabled": true
}
```

## Key Takeaways
1. Jupyter notebooks are great for data exploration and presentation
2. Kernel selection affects available packages and Python version
3. Variable Explorer shows notebook state during execution
4. Data Wrangler provides GUI for data cleaning operations
5. Export notebooks to `.py` for production code

## Connects To
- **Ch 5**: Python development basics
- **Ch 2**: Editing features in notebooks
- **Ch 3**: AI assistance in notebooks
- **Ch 12**: Debugging notebook code
