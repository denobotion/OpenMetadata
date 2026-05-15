# Python Checkstyle Skill

This skill enforces Python code style and quality standards for the OpenMetadata project.

## Overview

The Python checkstyle skill validates Python source files against the project's coding standards, including PEP 8 compliance, type hints, docstrings, and import ordering.

## Tools Used

- **black**: Code formatting
- **isort**: Import sorting
- **flake8**: Style and lint checks
- **mypy**: Static type checking
- **pylint**: Code analysis

## Rules

### Formatting
- All Python files must be formatted with `black` using default settings
- Line length maximum: **120 characters**
- Imports must be sorted using `isort` with `black` profile

### Type Hints
- All public functions and methods **must** include type annotations
- Return types must be explicitly declared
- Use `Optional[T]` instead of `T | None` for Python < 3.10 compatibility
- Avoid using `Any` unless absolutely necessary; document why if used

### Docstrings
- All public modules, classes, and functions must have docstrings
- Use Google-style docstrings
- Example:
  ```python
  def fetch_metadata(entity_id: str, include_deleted: bool = False) -> EntityMetadata:
      """Fetch metadata for a given entity.

      Args:
          entity_id: The unique identifier of the entity.
          include_deleted: Whether to include soft-deleted entities.

      Returns:
          EntityMetadata object containing the entity's metadata.

      Raises:
          EntityNotFoundException: If the entity does not exist.
      """
  ```

### Imports
- Standard library imports first
- Third-party imports second
- Local/project imports last
- Each group separated by a blank line
- No wildcard imports (`from module import *`)

### Naming Conventions
- **Classes**: `PascalCase`
- **Functions/Methods**: `snake_case`
- **Variables**: `snake_case`
- **Constants**: `UPPER_SNAKE_CASE`
- **Private members**: prefix with single underscore `_private`
- **Dunder methods**: `__dunder__`

### Error Handling
- Never use bare `except:` clauses; always specify exception type
- Log exceptions with appropriate context before re-raising
- Use custom exceptions from `openmetadata.exceptions` where applicable

### Testing
- Test files must be named `test_<module_name>.py`
- Test classes must be named `Test<ClassName>`
- Test methods must be named `test_<description>`
- All tests must have docstrings explaining what is being tested

## Running Checks Locally

```bash
# Format code
black ingestion/src/ ingestion/tests/
isort ingestion/src/ ingestion/tests/

# Lint checks
flake8 ingestion/src/ --max-line-length=120
pylint ingestion/src/

# Type checking
mypy ingestion/src/ --ignore-missing-imports
```

## CI Integration

All checks are run automatically on pull requests via GitHub Actions.
The workflow file is located at `.github/workflows/py-checkstyle.yml`.

A pull request **cannot be merged** if any of the following checks fail:
- `black --check`
- `isort --check-only`
- `flake8`
- `mypy`

## Suppressing Warnings

Use suppression comments sparingly and always include a justification:

```python
result = some_function()  # noqa: E501 - URL cannot be shortened
x: Any = dynamic_loader()  # type: ignore[assignment] - dynamic plugin system requires Any
```
