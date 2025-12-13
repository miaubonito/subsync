# SubSync - Copilot Instructions

## Project Overview

SubSync is a Python CLI application that generates Netflix-compliant subtitles for YouTube videos. It uses local AI models to transcribe audio and will support translation to other languages in the future.

## Tech Stack

- **Language**: Python 3.13+
- **Package Manager**: uv
- **Build System**: uv_build
- **Testing**: pytest
- **Linting/Formatting**: ruff
- **Task Runner**: Taskfile

### Key Dependencies (Planned)

- `yt-dlp` - YouTube audio extraction
- `openai-whisper` - Speech-to-text transcription

## Project Structure

```
src/subsync/          # Main package
  cli.py              # CLI entry point
docs/                 # Algorithm documentation
  ALGORITHM.md        # Core algorithm overview
  SUBTITLE_GENERATION_ALGORITHM.md  # Detailed implementation spec
```

## Development Workflow

Use Taskfile commands for common operations:

- `task install` - Install dependencies
- `task run` - Run the application (`uv run subsync`)
- `task test` - Run tests (`uv run pytest`)
- `task lint` - Lint code (`uv run ruff check .`)
- `task format` - Format code (`uv run ruff format .`)

For any other operations, use `uv` directly:

- **Never use `pip`** - Always use `uv add` for dependencies, `uv run` for execution
- Add dependencies: `uv add <package>`
- Add dev dependencies: `uv add --group dev <package>`
- Run scripts: `uv run <command>`

## Coding Conventions

### Python Style

- Favor pragmatic, readable code over clever abstractions - simplicity wins
- Follow PEP 8 style guide
- Use type hints for all function arguments and return values
- Write docstrings for modules, classes, and functions
- Use specific exceptions, not generic `Exception`
- Use `logging` module for debug/info messages (not `print`, unless CLI user output)

### CLI Design

- Entry point is `subsync.cli:main`
- Design for future commands: transcribe, translate
- Follow Unix CLI conventions (clear exit codes, stderr for errors)

### Architecture Patterns

- Modular design with clear separation of concerns:
  - URL Handler → Audio Extractor → Transcriber → Subtitle Processor → Writer
- Subtitle output follows Netflix Timed Text Style Guide standards
- Support SRT (primary) and VTT formats

## Testing

- Place tests in the project root or `tests/` directory
- Use pytest conventions
- Write tests when adding logic, especially to cover edge cases
- Prefer unit tests for pure functions
- Keep tests deterministic - avoid network calls unless explicitly requested and properly isolated
