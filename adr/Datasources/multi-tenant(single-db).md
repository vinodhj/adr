# Data Source Pattern for Multi-Tenant Applications (Single DB)

## **Context**

In a multi-tenant architecture with a single database, tenant isolation must be enforced at the application level. Each tenant's data must be securely filtered based on their tenant ID.

## **Decision**

For multi-tenant applications sharing a single database, we will use the **constructor-based approach** with a `tenantId` parameter to enforce tenant-based filtering.

## **Rationale**

- **Single DB Connection:** Avoids unnecessary connection overhead.
- **Performance Optimization:** Reuses data source instances across requests.
- **Tenant Isolation:** Queries are filtered by `tenantId`.
- **Simplicity:** No need for dynamic factory logic.

## **Implementation**

```ts
class UserDataSource {
  private db;
  private tenantId;

  constructor(db, tenantId) {
    this.db = db;
    this.tenantId = tenantId;
  }

  async getUserById(id: string) {
    return this.db.users.findUnique({ where: { id, tenantId: this.tenantId } });
  }
}
```

### **Context in GraphQL Server**

```ts
context: async ({ request }) => {
  const tenantId = request.headers.get('X-Tenant-ID');
  const db = getDatabaseConnection();
  return {
    datasources: {
      user: new UserDataSource(db, tenantId),
    },
    tenantId,
  };
};
```

## **Alternatives Considered**

1. **Factory Function Approach:** Rejected as it adds unnecessary complexity.
2. **Separate Databases Per Tenant:** Not viable due to increased management overhead.

## **Consequences**

- **Pros:** Secure, performant, and easy to implement.
- **Cons:** Requires tenant-based query filtering for all operations.
