# Bash Checkstyle Skill

This skill enforces Bash/Shell script coding standards and best practices for the OpenMetadata project.

## Trigger

Use this skill when reviewing or writing Bash/Shell scripts (`.sh`, `.bash` files).

## Rules

### 1. Shebang Line
- Always start scripts with `#!/usr/bin/env bash` (preferred over `#!/bin/bash` for portability)
- For POSIX-compatible scripts, use `#!/bin/sh`

### 2. Script Header
- Include a comment block describing the script's purpose
- Document required environment variables
- Document expected arguments

```bash
#!/usr/bin/env bash
# Description: Brief description of what this script does
# Usage: ./script.sh [OPTIONS] <arg1> <arg2>
# Arguments:
#   arg1 - Description of arg1
# Environment Variables:
#   ENV_VAR - Description of ENV_VAR
```

### 3. Strict Mode
- Always enable strict mode at the top of scripts:

```bash
set -euo pipefail
```

- `-e`: Exit immediately on error
- `-u`: Treat unset variables as errors
- `-o pipefail`: Return exit code of first failed command in pipeline

### 4. Variable Naming
- Use `UPPER_CASE` for environment variables and constants
- Use `lower_case` for local variables
- Always quote variables: `"${variable}"` not `$variable`
- Use `local` keyword for function-scoped variables

```bash
# Good
local file_path="${1}"
REQUIRED_VERSION="1.0.0"

# Bad
filePath=$1
required_version=1.0.0
```

### 5. Functions
- Use `function_name()` syntax (no `function` keyword)
- Place all functions before main logic
- Document functions with comments

```bash
# Prints an error message and exits with code 1
# Arguments:
#   $1 - Error message
error_exit() {
  echo "ERROR: ${1}" >&2
  exit 1
}
```

### 6. Error Handling
- Check return codes for critical operations
- Use meaningful error messages directed to stderr
- Implement cleanup traps for temporary files

```bash
# Cleanup on exit
cleanup() {
  rm -f "${TMP_FILE:-}"
}
trap cleanup EXIT

# Error with context
command || error_exit "Failed to execute command"
```

### 7. Conditionals
- Use `[[ ]]` for conditionals (not `[ ]` or `test`)
- Use `-z` and `-n` for string emptiness checks
- Quote all variable expansions in conditionals

```bash
# Good
if [[ -z "${MY_VAR}" ]]; then
  echo "Variable is empty"
fi

# Bad
if [ $MY_VAR == "" ]; then
  echo "Variable is empty"
fi
```

### 8. Command Substitution
- Use `$()` instead of backticks

```bash
# Good
current_date=$(date +%Y-%m-%d)

# Bad
current_date=`date +%Y-%m-%d`
```

### 9. Logging
- Use consistent logging functions
- Include timestamps in log output
- Direct errors to stderr

```bash
log_info() {
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] INFO: ${*}"
}

log_error() {
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] ERROR: ${*}" >&2
}
```

### 10. Script Example

```bash
#!/usr/bin/env bash
# Description: Example OpenMetadata deployment helper script
# Usage: ./deploy.sh [--env <environment>] [--version <version>]
# Environment Variables:
#   OM_HOST - OpenMetadata server host (default: localhost)

set -euo pipefail

# Constants
DEFAULT_HOST="localhost"
DEFAULT_PORT="8585"

# Logging helpers
log_info() {
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] INFO: ${*}"
}

log_error() {
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] ERROR: ${*}" >&2
}

error_exit() {
  log_error "${1}"
  exit 1
}

# Cleanup temporary resources
cleanup() {
  [[ -f "${TMP_CONFIG:-}" ]] && rm -f "${TMP_CONFIG}"
}
trap cleanup EXIT

# Check required dependencies
check_dependencies() {
  local deps=("curl" "jq" "docker")
  for dep in "${deps[@]}"; do
    command -v "${dep}" &>/dev/null || error_exit "Required dependency '${dep}' not found"
  done
}

main() {
  local environment="dev"
  local version="latest"
  local host="${OM_HOST:-${DEFAULT_HOST}}"

  # Parse arguments
  while [[ $# -gt 0 ]]; do
    case "${1}" in
      --env) environment="${2}"; shift 2 ;;
      --version) version="${2}"; shift 2 ;;
      *) error_exit "Unknown argument: ${1}" ;;
    esac
  done

  check_dependencies
  log_info "Deploying OpenMetadata version=${version} env=${environment} host=${host}"
}

main "${@}"
```

## ShellCheck

All shell scripts must pass [ShellCheck](https://www.shellcheck.net/) with no warnings or errors:

```bash
shellcheck --severity=warning script.sh
```
