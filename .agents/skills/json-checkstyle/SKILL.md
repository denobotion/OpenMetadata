# JSON Checkstyle Skill

This skill enforces JSON formatting and style conventions for the OpenMetadata project.

## Rules

### Formatting
- Use 2-space indentation (no tabs)
- Keys must be double-quoted strings
- No trailing commas
- No comments (JSON spec does not support them)
- Files must end with a single newline character
- Maximum line length of 120 characters
- Arrays and objects with more than 2 items should be formatted vertically (one item per line)

### Naming Conventions
- Object keys must use `camelCase` for application config files
- Object keys must use `snake_case` for API schema/OpenAPI files
- Object keys must use `kebab-case` for package.json-style manifests
- Avoid abbreviations unless they are well-known (e.g., `id`, `url`, `api`)

### Structure
- Top-level keys in config files should be ordered alphabetically
- Required fields should appear before optional fields in schema definitions
- `$schema` and `$id` fields, when present, must appear first

### Values
- Boolean values must be `true`/`false` (not `"true"`/`"false"` strings)
- Numeric values must not be quoted
- Null values must use `null` (not empty string `""`)
- Avoid deeply nested structures (max depth: 6 levels)

## Examples

### ✅ Correct

```json
{
  "apiVersion": "v1",
  "enabled": true,
  "maxRetries": 3,
  "metadata": {
    "description": "OpenMetadata service configuration",
    "name": "openmetadata",
    "tags": [
      "data",
      "metadata",
      "catalog"
    ]
  },
  "timeout": null
}
```

### ❌ Incorrect

```json
{
    "apiVersion": "v1",  // version field
    "enabled": "true",
    "maxRetries": "3",
    "metadata": {"description": "OpenMetadata service configuration", "name": "openmetadata", "tags": ["data", "metadata", "catalog"]},
    "timeout": "",
}
```

Problems:
- 4-space indentation instead of 2-space
- Inline comment (not valid JSON)
- Boolean `enabled` stored as a string `"true"`
- Numeric `maxRetries` stored as a string `"3"`
- `metadata` object not expanded vertically
- Empty string used instead of `null` for `timeout`
- Trailing comma after last property

## Automated Checks

Run the following to validate JSON files:

```bash
# Validate and pretty-print a single file
npx prettier --check --parser json path/to/file.json

# Auto-fix formatting issues
npx prettier --write --parser json path/to/file.json

# Validate all JSON files in the project
npx prettier --check --parser json "**/*.json" --ignore-path .prettierignore

# Schema validation (requires ajv-cli)
npx ajv validate -s schema.json -d data.json
```

## Prettier Configuration

Ensure your `.prettierrc` or `prettier.config.js` includes:

```json
{
  "printWidth": 120,
  "tabWidth": 2,
  "useTabs": false,
  "trailingComma": "none"
}
```

## Exceptions

- `package-lock.json` and `yarn.lock` are auto-generated and exempt from manual style enforcement
- Minified JSON files (`*.min.json`) are exempt from formatting rules
- Files under `node_modules/` are excluded from all checks
