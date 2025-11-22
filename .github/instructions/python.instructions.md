---
applyTo: "**/*.py"
---
# Python Coding Guidelines

## Code Style
- **PEP 8**: Follow the PEP 8 style guide for Python code.
- **Type Hints**: Use Python's type hinting system for all function arguments and return values.
- **Docstrings**: Write clear docstrings for modules, classes, and functions.

## Error Handling
- **Exceptions**: Use specific exception handling rather than catching generic `Exception`.
- **Logging**: Use the standard `logging` module instead of `print` statements for debug and info messages (unless outputting to stdout for CLI user interaction).
