# API Checkstyle Skill

This skill enforces consistent API design patterns and OpenAPI/REST conventions for the OpenMetadata project.

## Trigger

Use this skill when:
- Reviewing or writing REST API endpoint definitions
- Working with OpenAPI/Swagger specifications
- Defining API request/response schemas
- Writing API client code in TypeScript

## Rules

### 1. HTTP Method Conventions

- Use `GET` for read-only operations
- Use `POST` for creating resources
- Use `PUT` for full resource replacement
- Use `PATCH` for partial resource updates
- Use `DELETE` for resource removal
- Never use `GET` with a request body

### 2. URL Path Conventions

- Use **kebab-case** for URL path segments: `/api/v1/table-profiles` not `/api/v1/tableProfiles`
- Use **plural nouns** for collection endpoints: `/api/v1/tables` not `/api/v1/table`
- Use resource IDs in path for specific resources: `/api/v1/tables/{id}`
- Nest related resources: `/api/v1/tables/{id}/columns`
- Version all APIs under `/api/v1/`

### 3. Response Status Codes

| Operation | Success Code |
|-----------|-------------|
| GET (single) | 200 OK |
| GET (list) | 200 OK |
| POST (create) | 201 Created |
| PUT/PATCH | 200 OK |
| DELETE | 200 OK or 204 No Content |

Error codes:
- `400 Bad Request` — invalid input
- `401 Unauthorized` — missing/invalid auth
- `403 Forbidden` — insufficient permissions
- `404 Not Found` — resource does not exist
- `409 Conflict` — duplicate resource
- `422 Unprocessable Entity` — validation failure
- `500 Internal Server Error` — unexpected server error

### 4. Request/Response Schema

- Always define `Content-Type: application/json`
- Response bodies must be typed objects, never raw primitives
- List responses must use the paginated envelope:

```json
{
  "data": [...],
  "paging": {
    "total": 100,
    "before": "<cursor>",
    "after": "<cursor>"
  }
}
```

- Error responses must follow:

```json
{
  "code": 404,
  "message": "Table not found",
  "details": "No table with id '...' exists"
}
```

### 5. TypeScript API Client Conventions

- Use `async/await` — no raw `.then()` chains
- Always handle and type errors explicitly
- Use generated types from OpenAPI spec where available
- Centralize base URL and auth headers in a shared `apiClient` instance

**Good:**
```typescript
export const getTableById = async (id: string): Promise<Table> => {
  const response = await apiClient.get<Table>(`/tables/${id}`);
  return response.data;
};
```

**Bad:**
```typescript
export const getTableById = (id) => {
  return fetch(`/api/v1/tables/${id}`).then(res => res.json());
};
```

### 6. Query Parameter Naming

- Use **camelCase** for query parameters: `?pageSize=10&afterCursor=abc`
- Standard pagination params: `limit`, `offset` or cursor-based `before`/`after`
- Use `fields` for sparse fieldsets: `?fields=id,name,owner`
- Use `include` for related resources: `?include=tags,followers`

### 7. Authentication Headers

- Always pass auth token via `Authorization: Bearer <token>`
- Never pass credentials in URL query parameters
- Bot tokens use the same header format

### 8. Deprecation

- Mark deprecated endpoints with `x-deprecated: true` in OpenAPI spec
- Add `Deprecation` response header with date
- Provide migration path in endpoint description

## Checklist

Before submitting API-related code:

- [ ] Endpoint paths use kebab-case and plural nouns
- [ ] Correct HTTP method used for the operation
- [ ] Response uses correct status code
- [ ] Error responses follow the standard error schema
- [ ] List endpoints return paginated envelope
- [ ] TypeScript client uses `async/await`
- [ ] Auth is passed via `Authorization` header
- [ ] No credentials in URLs or logs
- [ ] OpenAPI spec updated if adding/modifying endpoints
