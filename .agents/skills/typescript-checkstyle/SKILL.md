# TypeScript Checkstyle Skill

This skill performs TypeScript/JavaScript code style checks for the OpenMetadata UI codebase.

## Overview

Enforces consistent code style, best practices, and quality standards across TypeScript files in the OpenMetadata frontend.

## Tools Available

### `fetch_metadata`
Fetches project metadata and ESLint/Prettier configuration from the repository.

**Parameters:**
- `file_path` (string): Path to the TypeScript file to analyze
- `config_path` (string, optional): Path to ESLint config file (defaults to `.eslintrc`)

**Returns:** Object containing file content and applicable style rules

## Checks Performed

### 1. Import Organization
- External imports must come before internal imports
- Imports should be alphabetically sorted within groups
- No unused imports allowed
- Use named imports over default imports where possible

**Bad:**
```typescript
import { useState } from 'react';
import { MyComponent } from './components/MyComponent';
import axios from 'axios';
import { useEffect } from 'react';
```

**Good:**
```typescript
import axios from 'axios';
import { useEffect, useState } from 'react';

import { MyComponent } from './components/MyComponent';
```

### 2. Type Annotations
- All function parameters must have explicit type annotations
- Return types must be explicitly declared for exported functions
- Avoid using `any` type; use `unknown` or proper types instead
- Prefer `interface` over `type` for object shapes

**Bad:**
```typescript
export const fetchData = async (id) => {
  const result: any = await api.get(id);
  return result;
};
```

**Good:**
```typescript
export const fetchData = async (id: string): Promise<EntityData> => {
  const result: EntityData = await api.get<EntityData>(id);
  return result;
};
```

### 3. React Component Standards
- Functional components must use arrow function syntax
- Props interfaces must be named `<ComponentName>Props`
- Components must be exported as named exports (not default)
- Use `React.FC` sparingly; prefer explicit return types

**Bad:**
```typescript
function MyComponent(props) {
  return <div>{props.name}</div>;
}
export default MyComponent;
```

**Good:**
```typescript
interface MyComponentProps {
  name: string;
  onClick?: () => void;
}

export const MyComponent = ({ name, onClick }: MyComponentProps): JSX.Element => {
  return <div onClick={onClick}>{name}</div>;
};
```

### 4. Naming Conventions
- Variables and functions: `camelCase`
- Interfaces and Types: `PascalCase`
- Constants: `UPPER_SNAKE_CASE` for module-level constants
- React components: `PascalCase`
- Files containing components: `PascalCase.tsx`
- Utility files: `camelCase.ts`

### 5. Error Handling
- All async operations must have try/catch blocks or `.catch()` handlers
- Errors must be logged using the project's logger utility, not `console.error`
- Never swallow errors silently

**Bad:**
```typescript
try {
  await riskyOperation();
} catch (e) {
  // do nothing
}
```

**Good:**
```typescript
try {
  await riskyOperation();
} catch (error: unknown) {
  showErrorToast(error as AxiosError);
  Logger.error('Failed to perform operation', error);
}
```

### 6. Hook Rules
- Custom hooks must be prefixed with `use`
- Hooks must not be called conditionally
- Dependencies arrays in `useEffect`/`useCallback`/`useMemo` must be complete

## Configuration Files

This skill reads from:
- `.eslintrc` or `.eslintrc.js` — ESLint rules
- `.prettierrc` — Prettier formatting rules
- `tsconfig.json` — TypeScript compiler options

## Usage

When reviewing a TypeScript file, apply all checks above and report violations with:
1. Line number
2. Rule violated
3. Description of the issue
4. Suggested fix
