# ADR: Input Format

**Context:** We need to define the input format for passing inputs to the various queries and mutations of the microservices.

**Problem:**
1. We need to provide inputs for the various CRUD operations defined by the microservices.
2. We need a consistent input format among all the microservices.
3. We need to define an input format which can be easily passed from gateway through extended queries as well.

**Considered Options:**
1. **Array parameters:** We've considered this option based on previous projects and POCs.
2. **Single parameter:**: We've considered this option based on previous projects and POCs.

**Discarded Options:** N/A

**Decision:** 
1. We'll allow the **Array parameters** as inputs for the various queries and **selected** mutations for the respective reasons:
    1. GraphQL Mesh allows batching of requests into a single request and expects the query to accept an array of parameters.
    2. Sending an array of parameters as input instead of multiple requests with single paramater will reduce network round-trips and improving efficiency.
2. We'll use a **single parameter** for majority of the **CREATE** mutations for the following reasons:
    1. Unlike queries, with mutations the services cannot validate the output for creation, this may lead to failure of entire operations even if one input parameter fails in case we use an array of parameters.


**Consequences:**
1. Team members will need to design the APIs such that they accept and work well with an array of parameters.
2. Team members will need to handle the inputs in such a manner that one bad parameter does not block the entire operation.

**References:** We'll be adding another [ADR](https://github.com/KAOS-agency/feedback-circuit/issues/71) which will contain more information with details on the kind of input parameters to choose as well as the conventions to follow.

