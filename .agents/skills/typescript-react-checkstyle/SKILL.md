# TypeScript React Checkstyle Skill

This skill enforces TypeScript React coding standards and best practices for the OpenMetadata UI codebase.

## Rules

### Component Structure

1. **Functional Components Only**: Use functional components with hooks. No class components.
2. **Component File Naming**: Use PascalCase for component files (e.g., `UserProfile.tsx`).
3. **One Component Per File**: Each file should export a single primary component.
4. **Props Interface**: Always define a typed `Props` interface or type alias for component props.

```tsx
// ✅ Good
interface UserCardProps {
  userId: string;
  displayName: string;
  avatarUrl?: string;
  onSelect?: (userId: string) => void;
}

const UserCard: React.FC<UserCardProps> = ({ userId, displayName, avatarUrl, onSelect }) => {
  return (
    <div className="user-card" onClick={() => onSelect?.(userId)}>
      {avatarUrl && <img src={avatarUrl} alt={displayName} />}
      <span>{displayName}</span>
    </div>
  );
};

export default UserCard;

// ❌ Bad
const UserCard = (props: any) => {
  return <div>{props.name}</div>;
};
```

### Hooks Usage

5. **Custom Hooks Prefix**: Custom hooks must start with `use` (e.g., `useEntityDetails`).
6. **Dependency Arrays**: Always provide complete dependency arrays for `useEffect`, `useMemo`, and `useCallback`.
7. **Avoid Inline Functions in JSX**: Extract handlers using `useCallback` to prevent unnecessary re-renders.

```tsx
// ✅ Good
const handleClick = useCallback(() => {
  onSelect(entityId);
}, [entityId, onSelect]);

return <Button onClick={handleClick}>Select</Button>;

// ❌ Bad
return <Button onClick={() => onSelect(entityId)}>Select</Button>;
```

### Imports

8. **Import Order**: Follow this order:
   - React and React-related imports
   - Third-party libraries (antd, lodash, etc.)
   - Internal absolute imports (`components/`, `utils/`, `constants/`)
   - Relative imports (`./`, `../`)
   - Type-only imports (use `import type`)
   - Style imports (`.less`, `.css`)

```tsx
// ✅ Good
import React, { useCallback, useState } from 'react';
import { Button, Typography } from 'antd';
import { isEmpty } from 'lodash';
import { EntityType } from 'enums/entity.enum';
import { getEntityName } from 'utils/EntityUtils';
import EntityLink from '../EntityLink/EntityLink';
import type { EntityReference } from 'generated/type/entityReference';
import './EntityCard.less';
```

### State Management

9. **Avoid Prop Drilling**: Use context or state management for deeply nested props (more than 2 levels).
10. **State Initialization**: Initialize state with explicit types when inference is insufficient.

```tsx
// ✅ Good
const [selectedEntities, setSelectedEntities] = useState<EntityReference[]>([]);

// ❌ Bad
const [data, setData] = useState(null);
```

### Async Operations

11. **Loading and Error States**: Always handle loading and error states for async operations.
12. **Cleanup Effects**: Cancel or ignore async operations in `useEffect` cleanup functions.

```tsx
// ✅ Good
const fetchEntityDetails = useCallback(async () => {
  setIsLoading(true);
  try {
    const data = await getEntityDetails(entityId);
    setEntityDetails(data);
  } catch (error) {
    showErrorToast(error as AxiosError);
  } finally {
    setIsLoading(false);
  }
}, [entityId]);

useEffect(() => {
  let isMounted = true;
  fetchEntityDetails().then(() => {
    if (!isMounted) return;
  });
  return () => {
    isMounted = false;
  };
}, [fetchEntityDetails]);
```

### Accessibility

13. **ARIA Attributes**: Add appropriate `aria-label`, `aria-describedby`, and `role` attributes to interactive elements.
14. **Alt Text**: Always provide `alt` text for images.
15. **Keyboard Navigation**: Ensure all interactive elements are keyboard accessible.

### Testing

16. **Test File Co-location**: Place test files alongside components (e.g., `UserCard.test.tsx`).
17. **Test IDs**: Use `data-testid` attributes for test selectors, not class names or text content.

```tsx
// ✅ Good
<Button data-testid="save-entity-button" onClick={handleSave}>
  Save
</Button>
```

## Example: Compliant Component

```tsx
import React, { useCallback, useEffect, useState } from 'react';
import { Card, Skeleton, Typography } from 'antd';
import { AxiosError } from 'axios';
import { EntityType } from 'enums/entity.enum';
import { showErrorToast } from 'utils/ToastUtils';
import { getEntitySummary } from 'rest/entityAPI';
import type { EntitySummary } from 'generated/type/entitySummary';
import './EntitySummaryCard.less';

const { Text, Title } = Typography;

interface EntitySummaryCardProps {
  entityFqn: string;
  entityType: EntityType;
  onEntityClick?: (fqn: string) => void;
}

const EntitySummaryCard: React.FC<EntitySummaryCardProps> = ({
  entityFqn,
  entityType,
  onEntityClick,
}) => {
  const [summary, setSummary] = useState<EntitySummary | undefined>();
  const [isLoading, setIsLoading] = useState<boolean>(false);

  const fetchSummary = useCallback(async () => {
    setIsLoading(true);
    try {
      const data = await getEntitySummary(entityFqn, entityType);
      setSummary(data);
    } catch (error) {
      showErrorToast(error as AxiosError);
    } finally {
      setIsLoading(false);
    }
  }, [entityFqn, entityType]);

  useEffect(() => {
    fetchSummary();
  }, [fetchSummary]);

  const handleCardClick = useCallback(() => {
    onEntityClick?.(entityFqn);
  }, [entityFqn, onEntityClick]);

  if (isLoading) {
    return <Skeleton active paragraph={{ rows: 3 }} />;
  }

  return (
    <Card
      aria-label={`Entity summary for ${entityFqn}`}
      className="entity-summary-card"
      data-testid="entity-summary-card"
      hoverable={Boolean(onEntityClick)}
      onClick={handleCardClick}>
      <Title level={5}>{summary?.displayName ?? entityFqn}</Title>
      {summary?.description && (
        <Text className="entity-description" type="secondary">
          {summary.description}
        </Text>
      )}
    </Card>
  );
};

export default EntitySummaryCard;
```
