# Service Level GraphQL Specs and Conventions

**Context:** We need to set a standard for naming the queries & mutations, the query format and the input format for the different services.

**Problem:**
1. We need to standardize the naming convention for the queries and mutations of the different services including the gateway.
2. We need to standardize the format in which the data will be request from the query through the gateway(especially in case of nested queries).
3. We need to standardize the input format for the queries and mutations of the different services.
We need these to ensure a smooth development process and consistancy throughout the application.

**Considered Options:**
1. **Schema Naming Convention**
    1. Using `get` as prefix for queries eg. getUsers.
    2. Using the entity name for queries eg. users.
    3. Using camelCase for naming queries & mutations.
    4. Using camelCase for naming queries & mutations inputs fields.
    5. Using camelCase for naming various types.
    6. Naming inputs as `MutationName` + `Input` prefix.
    7. Naming responses as `EntityName` + `Response` prefix.
    8. Naming enum values as SCREAMING_SNAKE_CASE

2. **Requesting data from the gateway in case of nested queries**
    1. Do not use the `get` prefix, `byId` suffix etc to the extended query names, use the entity name in camelCase.
    2. Always request the `primary identifier` of the field being queried and the `common field` in the parent & nested query.

3. **Input Format for queries and mutations**
    1. Use Array of parameters as input for the queries.
    2. Use Array of parameters as input for the mutations.
    3. Use single parameter as input for the queries.
    4. User single parameter as input for the parameters.

**Discarded Options:**
1. **Schema Naming Convention**
    1. Using `get` as prefix for Queries -> Based on the schema naming convention [document](https://www.apollographql.com/docs/technotes/TN0002-schema-naming-conventions/#field-names).

2. **Requesting data from the gateway in case of nested queries**
    N/A

3. **Input Format for queries and mutations**
    1. Use single parameter as input for the queries(READ) -> Queries are mostly read operations which can return multiple values at once.
    So we can pass an array of parameters as inputs, example input of ids to receive the data. This will prevent from using multiple requests causing network overhead.
    This is based on the POCs and projects done before.


**Decision:** 
1. **Schema Naming Convention**
    1. Using the entity name for queries eg. users.
    2. Using camelCase for naming queries & mutations.
    3. Using camelCase for naming queries & mutations input fields.
    4. Using camelCase for naming various types like type, enum, interface, union, scalar.
    5. Naming inputs as `MutationName` + `Input` prefix eg. CreateUserInput.
    6. Naming responses as `EntityName` + `Response` prefix eg. CreateUserResponse.
    7. Naming enum values as SCREAMING_SNAKE_CASE eg.
        ```
            enum MyEnum {
                VALUE_ONE
                VALUE_TWO
            }
        ```
    Based on the schema naming conventions of [Apollo](https://www.apollographql.com/docs/technotes/TN0002-schema-naming-conventions/#field-names), [Medium](https://medium.com/@andriiandriiets/graphql-standards-and-practices-da3246dfb619).

2. **Requesting data from the gateway in case of nested queries**
    1. The naming of queries and entities will follow the same convention as naming in individual services, since the nested query in the gateway is nothing but a nested entity when fetched from the frontend.
    2. Always requesting the `primary identifier` (could be `id` or any other field) and the `common field` is required to ensure A. No unexpected errors and B. No logical error in terms of data returned.
    For example, if we have tasks and user assigned to those tasks and we want to fetch user details from the tasks, the user's id will be the common identifier in this case, hence the query should be made like-
    ```
    tasks{
        id
        title
        status
        assignee_id
        assignee {
            id
            name
            role
        }
    }
    ```
    Here, the `assignee_id` of the task will be used to map with the `id` of the assignee(User) to return the correct result.
    The gateway requires some common identifier between the parent and nested query to map the values with the correct parent, this is especially required when queries are batched.
    Based on the proejcts and POCs done previously.

    > **_NOTE:_**  Assuming above case, of fetching task and then the assignee's (user's) details from the task, there are a few things to keep in mind. 
    Firstly, it is possible that the task might not have an assignee assigned yet, in such case, `the assignee_id` would be `null`, the microservices should be equipped so handle such cases.
    Secondly, the gateway will throw an error in case any required field is not a part of selection set. Here, the required field refers to the field that will be passed as input to the nested query (eg. the `assignee_id` will be used to query the Users microservice to get the user details by id, if this value is missing in selection set of task, the gateway will throw error). Kindly, recheck and fix your query if such problems arise.

3. **Input Format for queries and mutations**
    1. Queries - Read Operations
        1. Use array of parameters as input - This is based on the POCs and projects done before. We can query for multiple fields of data reducing network round trips and increasing efficiency.
    2. Mutations
        1. Create
            1. Use single parameter as input -  Some mutations require security and are limited to one entity, for example, authentication, we should not alllow to create and return multiple access token for multiple users in one go, this compromises the security.
            Similarly, creation of admin for a team sometimes requires only one admin.
            2. Use array of parameters as input - Sometimes a user should be able to create multiple things at once, for example, if a user is adding the steps to do for a task, they should be able to send a request with all the tasks in one go. This will lead to a better user experience than writing out one step and pressing send.

        2. Update
            1. Use single parameter as input - If a mutation handles something critical, or something non-trivial, always make sure to use single parameter as input, we should not let the update operations compromise the security or privacy of the system. Single parameters are highly preferred in cases like updating emails or passwords, these things should only be allowed for an authenticated and authorised user for the user's account only.
            2. Use array of parameters as input - Some update mutations can be designed to accept an array of parameters as input, this can be allowed for  mutations like archiving, or, if there's a checklist then we should allow user to tick many things at once etc. This is done considering the security requirements and permissions a user has.
        
        3. Delete
            1. Use single parameter as input - Delete mutations should also be designed with single paramater inputs preferrably. Deletion of users etc are highly trivial operations and should be carried out carefully, allowing only authorised user to perfom this operation for a single input at a time will ensure robustness in the system.
            2. Use array of parameters as input - Some delete mutations should be allowed to have an array of parameters, as long as the user is authorised to perform those operations . A user might want to delete all the read notifications etc. 

        > :warning:
        > Array parameters support should be added ONLY if it is absolutely required by product and the developer should add information justifying the use for the same. Developers should always calculate the requirements and its possible effects before deciding which option to choose(i.e. single parameter or array of parameters).

    Have referred to options provided by certain ORMs like [Hasura](https://hasura.io/docs/latest/queries/postgres/index/) as well as insights gained from previously done POCs and projects.

    3. Input format and conventions for filters and sorting - [Open CRUD](https://www.opencrud.org/#sec-Queries-Filters) has listed the conventions for various types of filtering and how it should be passed as inputs in case any such option is being provided.


**Consequences:**
1. The team members will need to familiarize themselves with the concept of gateway nested queries.
2. The team members will need to update their schema designs with this document and make sure they adhere to the guidelines.

**References:**
1. [Apollo Schema Convention](https://www.apollographql.com/docs/technotes/TN0002-schema-naming-conventions/)
2. [OpenCRUD](https://www.opencrud.org/#sec-Mutations-Nested)
3. [Medium.com GQL Convention](https://medium.com/@andriiandriiets/graphql-standards-and-practices-da3246dfb619)


**NOTE:** Cursor based pagination(relay specs) has been done as a part of another [ADR](https://github.com/KAOS-agency/feedback-circuit/pull/77).