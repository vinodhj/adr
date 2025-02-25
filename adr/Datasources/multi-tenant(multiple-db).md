# Data Source Pattern for Multi-Tenant Applications (Multiple DBs)

## **Context**

In a multi-tenant architecture where each tenant has a separate database, the application needs to dynamically determine the correct database connection for each request. The approach should ensure proper tenant isolation while maintaining efficiency.

## **Decision**

For multi-tenant applications with multiple databases, we will use the **Factory Function Approach** to create data source instances dynamically based on the tenant's database.

## **Rationale**

- **Dynamic Database Selection:** Ensures each tenant is connected to the correct database instance.
- **Avoids Connection Overhead:** Maintains a pool of connections instead of keeping static instances.
- **Scalability:** Easily accommodates new tenants by initializing new database connections on demand.
- **Strict Tenant Isolation:** Prevents data leakage between tenants.

## **Implementation**

### **Factory Function for Data Sources**
```ts
const createUserDataSource = (db) => ({
  getUserById: async (id: string) => {
    return db.users.findUnique({ where: { id } });
  },
});
```

### **Context in GraphQL Server**
```typescript
context: async ({ request }) => {
  const tenantId = request.headers.get('X-Tenant-ID');
  const db = getDatabaseForTenant(tenantId); // Function to retrieve the correct DB connection
  return {
    datasources: {
      user: createUserDataSource(db),
    },
    tenantId,
  };
};
```

## **Alternatives Considered**

### 1. Constructor-Based Approach  
❌ **Rejected:** Requires maintaining multiple static instances for each tenant, increasing memory usage and complexity.

### 2. Global Database Connection Pool  
❌ **Avoided:** Potential cross-tenant access risks, which could lead to security vulnerabilities.

## **Consequences**

### ✅ Pros:
- Enables strict tenant isolation.
- Scalable and efficient.
- Dynamically adapts to new tenants.

### ❌ Cons:
- Slightly more complex implementation.
- Requires careful connection management.

---

## **Final Recommendation**
- ✅ **Use the Factory Function Approach** for multi-tenant applications with multiple databases.
- ⚠️ **Ensure Proper Connection Management** to prevent excessive database connections.

This ADR should be referenced when designing multi-tenant applications that require database isolation.

