# ADR: Data Storage

**Context**: We need to provide a solution to store data for Feedback Circuit with the following criteria:
1. We need a storage solution to store feedback data, notes of users, user interactions, keys, notifications.
2. We need storage solutions for storing files/ attachments added to feedbacks, notes, collaborative knowledge repository. 

**Problem**: 

1. Storing feedback data, notes securely.
2. Storing files securely with good accessibility.
3. Storing notifications.
4. Storing temporary keys with the ability to expire after a specific duration.
5. Storing comments, for user interactions.


**Considered Options**:

1. **[TursoDB](https://turso.tech/)**: Turso is a SQLite-compatible database built on libSQL, the Open Contribution fork of SQLite. It enables scaling to hundreds of thousands of databases per organization and supports replication to any location, including your own servers, for microsecond-latency access.
2. **[DynamoDB](https://aws.amazon.com/dynamodb/)**: NoSQL database for scalability and low latency access.
3. **[Amazon S3](https://aws.amazon.com/s3/)**: Amazon S3 is an object storage service that offers scalability, data availability, security, and performance.
4. **[Cloudflare D1](https://developers.cloudflare.com/d1/)**: Cloudflare D1 is a serverless SQL database designed to work with Cloudflare Workers and Pages projects. It allows you to build scalable applications that can handle a large number of users without additional cost. D1 offers a free plan and also has paid plans. One of its key features is the ability to restore your database to any point in time within the last 30 days.
5. **[Cloudflare R2](https://developers.cloudflare.com/r2/)**: Cloudflare R2 Storage allows to store large amounts of unstructured data without the costly egress bandwidth fees associated with typical cloud storage services.
6. **[Cloudflare KV Namespace](https://developers.cloudflare.com/kv/)**: A KV namespace is a key-value database replicated to Cloudflare’s global network.
7. **[Cloudflare Durable Objects](https://developers.cloudflare.com/durable-objects/)**: Durable Objects provide a powerful API for coordinating multiple clients or users, each with private, transactional and strongly consistent storage attached.

**Discarded Options**:

1. **TursoDB**: Since we don't require multi tenant databases, we won't be using TursoDB for database.
2. **Amazon S3**: Since we'll be using cloudflare workers and they natively support R2, it's easier to integrate. S3 is more complicated to integrate with clouflare workers as compared to other considered options.
3. **Cloudflare KV Namespace**: Since we can use Dynamodb for storing key value pair data with TTL, we don't need to use KV along with DynamoDB, also Dynamodb has additional capabilities such as local and global secondary indexes allowing different access patterns for the same underlying data.

**Decision**: We'll use **Cloudflare D1, R2, Durable Objects** and **DynamoDB** for the following reasons -

1. **Cloudflare D1**
    1. **Ease of Use**: D1 is designed to be serverless, meaning Cloudflare manages the infrastructure behind the database.
    2. **Integration with Cloudflare Workers**: D1 integrates seamlessly with Cloudflare Workers.
    3. **Global Network**: Cloudflare's global network can provide low latency access to database for users around the world. D1 automatically creates read-replicas of your data close to users, improving performance.
    4. **Scalability**: D1 is designed to handle large amounts of users without needing you to manage scaling yourself.
    5. **Fast Performance**: Recent improvements have made D1 significantly faster, compared to the initial alpha version.
2. **Cloudflare R2**
    1. Cloudflare R2 provides storage for large amounts of unstructured data.
    2. The costs associated with storing data is comparatively less as compared to other cloud storage services.
    3. Cloudflare R2 is natively supported by cloudflare workers and integrates easily as compared to Amazon S3.
3. **DynamoDB**
    1. DynamoDB is highly scalable and will be used to store notifications, past/historical/activities data.
    2. DynamoDB is used to store data with TTL (expiration time).
    2. DynamoDB's pay-per-request model can align well with usage patterns.
4. **Cloudflare Durable Objects**
    1. Cloudflare Durable Objects can be used for managing comments(CRUD) for Collaborative Knowledge Respository content, which will facilitate user interactions.
    2. Cloudflare Durable Objects provides a consistent Key/Value Storage.


**Consequences**:

1. Team members will need to familiarize themselves with Cloudflare D1's API and integration with Cloudflare Workers.
2. Team members will need to familiarize themselves with Cloudflare R2's API.
3. Team members will need to familiarize themselves with DynamoDB, access patterns and SDK.
4. Team members will need to familiarize themselves with Cloudflare KV.
5. Team members will need to familiarize themselves with Cloudflare Durable Objects.


**Conventions for SQL database**:  
Following conventions are added based on the conventions we followed earlier for other projects where SQL database was used.

1. **Tables and Columns Naming**: Use snake_case for table names and column names. Table names should be singular. Column names should be descriptive and singular, reflecting the attribute they represent. 
For example, a table storing feedback data could be named feedback with columns such as id, user_id, content, created_at, and updated_at.
2. **Foreign Key Naming**: When defining foreign key constraints, follow a consistent naming convention. Typically, foreign keys should have the same name as the primary key they reference, prefixed with the name of the referencing table. For example, if a table feedback references the user table's primary key id, the foreign key column could be named user_id.
3. **Primary Key Selection**: Choose an appropriate primary key for each table. It's common to use an auto-incrementing integer column (id) as the primary key, but in some cases, natural keys or composite keys may be more suitable. Ensure that primary keys are unique and not null.
4. **Indexing Strategy**: Determine which columns need indexing to improve query performance. Typically, columns used frequently in WHERE clauses, JOIN conditions, or ORDER BY clauses are good candidates for indexing. However, avoid over-indexing as it can impact write performance.
5. **Transaction Management**: Use transactions to ensure data integrity and consistency. Begin and commit transactions appropriately to group related database operations together. Additionally, handle transaction rollbacks in case of errors to maintain data consistency.
6. **Normailization**: Implement proper data normalization to reduce redundancy and ensure data integrity.