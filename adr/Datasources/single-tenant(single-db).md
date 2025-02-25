# Data Source Pattern for Single-Tenant Applications

## **Context**

In a single-tenant architecture, all users share the same database and configurations. The goal is to ensure efficient data access, maintainability, and performance while keeping the system simple and scalable.

## **Decision**

For single-tenant applications, we will use the **constructor-based approach** for managing data sources.

## **Rationale**

- **Single DB Connection:** A single instance of the data source is created and reused.
- **Performance Optimization:** No need to create a new data source for every request.
- **Easier Dependency Management:** Works well with dependency injection frameworks.
- **Simple Implementation:** No need for dynamic tenant-based initialization.

## **Implementation**

```ts
class UserDataSource {
  private db;
  
  constructor(db) {
    this.db = db;
  }

  async getUserById(id: string) {
    return this.db.users.findUnique({ where: { id } });
  }
}
```

### **Context in GraphQL Server**

```ts
context: async () => {
  const db = getDatabaseConnection();
  return {
    datasources: {
      user: new UserDataSource(db),
    },
  };
};
```

## **Alternatives Considered**

1. **Factory Function Approach:** Rejected because it creates unnecessary instances per request.
2. **Global Static Instance:** Avoided due to lack of flexibility for dependency injection.

## **Consequences**

- **Pros:** Simpler, more efficient, and easier to maintain.
- **Cons:** Less flexibility if tenant-specific logic is needed in the future.
