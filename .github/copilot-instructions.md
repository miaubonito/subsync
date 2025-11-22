# Project Instructions

## Project Management
- **Tooling**: This project uses `uv` for dependency and project management.
- **Commands**: Always use `uv` commands (e.g., `uv add`, `uv remove`, `uv run`, `uv sync`) for managing dependencies and running scripts. Do not use `pip` or `poetry` commands directly.
- **Workflows**: Use `task` commands defined in `Taskfile.yml` for common operations like running the app, testing, linting, and formatting (e.g., `task run`, `task test`, `task lint`, `task format`).

## Python Version
- **Version**: Use Python version **>= 3.13** as defined in `pyproject.toml`.
- **Compatibility**: Ensure all generated code is compatible with Python 3.13+.
- **Features**: Utilize modern Python features available in 3.13 where appropriate (e.g., improved type hinting, performance improvements).

## Application Type
- **CLI App**: This is a Command Line Interface (CLI) application.
- **Goal**: The tool generates subtitles for videos optimized for YouTube.
