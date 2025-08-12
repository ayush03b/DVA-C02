# Development with AWS Services (32%)

## Lambda :

- Stateless compute. Event-driven.
- Triggers: API Gateway (sync), S3/SNS (async), SQS/Kinesis (poll-based).
- Concurrency: default limit, provisioned concurrency available.
- Handle retries, idempotency (e.g., using request IDs).
- /tmp = 512MB local storage.
- Timeout: Max 15 mins.
- Deployment: .zip or container image.
- Billing: Per invocation + execution time.
- Integrates: API Gateway, ALB, S3, DynamoDB, CloudWatch.
    - Execution : 
        - Cold Start: New container (slower).
        - Warm Start: Reused container (faster).
- Configuration :  
    - Layers: Share code/dependencies.
    - Env Vars: Config without code changes (KMS encryption possible).
    - Versions: Immutable snapshots.
    - Aliases: Named pointer to version (supports canary deployments).
- Invocation Types : 
    - Synchronous : 
        - Caller waits for result -> Errors handled on client side 
        - Services: API Gateway, ALB, CLI, SDK, Lambda@Edge.
    - Asynchronous : 
        - Event queued, retried up to 3 times.
        - Idempotent code recommended.
        - DLQ for failed events.
        - Services: S3, SNS, EventBridge.
- Destinations : 
    - Lambda Destinations enable routing of asynchronous invocation results and failures to various AWS Services.
    - Async success/failure → SQS, SNS, Lambda, EventBridge. DLQ handles only failures.
- Permissions : 
    - IAM execution role required (best: 1 role per function).
    - Resource policies allow cross-account/service invokes.
- Monitoring : 
    - CloudWatch Logs & Metrics: Invocations, duration, errors, throttles & X-Ray: Tracing (enable active tracing).
- Lambda@Edge : Runs at CloudFront edge → low latency.
- Performance : 
    - More RAM = more CPU.
    - Execution context reused → cache connections.
    - Persistent storage → use S3 or EFS.
- Access the unique request ID for each invocation using : context.aws_request_id


## API Gateway

- REST APIs (more features), HTTP APIs (faster, cheaper), WebSocket APIs.
- Handles routing, security, authentication, throttling, caching, and monitoring without managing servers.
- Stages = dev, test, prod. Supports stage variables(REST APIs only).
- Authorizers: IAM, Cognito, Lambda.
- Caching supported (REST only).
- Supports request/response mapping.
- OpenAPI Support
- CORS Support
- Amazon API Gateway supports mock integrations, which allow you to simulate backend behavior without actually calling any backend service

## SQS

- Queue-based decoupling.
- Flow: Producer → [SQS Queue] → Consumer (polls messages).
- Messages stored durably until processed & deleted. If processing fails → message becomes visible again for retry.
- No native message filtering → use SNS or EventBridge for that.
- Standard = at-least-once, best-effort order. FIFO = exactly-once, ordered.
- Visibility timeout: time to process before reappears.
- DLQ: messages move after maxReceiveCount.
- Lambda can poll SQS (event source mapping).
- Message & Queue Properties : 
    - Retention: 4 days (default) → max 14 days.
    - Message Size: ≤ 256 KB.
    - Delivery: At-least-once (duplicates possible).
    - Ordering: Best-effort (may arrive out of order, unless FIFO queue).
    - Throughput: Unlimited messages & TPS (Standard queue).
    - Latency: < 10 ms typical.
    - Encryption at rest via KMS.
- Long Polling : Consumer waits (up to 20s) for new messages instead of constant polling. Reduces API calls & cost, lowers latency.
- For Large Message Handling store payload in S3, send pointer via SQS 
- Send/receive/delete multiple msgs in one API call → cost & performance benefits.

## SNS

- Pub/Sub. Fan-out to Lambda, SQS, HTTP endpoints.
- Message persistence is NOT guaranteed for SNS
- Filtering: Subscribers can set filter policies (JSON) to receive only matching messages.
- Supports push-based delivery (no polling by subscribers).
- Subscriber Types : 
    - Other AWS Services
    - Other: Email, SMS, HTTP/S endpoints.
    - Can integrate with AWS event sources (e.g., S3 event → SNS).
- Encryption : In-transit (TLS), At-rest (KMS), Client-side encryption (optional)

## DynamoDB

- NoSQL, millisecond reads/writes.
- GSI (global index) vs LSI (local index).
- Query (fast, by key) vs Scan (slow, all items).
- TTL for auto-expiry.
- DynamoDB Streams capture table changes in order so you can process them in near real time with consumers like Lambda. 
- Strongly consistent or eventual reads.
- Write throughput = WCU, read = RCU.
- DAX: Fully managed, in-memory cache, microsecond latency, solves hot-key issues, TTL default 5 min.
- Performance & Partitioning : 
    - Choose high-cardinality partition keys to avoid hot partitions.
    - Use suffix/prefix hashing to distribute load evenly.
    - Unprocessed items generate UnprocessedKeys response object when using BatchGetItem or BatchWriteItem, Solution : Retry throttled requests with exponential backoff + jitter.
- BatchWrite/BatchGet reduce requests.
- Transactions : 
    - ACID across multiple items/tables.
    - Max 25 operations per transaction.
    - Use TransactWriteItems / TransactGetItems.
    - Costs 2× RCU/WCU.

## EventBridge

- Serverless event bus for routing events between AWS services, SaaS apps, and custom apps.
- Supports event filtering, routing, and transformation.
- Better than SNS for microservices.
- Default bus for AWS events, custom bus for app events.
- Event Sources : 
    - AWS Services → e.g., EC2 state changes, S3 uploads, CodeBuild failures.
    - Custom Applications → send app-specific JSON events.
    - SaaS Applications → via Partner Event Buses (e.g., Zendesk, Datadog, Auth0).
- Targets: Lambda, Step Functions, SNS / SQS, API Gateway, Event Buses (incl. cross-account), ECS tasks, Batch jobs, Kinesis, CodePipeline, SSM Automation.
- Event Buses : 
    - Default Bus → Receives AWS service events automatically.
    - Custom Bus → For your own applications/events.
    - Partner Bus → Direct integration with SaaS providers.
    - Cross-Account Access → Resource-based policies allow other accounts to send events.
- Schedule Events (CRON) : 
    - Event Bus : Pipeline for events. Types: default, custom, or partner.
    - Event → JSON describing something that happened.
    - Rule → Pattern to match events, define routing.
    - Target → Destination action (e.g., Lambda, SQS, Step Functions, API Gateway).

## Step Functions

- Serverless workflow orchestration — coordinates AWS services & custom code.
- Workflow orchestration, JSON state machine.
- State Types : 
    - Task → Runs Lambda, ECS, Batch, DynamoDB, SNS/SQS, API Gateway, etc.
    - Choice → Conditional branching.
    - Fail / Succeed → End workflow with failure/success.
    - Pass → Pass data without performing work.
    - Wait → Delay execution.
    - Map → Run steps for each item in a dataset.
    - Parallel → Run branches concurrently.
- Error Handling : 
    - Retry → Set attempts, delay, backoff rate, max attempts (for transient errors).
    - Catch → Define alternate flow on error, pass error info via ResultPath.
- Use for complex sequencing and error handling.
- Express vs Standard: short-lived vs long-running.

## Kinesis

- Data Streams = low-latency real-time processing.
- Lambda can consume from shards.
- Good for clickstreams, logs.
- Firehose auto-loads to S3, Redshift.
- shard capacity = 1MB/s in, 2MB/s out, 5 TPS reads

## S3

- Object storage. 
- Supports SSE-S3, SSE-KMS, Enforce encryption via bucket policy (x-amz-server-side-encryption).
- Security : IAM policies (user-based), bucket policies (resource-based), ACLs.
- Versioning and lifecycle rules.
- Triggers: Lambda, EventBridge.
- Consistency: read-after-write.
- Tiering: Standard, IA, One Zone, Glacier.
- Lifecycle Rules : Transition objects between storage classes (e.g., Standard → IA → Glacier), Expire/delete old versions, Filter by prefix or tags.
- Supports Event Notifications
- Replication : CRR (cross-region) / SRR (same region), Requires versioning on both source & destination, IAM permissions needed, Use cases: compliance, DR, log aggregation.
- Supports Transfer Acceleration → Use edge locations to speed uploads/downloads.
- Requester Pays → Downloader pays data transfer cost.
- Presigned URLs → Temporary GET/PUT access (console: up to 12h, CLI: up to 7d).
- Access Points → Custom endpoints w/ specific policies (internet or VPC).
- Object Lambda → Modify data on retrieval via Lambda (e.g., redact, transform).
- CORS : 
    - Controls browser cross-origin requests.
    - Requires explicit permission via CORS config in S3.

## Architectural Patterns

- Loose coupling: SQS, SNS, EventBridge.
- Microservices: use APIs or messaging.
- Choreography (event-driven) vs orchestration (Step Functions).
- Idempotency = safe retries.
- Stateless services = easier to scale.

# Security (26%)

## IAM

- Users, groups, roles. Use roles, not hardcoded credentials.
- Policies: identity-based (who can do what), resource-based (who can access resource).
- Roles : Temporary creds via STS for services, cross-account, federation.
- Least privilege = best practice.
- Trust policy = who can assume a role.
- STS = temporary creds.

## Authentication

- Cognito Identity Pools : Give users access to AWS services — securely
- Cognito User Pools : Manage your users like user sign up / sign in
- Integrate with third-party IdPs (Google, Facebook, SAML, OIDC)
- API Gateway can use Cognito or Lambda authorizer.

## Authorization

- JWT = bearer token. Validate signature and claims.
- IAM policies with conditions.
- Role Based Access Control with Cognito groups.

## Encryption

- At rest: SSE-S3, SSE-KMS, SSE-C, RDS encryption.
- In transit: TLS/SSL (HTTPS).
- KMS: customer-managed vs AWS-managed keys.
- Key rotation (customer-managed: 1 year, AWS-managed: auto every 3).
- CMK = customer master key.
- Envelope encryption: encrypt data key with CMK.
- Client-side encryption: You encrypt before uploading to AWS.

## Secrets Management

- Secrets Manager: stores and rotates secrets.
- Parameter Store: secure strings, no auto-rotation.(for config data)
- Avoid env vars for secrets unless encrypted.

## Secure Practices

- No hardcoded creds or tokens.
- Audit IAM actions with CloudTrail.
- Use MFA, SCPs, permission boundaries (advanced).

# Deployment (24%)

## CodePipeline

- Orchestration: source → build → test(optional) → deploy.
- Supports approvals, notifications.
- Triggered by repo push or manual.

## CodeBuild

- Compiles, tests code.
- buildspec.yml defines build steps.
- Securely access env variables and secrets.

## CodeDeploy

- Deployment to EC2, Lambda, ECS.
- In-place(ApplicationStop → BeforeInstall → AfterInstall → ApplicationStart → ValidateService), blue/green, All-at-once.
- For Lambda: supports linear and canary.

## Deployment Strategies

- All-at-once: fast, risky.
- Rolling: update in batches.
- Blue/green: parallel envs, switch traffic.
- Canary: test new version with % of users.
- Use stages (API Gateway), aliases (Lambda), stacks (CFN).

## IaC Tools

- CloudFormation: declarative templates (YAML/JSON).
- AWS SAM: shorthand for Lambda + API Gateway + DynamoDB.
- CDK: write infra in Python, TypeScript, etc.
- Copilot: ECS/Fargate deployment.
- Amplify: frontend + backend CI/CD for web apps.

## Artifact Repositories

- CodeArtifact: package manager (npm, Maven).
- ECR: Docker image storage.

## Configuration Management

- AppConfig: dynamic config, feature flags.
- Parameter Store: config + secrets.
- Env vars: easy, but not secure for secrets.

# Troubleshooting and Optimization (18%)

## Monitoring

- CloudWatch: metrics, alarms, dashboards.
- Custom metrics via PutMetricData or EMF.
- Logs: view logs from Lambda, ECS, EC2.

## CloudWatch Logs Insights

- Search logs: filter, stats, fields.
- Great for pinpointing errors.

## X-Ray

- Distributed tracing.
- Visualize latency across services.
- Service map and annotations.

## Common Errors

- 4XX: client errors (403, 404, 429).
- 5XX: server errors (500, 502, 503).
- Lambda timeouts, out-of-memory, throttling.
- DynamoDB throughput exceeded = backoff.

## Optimization

- Use cache (ElastiCache, DAX).
- Batch operations (SQS, DynamoDB).
- CloudFront for global content.
- API Gateway cache for REST APIs.

## Concurrency

- Lambda scales automatically, but watch for concurrency limits.
- Provisioned concurrency reduces cold starts.

## Profiling

- CodeGuru Profiler: find hotspots in code.
- Right-size memory/CPU for Lambda or EC2.

## Debugging Tools

- Logs + X-Ray = root cause.
- Use trace ID/correlation ID for tracking.

## Notifications

- SNS for alerts.
- Alarms on errors, high latency, usage thresholds.

## Service Quotas

- Know limits: Lambda timeout (15 min), SQS size (256KB), Lambda concurrency (default 1000), DynamoDB WCU/RCU throttling.
- Use Trusted Advisor or Service Quotas to check.
