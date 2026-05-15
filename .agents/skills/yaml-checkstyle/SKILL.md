# YAML Checkstyle Skill

This skill enforces YAML formatting and style conventions for the OpenMetadata project.

## Trigger

Use this skill when:
- Reviewing or writing YAML configuration files
- Modifying CI/CD pipeline definitions (`.github/workflows/*.yml`)
- Editing Docker Compose files (`docker-compose*.yml`)
- Updating Kubernetes manifests (`*.yaml`)
- Changing OpenAPI/Swagger specs (`openapi.yaml`)
- Any file with `.yml` or `.yaml` extension

## Rules

### Indentation
- Use **2 spaces** for indentation (never tabs)
- Nested mappings and sequences must be consistently indented
- List items (`-`) should align with their parent key's value column

```yaml
# Good
services:
  openmetadata:
    image: openmetadata/server:latest
    ports:
      - "8585:8585"
    environment:
      - DB_HOST=mysql

# Bad
services:
    openmetadata:
        image: openmetadata/server:latest
```

### Quoting
- Use **double quotes** for strings containing special characters (`:`, `#`, `{`, `}`, `[`, `]`)
- Use **no quotes** for plain alphanumeric strings and booleans
- Use **single quotes** only when the string contains double quotes
- Version numbers and port mappings must be quoted

```yaml
# Good
version: "3.8"
name: openmetadata
enabled: true
message: 'He said "hello"'

# Bad
version: 3.8
enabled: "true"
```

### Spacing
- Always add a single space after `:` in key-value pairs
- Always add a single space after `-` in list items
- No trailing whitespace on any line
- Files must end with a single newline character

### Comments
- Comments must start with `# ` (hash followed by a single space)
- Inline comments must have at least two spaces before the `#`
- Use comments to explain non-obvious configuration choices

```yaml
# Good
# Database configuration for OpenMetadata
db:
  host: localhost  # Override with DB_HOST env var

# Bad
#Database configuration
db:
  host: localhost #Override with DB_HOST env var
```

### Keys
- Use **camelCase** for application config keys
- Use **kebab-case** for Kubernetes resource names and labels
- Use **UPPER_SNAKE_CASE** for environment variable names
- No duplicate keys within the same mapping level

### Booleans and Nulls
- Use `true`/`false` (lowercase) for booleans — never `yes`/`no`/`on`/`off`
- Use `null` or `~` for null values — never leave values empty

```yaml
# Good
enabled: true
optionalField: null

# Bad
enabled: yes
optionalField:
```

### Multi-line Strings
- Use `|` (literal block scalar) for multi-line strings that preserve newlines
- Use `>` (folded block scalar) for long single-paragraph strings
- Avoid embedding newlines with `\n` in quoted strings

```yaml
# Good
script: |
  echo "Starting OpenMetadata"
  ./bin/openmetadata-server.sh start

description: >
  OpenMetadata is the all-in-one platform for data discovery,
  lineage, data quality, observability, and governance.
```

### Document Structure
- GitHub Actions workflows must include `name:` at the top level
- Docker Compose files must include `version:` as the first key
- Kubernetes manifests must include `apiVersion:`, `kind:`, `metadata:`, and `spec:`
- OpenAPI specs must include `openapi:` version and `info:` block

## Automated Checks

The following tools are used for YAML validation:

```bash
# Lint YAML files
yamllint -c .yamllint.yml <file>

# Validate Kubernetes manifests
kubectl apply --dry-run=client -f <manifest.yaml>

# Validate GitHub Actions workflows
actionlint .github/workflows/*.yml
```

## Common Mistakes to Avoid

1. **Anchor/alias overuse** — only use `&anchor` and `*alias` when duplication is significant
2. **Implicit type coercion** — quote values like `1.0`, `on`, `null` if they should be strings
3. **Mixed indentation** — never mix 2-space and 4-space indentation in the same file
4. **Missing quotes on port mappings** — `"8585:8585"` not `8585:8585`
