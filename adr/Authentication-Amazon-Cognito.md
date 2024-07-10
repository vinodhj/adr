# ADR: Authentication

**Context**: We need to provide a solution to authenticate and authorise the users of Feedback Circuit.

**Problem**: 

1. We will have multiple users associated with the organisation.
2. We will have different access levels based on different roles for users.


**Considered Options**:

1. **[Amazon Cognito](https://aws.amazon.com/cognito/)**: Simplify user authentication for your web and mobile apps with Amazon Cognito. This powerful identity service offers built-in user sign-up, sign-in, and access control, scaling effortlessly to millions of UAS (Unique Active Users). Enable social logins via Google, Facebook, or your enterprise directory for a familiar and frictionless user experience. Amazon Cognito prioritizes security with advanced features to protect your users and business.

2. **[Auth0](https://auth0.com/)**: Auth0 is a flexible, drop-in solution to add authentication and authorization services to your applications.

**Discarded Options**:

1. **Auth0**: For our use case Auth0 does not fit well with the requirements, Auth0 is expensive compared to Cognito and also Cognito seamlessly integrates with other AWS services, reducing the complexity of managing multiple vendors.

**Decision**: We'll use the **Amazon Cognito** along with **AWS IAM** for the following reasons -

1. Cognito will let us create a userpool.
2. Cognito is a managed service, handles authentication, user sign up, password reset
3. Cognito will let us create user groups in Cognito user pool to categorize users based on roles (e.g., Admins, Managers, Individual Users).
4. For RBAC, we can use AWS IAM (Identity and Access Management) to define permissions for various app actions. Create IAM roles for each user group with appropriate permissions.


**Consequences**:

Team members will need to familiarise themselves with the AWS SDK cognito and iam API.