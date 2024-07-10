# ADR: Implement Relay Spec for GraphQL API

**Context**:  
We are developing GraphQL APIs to serve data to our front-end applications. As we scale our applications and aim for better performance and maintainability, we need to consider adopting best practices and standards in our API development process. One such consideration is implementing the Relay specification, which provides a set of guidelines for structuring GraphQL APIs to optimize client-side caching, pagination, and data fetching.

**Problem**:  
Without a standardized approach for structuring our GraphQL API, we risk inconsistency in how our clients consume data, leading to inefficiencies in data fetching, unnecessary re-renders, and difficulties in maintaining and scaling our applications.

**Considered Options**:

1. **[Relay Specification](https://relay.dev/graphql/connections.htm)**: Adhering to the Relay specification ensures a consistent and optimized approach to data fetching, pagination, and caching.
2. **[Official GraphQL Specification](https://spec.graphql.org/October2021/)**: This is the specification for GraphQL, a query language and execution engine originally created at Facebook in 2012 for describing the capabilities and requirements of data models for client-server applications.
3. **[Apollo Federated Subgraph specification](https://www.apollographql.com/docs/federation/subgraph-spec/)**: For adding subgraph support to GraphQL server libraries.
4. **[OpenCRUD](https://www.opencrud.org/)**: OpenCRUD is a GraphQL CRUD API specification for databases that was started by Prisma.
5. **[GraphQL Data Specification (GDS)](https://github.com/hasura/graphql-data-specification/tree/main?tab=readme-ov-file)**: GDS is a GraphQL API specification for accesssing transactional, analytical and streaming data across multiple data sources that contain semantically related data. Accessing refers to reading, writing or subscribing to data.
6. **[GraphQL over HTTP](https://github.com/graphql/graphql-over-http/blob/main/spec/GraphQLOverHTTP.md)**: This specification details how GraphQL should be served and consumed over HTTP in order to maximize interoperability between clients, servers and tools. This specification does not override or replace the **[GraphQL specification](https://spec.graphql.org/October2021/)**, it extends it to cover the topic of serving GraphQL services over HTTP.
7. **[GraphQXL](https://gabotechs.github.io/graphqxl/)**: GraphQXL is an extension of the GraphQL language with some additional features that help creating big and scalable server-side schemas.
8. **[GraphQL Scalars](https://www.graphql-scalars.com/)**: GraphQL Scalars hosts community defined custom Scalars.

**Discard Options**:

1. **Official GraphQL Specification**: Since this does not specify specifications on pagination, this is will not be a good option.
2. **Apollo Federated Subgraph specification**: Since it focuses mainly on adding federated subgraph support to a GraphQL server library, its beyond the scope of our requirement, we wont use Apollo Federated Subgraphs.
3. **OpenCRUD**: The OpenCRUD specification is currently a working draft, the query connections section says WIP and gives us an example of a generated schema where pagination is added, it is similar to relay specs but since there is proper defination of how connections work for queries, we cannot consider OpenCRUD.
4. **GraphQL Data Specification (GDS)**: The GDS specification is still heavily a WIP and currently an early draft. GDS has `where`, `limit`, `offset`, `order_by` field arguments for queries, we can handle pagination using `limit`, `offset`, `order_by`, this is the model that we had been using earlier in other projects, but since this lacks the cursor based pagination, we are not considering GDS.
5. **GraphQL over HTTP**: Mostly focuses on serving GraphQL services on HTTP, since it extends the official GraphQL specification, it does not cover pagination, HTTP Specification.
6. **GraphQXL**: GraphQXL is just a compiler that will receive .graphqxl files as an input and will compile them down to a common .graphql file. It is not covering any specs for pagination.
7. **GraphQL Scalars**: Apart from hosting community custom scalars (currently its in draft with only one scalar defined), it does not cover Queries, pagination.

**Decision**: We will adopt the **Relay specification** officially known as **GraphQL Cursor Connections Specification** for structuring our GraphQL API due to the following reasons:

1. **Consistency**: Implementing the Relay specification will ensure consistency in how data is fetched and paginated across different parts of our application.
2. **Optimized Performance**: By following Relay's guidelines for pagination and caching, we can optimize the performance of our API and reduce unnecessary data transfers between the server and client.
3. **Cursor based pagination**: Relay specification implements cursor based pagination, which support filtering and sorting , plus easy pagination by referring to the cursor values.


**Consequences**:

Team members will need to familiarize themselves with relay specification.

**References**: N/A