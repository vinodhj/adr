# ADR: Gateway

**Context:** We need a gateway solution to provide seamless communication & integration between the different microservices.

**Problem:**
1. We will have many microservices which can function independently but the client should be able to communicate through the gateway only.
2. We need to provide a way to integrate the various features of each microservice.
3. The gateway should be compatible with the Cloudflare Workers.
4. We need a solution which will handle the authentication and pass valid inputs required by the microservices.

**Considered Options:**
1. **[GraphQL Mesh:](https://the-guild.dev/graphql/mesh/docs)** GraphQL Mesh is a framework to build GraphQL Gateways on top of GraphQL and non-GraphQL services (REST, gRPC, and more).
2. **[Schema Stitching:](https://the-guild.dev/graphql/stitching/docs)** Schema stitching creates a single GraphQL gateway schema from multiple underlying GraphQL services.

**Discarded Options:**
1. **Schema Stitching:**
    1. Schema Stiching does not support aggregating data from various sources.
    2. Schema Stitching does not provide support for hiding/renaming queries.

**Decision:** We'll use **GraphQL Mesh** for the respective reasons:
1. GraphQL Mesh aggregates data from various sources (GraphQL, REST, gRPC) into a single endpoint, simplifying client-side data fetching.
2. GraphQL Mesh is compatible with Cloudflare Workers, enabling deployment at the edge for enhanced latency and performance.
3. GraphQL Mesh combines multiple GraphQL schemas, streamlining the development of complex microservices.
4. GraphQL Mesh provides type safety by generating TypeScript types based on data source schemas, offering compile-time checks and autocomplete for reduced errors.
5.  GraphQL Mesh fetches only necessary data, reducing over/under-fetching and enhancing performance.
6. GraphQL Mesh is highly extensible and customizable, allowing developers to add custom resolvers, middleware, and plugins to tailor the behavior of their GraphQL APIs according to their specific requirements.
7. GraphQL Mesh supports batching of requests into a single request, reducing network round-trips and improving efficiency.

**Consequences:**
1. Team members will need to familiarize themselves with setting up a GraphQL Mesh server with Cloudflare Workers.
2. Team members will need to familiarize themselves with binding of services using Graphql Mesh and Cloudflare Workers.
3. Team members will need to familiarize themselves with extending of schema and adding extended resolvers.

**References**
This [document](https://www.notion.so/kaosagency/Gateway-Services-Mesh-SS-Document-6092c81d3761431aa9785326579b8682) is based on the POC done using both Graphql Mesh and Schema Stitching.