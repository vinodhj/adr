# ADR: Server

**Context:** We need an easy-to-deploy solution that is secure, with high performance, reliability, and scalability.

**Problem:**
1. We need a solution easy deployment without the need to manage infrastructure.
2. We need a solution with low response time and highly scalable nature.
3. We need a solution which supports development with microservice architecture.

**Considered Options:**
1. **[Cloudflare Workers:](https://developers.cloudflare.com/workers/)** Cloudflare workers build serverless applications and deploy instantly across the globe for exceptional performance, reliability, and scale.
2. **[Kubernetes:](https://kubernetes.io/docs/home/)** Also known as K8s, it is an open-source system for automating deployment, scaling, and management of containerized applications.
3. **[AWS Lambdas:](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)** AWS Lambdas allow to run code without provisioning or managing servers, creating workload-aware cluster scaling logic, maintaining event integrations, or managing runtimes.

**Discarded Options:**
1. **Kubernetes:**
    1. Kubernetes has a steep learning curve and can be complex to set up, configure, and manage.
    2. Kubernetes clusters require a significant amount of resources to operate efficiently, including CPU, memory, and storage.
    3. Ensuring high availability in Kubernetes involves deploying redundant components like multiple master nodes, etcd clusters, and worker nodes across various availability zones or regions. This complexity raises deployment architecture intricacies and operational burdens
2. **AWS Lambdas:**
    1. AWS Lambda functions can experience cold start latency.
    2. Lambda functions are constrained by execution time limits and memory constraints, which may limit their suitability for microservices needing longer processing times or handling large data volumes in one go.

**Decision:** We'll use the **Cloudflare Workers** for the respective reasons:
1. Cloudflare Workers enable quick and straightforward deployment of serverless applications without the need to manage infrastructure.
2. Cloudflare Workers can be deployed instantly across Cloudflare's vast network of data centers worldwide, ensuring low latency and high performance for users globally.
3. Cloudflare Workers automatically scale to handle varying levels of traffic without requiring manual intervention, making them suitable for applications with fluctuating demand.
4. Cloudflare Workers execute code at the edge of the network, closer to end-users, resulting in reduced latency and faster response times for requests.
5. Cloudflare Workers come with built-in services such as KV storage and Durable Objects,D1 database enabling developers to build powerful and resilient applications without relying on external services.
6. Cloudflare's robust security features, such as DDoS protection and Web Application Firewall (WAF), are seamlessly integrated with Workers, ensuring applications remain secure against various threats.
7. Cloudflare Workers' pricing model, allows the developers to only pay for the resources they consume, making it a cost-effective solution for both small-scale projects and large enterprises.
8. Cloudflare Workers natively support service bindings which allows to send HTTP requests to another Worker without those requests going over the Internet. The request immediately invokes the downstream Worker, reducing latency as compared to a request to a third-party service making this very useful for microservice architecture.

**Consequences:**
1. Team members will need to familiarize themselves with Cloudflare workers and its API.
2. Team members will need to familiarize themselves with setting up Graphql servers with Cloudflare Workers.

