# ANALYTICS 

* Amazon Athena : serverless query service
    *  Lets you run SQL queries directly on data in S3
    *  No servers to manage, no infra to provision

* AmazonKinesis Data Streams (KDS) : 
    * Use it to collect, buffer, and process real-time data (e.g., logs, metrics, events) 
    * (you build consumers) : Lambda, Kinesis Data Analytics, KCL apps, Firehose, EC2
    * Data is stored in shards, retained for up to 24 hrs - 365 days (configurable)
    * Pay for shards + data volume

* Amazon Kinesis Data Firehose :
    * Ingests streaming data and automatically delivers it to S3, Redshift, OpenSearch, or 3rd party tools 
    * fully managed
    * Supports optional data transformation, format conversion, compression, and encryption

# APPLICATION INTEGRATION 

* AWS AppSync : 
    * AppSync is a fully managed GraphQL API service that makes it easy to securely access, query, and update data from various sources.
    * Real-time subscriptions, Offline sync support
    * Request flexibility - Client can request exact data

* AWS EventBridge : serverless event bus service that helps you connect apps using events. 
    * Ingests events from: AWS services ,Custom apps ,SaaS apps
    * You can filter, route, and transform events to targets like: Lambda, Step Functions, SNS/SQS ,API Gateway ,Event buses (cross-account)
    * Schedule Events (CRON)
    * Event Buses : 
        * Default Bus : AWS service events
        * Custom Bus : Custom app events
        * Partner Bus : SaaS apps (like Zendesk, Auth0)

* Amazon SNS : A fully managed pub-sub (publish-subscribe) messaging service
    * Publisher → SNS Topic → 📩 Subscribers (SQS, Lambda, Email, etc.)
    * Lets you:
        * Publish messages to topics
        * Fan-out those messages to multiple subscribers
        * Push alerts, updates, and notifications

* Amazon SQS : A fully managed message queuing service
    * [Producer] --(send msg)--> [SQS Queue] --(poll msg)--> [Consumer]
    * Producers send messages to the queue
    * Messages stay in the queue (durable storage)
    * Consumers poll the queue, process messages
    * Messages auto-deleted after successful processing
    * Queue : Standard and FIFO
    * If the consumer fails to process, the message becomes visible again and gets retried. You can tune this timeout based on expected processing time.
    * Msg Retention : 4 days - 14 days
    * If a message fails X times (e.g., 5), it can be moved to a DLQ for later inspection
    * SQS has no native filtering → that’s SNS or EventBridge’s game

* AWS Step Functions : Fully managed serverless workflow orchestration service
    * Lets you coordinate multiple AWS services (or your own code) in sequence or parallel
    * Visual, reliable, and with built-in error handling
    * Think of it as: “Instead of writing glue code to call Lambda → SNS → DynamoDB → S3... let Step Functions handle the flow!”
    * Standard model for everyday tasks and express for higher throughput

# COMPUTE

* Amazon EC2 :  
    * AWS’s core service for running virtual servers (instances) in the cloud
    * Launch with AMI (Amazon Machine Image) → defines OS + preinstalled software
    * Storage types : 
        * EBS
        * EFS
        * Instance Store
    * Security Groups (SG) = virtual firewall (controls inbound/outbound ports)
    * Pricing Models : 
        * On-Demand : pay-as-u-go-model
        * Reserved : commit to 1-3 yrs, cheaper
        * Spot : Bid for unused capacity
        * Savings Plan : Commit to spending(like a netflix subscription)
    * Auto Scaling Groups (ASG) to scale based on load
    * Elastic Load Balancer (ELB) to distribute traffic across instances

* AWS Elastic Beanstalk : platform-as-a-service (PaaS) on AWS
    * Lets you deploy + manage web apps without worrying about the underlying infrastructure
    * You provide your code → EB does the rest
    * Deployment Structure : 
        * Application : The container for versions and environments
        * Environment : An actual deployed instance of your app (dev, test, prod)
        * Environment Tier : 
            * Web server (HTTP, front-facing apps)
            * Worker (background job processing)
    * Deployment Options : You can deploy via: ZIP file upload (via console or CLI), GitHub / CodePipeline, EB CLI, CI/CD
    * EB stores app versions in S3 under the hood
    * You can tweak all deployement options in the .ebextensions file, eb console, eb cli

* AWS Serverless Application Model : It’s a framework + syntax + tooling for building serverless apps on AWS
    * You can:
        * Define Lambda, API Gateway, DynamoDB, SQS, SNS, etc. in a simple YAML file
        * Build + test locally
        * Deploy with a single command
        * You must use an S3 bucket when deploying (SAM uploads artifacts there)

* AWS Lambda : Serverless compute service → run code without managing servers
    * You write the code → Lambda handles provisioning, scaling, fault tolerance
    * Pay only for:
        * Invocation count
        * Execution duration
    * Execution Model : 
        * Cold Start: First request spins up container (slower)
        * Warm Start: Reuses existing container (fast)
    * Timeout limit = 15 mins max
    * Use Layers to share code/dependencies
    * Use Environment Variables to config your function

# CONTAINERS

* AWS Copilot : CLI tool to deploy + manage containerized apps on AWS easily(ECR)
    * If you see a question about deploying Docker apps to ECS with minimal config, Copilot is the answer
    * Know it handles:
        * Docker build + push to ECR
        * Load balancer setup
        * IAM + VPC + ECS cluster creation
            * It simplifies CI/CD setup (CodePipeline)
            * Ideal for Docker devs who want AWS-native deployments

* Amazon Elastic Container Registry (Amazon ECR) : A fully managed Docker container registry for AWS
    * ECR supports lifecycle rules to delete old images (e.g., keep last 5, delete rest)

* Amazon ECS : AWS’s native container orchestration platform
    * Runs Docker containers on:
        * Fargate (serverless)
        * EC2 (self-managed VMs)
    * Cluster
        * A logical grouping of container instances or Fargate capacity
        * Can run on EC2 or Fargate
    * Task Definition : A blueprint for your app, Defines:
        * Docker image
        * CPU/memory
        * Env vars
        * Volumes
        * Networking
        * IAM roles
    * Task :  A running instance of a task definition
    * Service : Keeps a specified number of tasks running
    * IAM with ECS : 
        * Execution Role = ECS uses it to pull image from ECR, write logs, etc.
        * Task Role = Container assumes this role to access S3, DynamoDB, etc.

* Amazon EKS : AWS-managed Kubernetes
    * Runs upstream K8s clusters on AWS
    * If you see "Kubernetes YAML", "multi-cloud portability", or "use existing Helm chart" — it's EKS. If the dev wants “easy Docker deploys” or “simple backend” — go with ECS or Lambda.

# DATABASE

* Amazon Aurora : AWS’s high-performance relational DB, serverless option
    * compatible with MySQL and PostgresQL
    * Aurora has replication + failover + read scaling built-in
    * Aurora Global DB enables cross-region DR

* Amazon RDS : managed relational DB service
    * Supports: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Aurora
    * Handles: Backups, Patching, Monitoring ,Scaling 
    * DB Instance : A managed database server
    * DB Snapshot : Manual or automatic backups of your DB instance, Supports point-in-time restore
    * Read Replicas : Offload read traffic for scalability, Only available for MySQL, Postgres, MariaDB, and Aurora, Can be promoted to standalone DBs
    * Read Replicas = Scale reads, not HA
    * Multi-AZ = High Availability, not performance

* Amazon DynamoDB : managed NoSQL key-value + document DB
    * Serverless, scalable, fast AF (single-digit ms latency)
    * Use partition key (and optional sort key) for schema
    * Supports on-demand & provisioned throughput
    * DAX = in-memory cache, microsecond reads
    * DynamoDB Streams : Triggers on item-level changes (used w/ Lambda)
    * Global Tables for multi-region sync
    * so lsi is querying for info from partion key and sort but gsi is query for info from other attributes

* Amazon ElastiCache : fully managed in-memory data store
    * Redis  → caching + pub/sub + leaderboard + session store
    * Memcached → simple, fast caching, scalable, no persistence
    * Boosts performance, Reduces DB strain, Used in real-time apps
    * Use ElastiCache when data is ephemeral (cache)

* Amazon MemoryDB : managed in-memory DB, Redis-compatible
    * Use MemoryDB when you need in-memory performance + full durability
    * Multi-AZ cluster with synchronous writes
    * Stores data in in-memory nodes and durably replicates across AZs

# DEVELOPER TOOLS

* AWS Amplify : AWS’s toolkit + hosting platform for building full-stack apps FAST
    * You push code → Amplify builds → Deploys to global CDN → HTTPS + custom domain ready
    * Host web/mobile apps
    * Add backend features like auth, API, DB, storage 
    * Support CI/CD + custom domains

* AWS CodePipeline : 
    * CodePipeline = AWS’s fully managed CI/CD service
    * It automates the build → test → deploy steps for your code
    * [ Source ] → [ Build ] → [ Test (optional) ] → [ Deploy ]
    * Supports manual approvals

* AWS CodeBuild : CodeBuild = AWS’s fully managed build service
    *  It compiles your source code, runs tests, produces build artifacts (like zip, jar, docker image).
    * Supports environment variables
    * Pricing is based on build duration
    * Can build Docker images & push to ECR
    * Integrates with CodePipeline, GitHub, CodeCommit

* AWS CodeDeploy : AWS’s fully managed deployment service
    * It automates deploying your app to:
        * EC2 instances
        * On-prem servers
        * Lambda functions
        * ECS services
    * Deployment Types :
        * In-place
        * Blue/Green
        * Canary
        * Linear
        * All-at-once
    * Add pre/post hooks via Lambda functions for: Smoke testing, Logging, Feature flags

* AWS CodeArtifact : fully managed artifact repository service
    * It stores build artifacts like:
        * Libraries
        * Packages
        * Dependencies

* AWS CodeGuru : AI-powered code reviewer + performance profiler

* AWS X-Ray : distributed tracing service, Helps you visualize, trace, and debug complex requests as they pass through multiple AWS services.
    * Service Map → Visual flow of services + latencies
    * Trace List → Every invocation/request with status
    * Annotations & Metadata → Add custom tags to traces (e.g. user ID, order ID)
    * Analytics → Filter traces (e.g. show all 5xx errors in last 1hr)

# MANAGEMENT AND GOVERNANCE

* AWS AppConfig : AWS service for managing & deploying application configuration safely
    * Lets you:
        * Manage app configs separately from code
        * Deploy configs gradually (think: canary, linear, all-at-once)
        * Monitor for errors & rollback if things break
    * Configs stored in hosted profile / S3 / SSM / SecretsManager

* AWS CloudTrail : service that records all AWS API calls & actions across your account
    * Stores logs in S3, sends to CloudWatch Logs

* AWS CloudFormation : Infrastructure as Code (IaC) service for AWS
    * Stacks : A deployed instance of a template (the real infra)
    * Change Sets : Preview updates to a stack before applying them
    * Rollback : Auto-rollback if deployment fails
    * Nested Stacks : Break large templates into smaller reusable ones
    * Drift Detection : See if someone changed the infra outside the template
    * StackSets : Deploy infra across multiple accounts/regions

* Amazon CloudWatch : AWS’s monitoring + observability service
    * Metrics : Tracks performance (CPU, memory, request count, latency)
    * Logs : Collects logs from Lambda, ECS, EC2, apps, etc
    * Alarms : Sends alerts if metrics breach thresholds
    * Dashboards : Custom visualizations of metrics
    * Insights : Query logs (like SQL) for debugging
    * Events (now EventBridge) : Triggers on events like state changes

* Amazon CloudWatch Logs : centralized service to collect, store, monitor, and analyze log files from AWS resources + your apps
    * Log Group : Logical container for logs (e.g. /aws/lambda/my-func)
    * Log Stream : Sequence of logs from one resource instance (e.g. one Lambda instance)
    * Log Event : One single log line or message ({"timestamp": ..., "message": ...})
    * You can:
        * Export logs to S3 (long-term storage)
        * Stream logs to Elasticsearch/OpenSearch for advanced analytics
        * Forward logs to Lambda for custom processing
    * Retention is configurable (1 day → forever)
    * IAM required to read/write/stream logs

* AWS Systems Manager Parameter Store : Fully managed AWS service for storing config data + secrets (optional).
    * Stores key-value pairs for:
        * App configs (env vars, ARNs, URLs)
        * DB strings, API keys (via SecureString)
        * Passwords (via SecureString)

* AWS Cloud Development Kit : It lets you define AWS infrastructure using real programming languages (not YAML or JSON).

# NETWORK AND CONTENT DELIVERY 

* Amazon CloudFront : AWS’s Content Delivery Network (CDN)
    * Can serve from S3, EC2, ALB, API Gateway, or custom origin
    * Custom error pages (404.html etc)

* Elastic Load Balancing (ELB) : AWS service that automatically distributes incoming traffic across multiple targets (EC2, containers, IPs, Lambda)
    * ALB is used for HTTP/HTTPS, path/host-based routing
    * NLB is for ultra-low-latency TCP/UDP use cases
    * ELB supports EC2, ECS, Lambda, IPs as targets
    * Health checks remove bad targets automatically
    * NLB supports TLS pass-through, static IPs, cross-zone load balancing
	
* Amazon Route 53 : AWS’s scalable and highly available Domain Name System (DNS) + health checking + domain registration service, Routing policies, Traffic control, Global failover.
    * Simple : Default — one record = one target
    * Weighted : A/B testing: 70% to us-east-1, 30% to us-west-2
    * Latency-based : Route users to the lowest-latency AWS region
    * Failover : Primary → Secondary if Primary health check fails
    * Geolocation : Route based on user location (e.g. India → Mumbai, US → Ohio)
    * Geoproximity : Weighted + bias by distance (Traffic Flow only)
    * Multivalue : Return multiple healthy IPs (basic load balancing)

* Amazon Virtual Private Cloud (Amazon VPC) : your own private network inside AWS, Control ingress/egress with firewall rules with security grps and NACLs
    * CIDR Block : The IP range for your VPC (e.g., 10.0.0.0/16)
    * Subnet : Smaller slices inside your VPC (can be public or private)
    * Route Table : Controls traffic routing between subnets + internet
    * Internet Gateway (IGW) : Enables internet access for public subnets
    * NAT Gateway : Lets private subnets talk out to internet securely
    * Security Group : Firewall attached to resources (stateful)
    * Network ACL : Firewall at subnet level (stateless)
    * VPC Peering : Connect 2 VPCs together
    * Endpoints : Private access to AWS services inside VPC
        * Gateway : S3, DynamoDB (route table-based)
            * Interface : Other AWS services (ENI-based)

* AWS API Gateway : It’s a fully managed service that lets you:
    * Create, publish, secure, and monitor REST, WebSocket, and HTTP APIs
    * Basically, you put API Gateway in front of your services (like Lambda, EC2, DynamoDB, etc.)
    * It handles routing, security, throttling, caching, etc.
    * Enforce authentication, rate limiting, monitoring, caching, and throttling
    * Stage Variables = key-value pairs per stage, Only for REST APIs, NOT HTTP APIs, Useful for multi-environment deployments

# SECURITY, IDENTITY AND COMPLIANCE

* AWS Certificate Manager (ACM) : manage SSL/TLS certs for free, Automatically renews them, Deploys them to services like CloudFront, ALB, API Gateway, ELB, etc, 
    * Works with both public and private certificates
    * ACM checks that DNS record to verify domain ownership

* AWS IAM (Identity and Access Management) : control access to AWS services & resources

* AWS Private Certificate Authority (Private CA) : create your own private certs

* AWS WAF (Web Application Firewall) : protects your web apps and APIs from common attacks by filtering HTTP/S traffic at the edge or app layer.

* Amazon Cognito : Fully managed service for authenticating, authorizing, and managing users for your apps.
    *  It’s your app’s built-in login system — no need to roll your own auth
    * Cognito can:
        * Sign up / sign in users
        * Handle password resets, MFA, email/phone verification
        * Integrate with third-party IdPs (Google, Facebook, SAML, OIDC)
        * Issue JWT tokens (ID, access, refresh)
    * Cognito Identity Pools : Give users access to AWS services — securely
    * Cognito User Pools : Manage your users like user sign up / sign in

* AWS STS (Security Token Service) : It gives you temporary security credentials (access key, secret key, session token) to IAM users, Federated users (e.g., Google, SAML, Cognito), Cross-account roles, Applications needing short-term AWS access
    *  so:
        * You can securely access AWS resources
        * You don’t need to hard-code long-term IAM keys
    * AssumeRole : Assume a role from same or another account (most common!)
    * AssumeRoleWithWebIdentity = for Cognito/mobile apps
    * GetSessionToken = add MFA to sessions

* AWS KMS(Key Management Service) : A managed service that enables you to easily encrypt your data. KMS provides a highly available key storage, management, and auditing solution for you to encrypt data within your own applications and control the encryption of stored data across AWS services.

* AWS Secrets Manager : Fully managed AWS service for storing, managing, and rotating secrets securely. Secrets = API keys, DB creds, tokens, OAuth secrets, private keys, etc.

# STORAGE

* Amazon Elastic Block Store (EBS) : block storage for EC2, Snapshots are backups of EBS volumes stored in S3
* Amazon Elastic File System (EFS) : fully managed NFS file system
* Amazon S3 (Simple Storage Service) : object storage for any data type
        * Bucket : Top-level folder — where your files live (global namespace)
        * Object : The actual file you store (has key + metadata + data)
        * Key : File name inside the bucket (like img/logo.png)
        * Region : Bucket is created in one AWS region
        * Prefix : Path inside the key (like a folder structure)
    * Versioning — keep multiple versions of the same file
    * Lifecycle Rules — move files to IA/Glacier after N days
    * Event Notifications — trigger Lambda/SNS/SQS on upload/delete
    * Static Website Hosting — serve HTML/CSS/JS from your bucket
    * Presigned URLs — temporary access to private files
    * Multi-part Upload — break big files into chunks (e.g. 10 GB video)
    * Cross-Region Replication — copy files to another region auto-magically
    * Requester Pays — force downloader to pay the cost

* Amazon S3 Glacier : low-cost archival storage

