# Markdown Checkstyle Skill

This skill enforces consistent Markdown formatting and style conventions across the OpenMetadata project documentation.

## Rules

### Structure
- Every Markdown file must begin with a single `#` H1 heading
- Headings must follow a logical hierarchy (no skipping levels, e.g., H1 → H3 without H2)
- Files longer than 100 lines should include a Table of Contents section
- A blank line must exist before and after every heading
- A blank line must exist before and after every code block

### Formatting
- Use ATX-style headings (`#`, `##`, etc.), not Setext-style (underline with `===` or `---`)
- Use hyphens (`-`) for unordered lists, not asterisks (`*`) or plus signs (`+`)
- Use fenced code blocks (triple backticks) rather than indented code blocks
- Always specify a language identifier on fenced code blocks when the language is known
- Inline code must use single backticks, not escaped HTML
- Bold text uses `**double asterisks**`, not `__double underscores__`
- Italic text uses `*single asterisk*`, not `_single underscore_`
- No trailing whitespace on any line
- Files must end with a single newline character

### Links
- Prefer reference-style links for URLs used more than once
- All relative links must resolve to existing files within the repository
- Bare URLs must be wrapped in angle brackets: `<https://example.com>`
- Link text must be descriptive — avoid "click here" or "read more"

### Code Blocks

Bad:
```
some code without language tag
```

Good:
```typescript
const greeting: string = 'Hello, OpenMetadata!';
console.log(greeting);
```

### Lists

Bad:
* Item one
* Item two
+ Item three

Good:
- Item one
- Item two
- Item three

### Headings Hierarchy

Bad:
```markdown
# Title
### Skipped Level
```

Good:
```markdown
# Title
## Section
### Subsection
```

## Automated Checks

The following tools are used to enforce these rules:

- **markdownlint** (`markdownlint-cli2`) — primary linting engine
- **remark-lint** — secondary prose and link validation

### Running Locally

```bash
# Install dependencies
npm install -g markdownlint-cli2

# Lint all Markdown files
markdownlint-cli2 "**/*.md" "#node_modules"

# Lint a specific file
markdownlint-cli2 path/to/file.md

# Auto-fix fixable issues
markdownlint-cli2 --fix "**/*.md" "#node_modules"
```

### Configuration

The project's `.markdownlint.json` at the repository root defines rule overrides:

```json
{
  "default": true,
  "MD013": { "line_length": 120 },
  "MD033": false,
  "MD041": true
}
```

## Integration

This skill is applied automatically during:

1. **Pull Request checks** — markdownlint runs as a GitHub Actions step on any PR that modifies `.md` files
2. **Pre-commit hooks** — developers with `pre-commit` configured locally will have Markdown linted before each commit
3. **Agent reviews** — AI agents reviewing documentation PRs must apply these rules before approving

## Examples of Compliant Files

- `README.md` — project root readme
- `CONTRIBUTING.md` — contributor guidelines
- `docs/` — all files under the docs directory
- `.agents/skills/*/SKILL.md` — skill definition files (like this one)
