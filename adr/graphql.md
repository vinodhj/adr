# ADR - Graphql API

**Context:** Modern application architectures require a more efficient and flexible approach to managing an API.

**Problem:**
1. **Over-fetching and Under-fetching:** Clients usually receive more data than requested (over-fetching) or must make multiple requests to fetch all required data(under-fetching).

**Considered Options:**
1. **Graphql API:** We considered this approach based on prior projects.

**Discarded Options:** N/A

**Decision:** We adopted the organization-specific decision to use GraphQL for all projects since we had previously thought it worked well with **microservices**. It has also performed effectively with **Cloudflare workers** and the rest of the stack. So, after careful consideration, we have decided to use **GraphQL** as our API query language.
1. GraphQL provides a powerful query language that allows clients to request exactly the data they need, reducing over-fetching and under-fetching issues that are common in traditional RESTful APIs.
2. GraphQL promotes a more loosely coupled architecture. This reduces the risk of breaking changes and enables frontend and backend teams to work more independently.
3. GraphQL allows us to combine many REST endpoints into a single GraphQL endpoint, making it easier for clients to interact with the server.
4. GraphQL provides an interactive playground for exploring and testing GraphQL queries and mutations.
5.  GraphQL schemas are strictly typed, which means that each piece of data is assigned a specific type. This ensures that data sent between the client and server has a predictable structure.

**Consequences:**
1. Team members will need to familiarize themselves with GraphQL.
2. An effective GraphQL schema requires careful consideration of data structure in a way that is appropriate for your application's requirements. This includes defining types, fields, and the relationships between them. We must invest time and effort in developing a well-structured schema that meets the requirements of our application and its users.
3. Implementing GraphQL might require changes to our backend infrastructure and data access layer to support GraphQL queries and mutations.