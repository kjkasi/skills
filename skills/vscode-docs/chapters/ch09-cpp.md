# Chapter 9: C++ Development

## Core Idea
VS Code provides C++ development through the C/C++ extension, including IntelliSense, debugging, and cross-platform build support with CMake.

## Frameworks Introduced
- **C/C++ Extension**: Microsoft's extension for C/C++ development with IntelliSense and debugging
- **IntelliSense Modes**: Tag Parser, Microsoft, and Browse modes for different use cases
- **CMake Integration**: Build and debug CMake projects
- **Cross-compilation**: Configure IntelliSense for remote targets

## Key Concepts
- **IntelliSense Modes**: Tag Parser (fast, basic), Microsoft (full), Browse (slow, accurate)
- **Debugging**: GDB/LLDB integration for debugging C/C++ applications
- **CMake**: Cross-platform build system integration
- **Tasks**: Build tasks for compile, link, and other build steps
- **Configurations**: Multiple IntelliSense configurations for different build targets

## Mental Models
- Use **IntelliSense Mode: Tag Parser** for quick navigation in large codebases
- Use **IntelliSense Mode: Microsoft** for accurate completions during development
- Use **CMake** for cross-platform build configuration

## Code Examples
```json
// .vscode/c_cpp_properties.json
{
  "configurations": [
    {
      "name": "Linux",
      "includePath": ["${workspaceFolder}/**"],
      "defines": [],
      "compilerPath": "/usr/bin/gcc",
      "cStandard": "c17",
      "cppStandard": "c++17",
      "intelliSenseMode": "linux-gcc-x64"
    }
  ],
  "version": 4
}
```

## Key Takeaways
1. Choose Intellisense mode based on your needs: speed vs. accuracy
2. Configure `c_cpp_properties.json` for custom include paths and defines
3. Use CMake Tools extension for CMake project management
4. Debug configurations support GDB and LLDB on Linux/macOS
5. Use tasks.json for custom build commands beyond CMake

## Connects To
- **Ch 1**: Setting up C++ development environment
- **Ch 2**: Code editing and navigation
- **Ch 12**: Debugging C/C++ applications
- **Ch 11**: Cross-compilation and remote development
