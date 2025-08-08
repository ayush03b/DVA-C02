# ANALYTICS

- Amazon Athena : serverless query service
    - Lets you run SQL queries directly on data in S3
    - No servers to manage, no infra to provision

- AmazonKinesis Data Streams (KDS) :
    - Use it to collect, buffer, and process real-time data (e.g., logs, metrics, events)
    - (you build consumers) : Lambda, Kinesis Data Analytics, KCL apps, Firehose, EC2
    - Data is stored in shards, retained for up to 24 hrs - 365 days (configurable)
    - Pay for shards + data volume

- Amazon Kinesis Data Firehose :
    - Ingests streaming data and automatically delivers it to S3, Redshift, OpenSearch, or 3rd party tools
    - fully managed
    - Supports optional data transformation, format conversion, compression, and encryption

# APPLICATION INTEGRATION

- AWS AppSync :
    - AppSync is a fully managed GraphQL API service that makes it easy to securely access, query, and update data from various sources.
    - Real-time subscriptions, Offline sync support
    - Request flexibility - Client can request exact data

- AWS EventBridge : serverless event bus service that helps you connect apps using events.
    - Ingests events from: AWS services ,Custom apps ,SaaS apps
    - You can filter, route, and transform events to targets like: Lambda, Step Functions, SNS/SQS ,API Gateway ,Event buses (cross-account)
    - Schedule Events (CRON)
        - Event Bus : Pipeline for events. Types: default, custom, or partner.
        - Event : JSON object describing something that happened (e.g., "EC2 instance started").
            - Rule : Pattern that matches events and routes them to targets.
            - Target : Action taken when a rule matches: Lambda, SNS, SQS, Step Functions, etc.
    - Event Buses :
        - Default Bus : AWS service events
        - Custom Bus : Custom app events
        - Partner Bus : SaaS apps (like Zendesk, Auth0)

- Amazon SNS : A fully managed pub-sub (publish-subscribe) messaging service
    - Publisher → SNS Topic → 📩 Subscribers (SQS, Lambda, Email, etc.)
    - Lets you:
        - Publish messages to topics
        - Fan-out those messages to multiple subscribers
        - Push alerts, updates, and notifications
        - Subscribers can set filter policies to receive only specific messages

- Amazon SQS : A fully managed message queuing service
    - [Producer] --(send msg)--> [SQS Queue] --(poll msg)--> [Consumer]
    - Producers send messages to the queue
    - Messages stay in the queue (durable storage)
    - Consumers poll the queue, process messages
    - Messages auto-deleted after successful processing
    - Queue : Standard and FIFO
    - If the consumer fails to process, the message becomes visible again and gets retried. You can tune this timeout based on expected processing time.
    - Msg Retention : 4 days - 14 days
    - If a message fails X times (e.g., 5), it can be moved to a DLQ for later inspection
    - SQS has no native filtering → that’s SNS or EventBridge’s game

- AWS Step Functions : Fully managed serverless workflow orchestration service
    - Lets you coordinate multiple AWS services (or your own code) in sequence or parallel
    - Visual, reliable, and with built-in error handling
    - Think of it as: “Instead of writing glue code to call Lambda → SNS → DynamoDB → S3... let Step Functions handle the flow!”
    - Standard model for everyday tasks and express for higher throughput

# COMPUTE

- Amazon EC2 :
    - Amazon EC2 stands for Elastic Compute Cloud and provides Infrastructure as a Service (IaaS) on AWS.
    - Types: General, Compute, Memory, Storage optimized.
    - Bootstrapping EC2 instances is possible using User Data scripts that run once at first launch with root privileges.
    - Launch with AMI (Amazon Machine Image) → defines OS + preinstalled software
        - An AMI (Amazon Machine Image) customizes EC2 instances by prepackaging software and configurations.
        - AMIs can be created by AWS, customized by users, or purchased from the AWS Marketplace.
        - Creating your own AMI results in faster boot and configuration times for EC2 instances.
        - AMIs can be region-specific but are copyable across AWS regions to leverage global infrastructure.
        - You cannot enable encryption directly on an existing unencrypted AMI
        - If the source AMI is unencrypted, the copy will also be unencrypted unless you explicitly choose to encrypt during the copy

    - Storage types :
        - EBS(Elastic Block Storage) :
            - EBS Volumes are network-attached storage devices that persist data beyond the lifecycle of EC2 instances.
            - Each EBS Volume is bound to a specific availability zone and can only be attached to one instance at a time.
            - EBS Volumes must be provisioned with capacity and IOPS in advance, and can be detached and reattached quickly for failover scenarios.
            - The "Delete on Termination" attribute controls whether an EBS Volume is deleted when its associated EC2 instance is terminated, with root volumes deleted by default.
            - EBS Multiattach :
                - The Multi-Attach feature allows the same EBS volume to be attached to multiple EC2 instances within the same availability zone.
                - This feature is available only for the io1 and io2 families of EBS volumes.
                - Up to 16 EC2 instances can simultaneously attach to the same volume.
                - A cluster-aware file system is required to use Multi-Attach effectively.

            - EBS Snapshots :
                - EBS Snapshots are backups of EBS volumes at any point in time and do not require detaching the volume from the EC2 instance.
                - Snapshots can be copied across different Availability Zones and Regions, enabling volume restoration elsewhere.
                - The EBS Snapshot Archive tier offers up to 75% cost savings but requires 24 to 72 hours for restoration.
                - The Recycle Bin feature allows recovery of accidentally deleted snapshots with retention from one day to one year.
                - Fast Snapshot Restore initializes snapshots fully to eliminate first-use latency but incurs significant costs.
                - Migrating EBS volumes across AZs requires creating and restoring snapshots.
            - EBS Volumes :
                - EBS volumes come in six types: General Purpose(gp2, gp3), HighIOPS(io1, io2), HDD Volumes & cannot be used as boot volume(st1, and sc1).

        - EC2 Instance Store
            - EC2 Instance Store provides hardware-attached disk storage for EC2 instances, offering very high I/O performance.
            - Instance Store storage is ephemeral and data is lost if the instance is stopped or terminated.
            - Ideal use cases for Instance Store include buffers, caches, scratch data, or temporary content, but not long-term storage.
            - Users are responsible for backing up and replicating data stored on Instance Store to prevent data loss.
        - EFS(Elastic File System)
            - Amazon EFS is a managed, scalable, and highly available network file system compatible with Linux-based EC2 instances across multiple availability zones.
            - EFS supports different performance modes (General Purpose and Max I/O) and throughput modes (Bursting, Provisioned, Elastic) to suit various workload needs.
            - Storage classes include Standard, Infrequent Access (EFS-IA), and Archive tiers, with lifecycle management policies to optimize cost.
            - EFS offers multi-AZ and single-AZ deployment options, balancing availability, durability, and cost savings up to 90% with appropriate storage class choices.

    - Security Groups (SG) = virtual firewall (controls inbound/outbound ports)
    - Billing: On-Demand, Reserved (1–3 yr), Spot (cheap, interruptible), Savings Plans, Dedicated Host.
    - Never enter your IAM Access Key ID and Secret Access Key directly into an EC2 instance.Use IAM roles to securely provide AWS credentials to EC2 instances.
    - Security Groups :
        - Firewalls controlling inbound and outbound traffic for EC2 instances.
        - Security groups contain only allow rules and can reference IP addresses or other security groups.
        - Default security group behavior blocks all inbound traffic and allows all outbound traffic.
        - PORTS -> SSH:22, FTP:21, HTTP:80, HTTPS:443, RDP:3389
        - Timeouts when connecting to EC2 instances often indicate misconfigured security group rules.
        - Multiple security groups can be attached to an EC2 instance, and one security group can be attached to multiple instances.
        - AWS charges for each Public IPv4 address created in your account

    - LOAD BALANCING :
        - Load balancers distribute incoming traffic across multiple backend EC2 instances to optimize resource use and improve fault tolerance.
        - Elastic Load Balancers provide a single endpoint for users, hiding the complexity of backend instances.
        - Health checks enable the load balancer to route traffic only to healthy instances, enhancing reliability.
        - Application Load Balancer :
            - Application Load Balancers (ALB) operate at layer seven, supporting HTTP and enabling routing to multiple HTTP applications across machines.
            - ALBs support advanced routing features including path-based, host-based, query string, and header-based routing.
            - Target groups behind ALBs can consist of EC2 instances, ECS tasks, Lambda functions, or private IP addresses.
            - ALBs provide fixed hostnames and use headers like X-Forwarded-For to preserve client IP information during connection termination.
        - Network Load Balancer :
            - The Network Load Balancer (NLB) operates at layer 4, handling TCP and UDP traffic with ultra-low latency and high performance.
            - NLB provides one static IP per availability zone, allowing assignment of elastic IPs for applications requiring fixed IP addresses.
            - Target groups for NLB can include EC2 instances or hardcoded private IP addresses, including servers in your own data center.
            - NLB can be placed in front of an Application Load Balancer (ALB) to combine fixed IP benefits with advanced HTTP routing capabilities.
            - Health checks for NLB target groups support TCP, HTTP, and HTTPS protocols, enabling flexible backend monitoring.
        - Gateway Load Balancer :
            - Gateway Load Balancer (GWLB) is used to deploy, scale, and manage fleets of third-party network appliances in AWS.
            - GWLB enables all network traffic to be inspected by virtual appliances such as firewalls or intrusion detection systems before reaching applications.
            - It operates at the network layer (Layer 3), acting as a transparent gateway and load balancer distributing traffic across virtual appliances.
            - GWLB uses the GENEVE protocol on port 6081 and supports target groups consisting of EC2 instances or private IP addresses.
        - Sticky Sessions :
            - Sticky sessions, or session affinity, ensure that a client’s requests are consistently routed to the same backend instance.
            - Sticky sessions can be enabled on Classic Load Balancer, Application Load Balancer, and Network Load Balancer using cookies.
            - There are two types of cookies for stickiness: application-based cookies generated by the target application, and duration-based cookies generated by the load balancer.
            - Enabling stickiness may cause load imbalance if some users have very sticky sessions.
        - Cross Zone Load Balancing :
            - Cross zone load balancing distributes traffic evenly across all registered EC2 instances in all availability zones.
            - Without cross zone load balancing, traffic is distributed only within each availability zone, potentially causing imbalance if EC2 instance counts differ.
            - Application Load Balancer enables cross zone load balancing by default with no inter-AZ data charges, while Network and Gateway Load Balancers have it disabled by default and may incur charges if enabled.
            - Classic Load Balancer is being retired and is not recommended for new deployments.
        - SSL Certs :
            - SSL certificates enable encrypted traffic between clients and load balancers, ensuring secure in-flight data transmission.
            - TLS is the modern version of SSL, but the term SSL is still commonly used for simplicity.
            - Server Name Indication (SNI) allows multiple SSL certificates to be hosted on a single load balancer, enabling support for multiple domains.
            - Application Load Balancers (ALB) and Network Load Balancers (NLB) support multiple SSL certificates with SNI, whereas Classic Load Balancers support only one SSL certificate.
        - Connection Draining :
            - Connection Draining is a feature that allows instances to complete active requests before deregistration.
            - It is called Connection Draining in Classic Load Balancers and Deregistration Delay in Application and Network Load Balancers.
            - The draining period can be configured between 1 and 3600 seconds, with a default of 300 seconds.
            - Setting a low draining time is suitable for short requests, while longer draining times accommodate long-lived requests but delay instance removal.

    - Auto Scaling Group :
        - Auto Scaling Groups (ASGs) automatically adjust the number of EC2 instances to match changing load.
        - ASGs support scaling out (adding instances) and scaling in (removing instances) within defined minimum and maximum capacities.
        - ASGs integrate with load balancers to distribute traffic and perform health checks, replacing unhealthy instances automatically.
        - Scaling policies can be triggered by CloudWatch alarms based on metrics like average CPU utilization, enabling automatic scaling.
        - Scaling Policies :
            - Auto Scaling Groups (ASGs) support various scaling policies: dynamic scaling (including target tracking and step scaling), scheduled scaling, and predictive scaling.
            - Target tracking scaling maintains a specified metric, such as CPU utilization, at a target value by automatically scaling out or in.
            - Step scaling uses CloudWatch alarms to trigger capacity changes based on defined thresholds.
            - Scheduled scaling anticipates scaling actions based on known usage patterns, while predictive scaling forecasts load using historical data to schedule scaling ahead of time.
            - Common metrics for scaling include CPU utilization, RequestCountPerTarget, network in/out, and custom application-specific metrics.
            - Scaling cooldown periods prevent rapid scaling actions by enforcing a wait time after each scaling event, typically five minutes.
            - Using ready-to-use AMIs and enabling detailed monitoring can reduce instance startup time and improve scaling responsiveness.
        - Instance Refresh :
            - Instance Refresh is a feature of Auto Scaling groups that allows updating all EC2 instances using a new launch template.
            - Instead of manually terminating instances and waiting for replacements, Instance Refresh automates the process of terminating old instances and launching new ones.
            - You can set a minimum healthy percentage to control how many instances can be replaced at a time during the refresh.
            - A warm-up time can be specified to ensure new instances have enough time to become ready before serving traffic.

    - An IAM instance profile is a container for an IAM role that you can attach to an EC2 instance, so the instance can assume that role and get temporary credentials to access AWS services.

- AWS Elastic Beanstalk : platform-as-a-service (PaaS) on AWS
    - Lets you deploy + manage web apps without worrying about the underlying infrastructure
    - You provide your code → EB does the rest
    - Deployment Structure :
        - Application : The container for versions and environments
        - Environment : An actual deployed instance of your app (dev, test, prod)
        - Environment Tier :
            - Web server (HTTP, front-facing apps)
            - Worker (background job processing)
    - Deployment Options : You can deploy via: ZIP file upload (via console or CLI), GitHub / CodePipeline, EB CLI, CI/CD
    - EB stores app versions in S3 under the hood
    - You can tweak all deployement options in the .ebextensions file, eb console, eb cli

- AWS Serverless Application Model : It’s a framework + syntax + tooling for building serverless apps on AWS
    - You can:
        - Define Lambda, API Gateway, DynamoDB, SQS, SNS, etc. in a simple YAML file
        - Build + test locally
        - Deploy with a single command
        - You must use an S3 bucket when deploying (SAM uploads artifacts there)

- AWS Lambda : Serverless compute service → run code without managing servers
    - You can only increase the memory of a lambda function
    - You write the code → Lambda handles provisioning, scaling, fault tolerance
    - Pay only for:
        - Invocation count
        - Execution duration
    - Execution Model :
        - Cold Start: First request spins up container (slower)
        - Warm Start: Reuses existing container (fast)
    - Timeout limit = 15 mins max
    - Use Layers to share code/dependencies
    - Use Environment Variables to config your function
    - A Lambda function alias is like a named pointer to a specific version of a Lambda function, allowing you to manage deployments (like dev/prod) and route traffic between versions
        - Example : my-function:PROD
    - lambda uses /tmp directory to store temporary files that will be deleted in the future
    - Access the unique request ID for each invocation using : context.aws_request_id
    - AWS Lambda Destinations let you define a success or failure target for asynchronous invocations.

# CONTAINERS

- AWS Copilot : CLI tool to deploy + manage containerized apps on AWS easily(ECR)
    - If you see a question about deploying Docker apps to ECS with minimal config, Copilot is the answer
    - Know it handles:
        - Docker build + push to ECR
        - Load balancer setup
        - IAM + VPC + ECS cluster creation
            - It simplifies CI/CD setup (CodePipeline)
            - Ideal for Docker devs who want AWS-native deployments

- Amazon Elastic Container Registry (Amazon ECR) : A fully managed Docker container registry for AWS
    - ECR supports lifecycle rules to delete old images (e.g., keep last 5, delete rest)

- Amazon ECS : AWS’s native container orchestration platform
    - Runs Docker containers on:
        - Fargate (serverless)
        - EC2 (self-managed VMs)
    - Cluster
        - A logical grouping of container instances or Fargate capacity
        - Can run on EC2 or Fargate
    - Task Definition : A blueprint for your app, Defines:
        - Docker image
        - CPU/memory
        - Env vars
        - Volumes
        - Networking
        - IAM roles
    - Task : A running instance of a task definition
    - Service : Keeps a specified number of tasks running
    - IAM with ECS :
        - Execution Role = ECS uses it to pull image from ECR, write logs, etc.
        - Task Role = Container assumes this role to access S3, DynamoDB, etc.

- Amazon EKS : AWS-managed Kubernetes
    - Runs upstream K8s clusters on AWS
    - If you see "Kubernetes YAML", "multi-cloud portability", or "use existing Helm chart" — it's EKS. If the dev wants “easy Docker deploys” or “simple backend” — go with ECS or Lambda.

# DATABASE

- Amazon Aurora : AWS’s high-performance relational DB, serverless option
    - compatible with MySQL and PostgresQL
    - Aurora has replication + failover + read scaling built-in
    - Aurora Global DB enables cross-region DR
    - Amazon RDS : managed relational DB service
    - Supports: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Aurora
    - Handles: Backups, Patching, Monitoring ,Scaling
    - Storage Auto Scaling allows automatic increase of storage based on usage thresholds.
    - RDS does not provide SSH access to underlying instances, emphasizing its managed nature.
    - DB Instance : A managed database server
    - DB Snapshot : Manual or automatic backups of your DB instance, Supports pointj-in-time restore
    - Read Replicas : Offload read traffic for scalability, Only available for MySQL, Postgres, MariaDB, and Aurora, Can be promoted to standalone DBs
    - Read Replicas = Scale reads, not HA
    - Multi-AZ = High Availability, not performance
    - RDS Read Replicas are used to scale read operations with asynchronous replication, supporting up to 15 replicas across availability zones or regions.
    - Read Replicas are eventually consistent and can be promoted to standalone databases for write operations.
    - RDS Multi-AZ deployments provide synchronous replication for disaster recovery with automatic failover, enhancing availability but not scaling reads.
    - Transitioning from Single AZ to Multi AZ is a zero downtime operation involving an automatic snapshot and standby database creation.
    - Amazon Aurora is a cloud-optimized, proprietary AWS database compatible with PostgreSQL and MySQL drivers.
    - Aurora storage automatically grows from 10GB up to 128TB, eliminating manual disk monitoring.
    - It maintains six copies of data across three Availability Zones, ensuring high availability and self-healing.
    - Aurora supports up to 15 read replicas with fast replication and failover, including writer and reader endpoints for seamless connection management.
    - Data at-rest encryption on RDS and Aurora is enabled using KMS and must be defined at database launch.
    - In-flight encryption is enabled by default, requiring clients to use AWS TLS root certificates.
    - Database authentication supports both username/password and IAM roles for enhanced security management.
    - Network access is controlled via security groups, and audit logs can be sent to CloudWatch Logs for long-term retention.
    - Amazon RDS Proxy pools and shares database connections to improve efficiency and reduce resource stress.
    - It is fully serverless, auto-scaling, and highly available across multiple Availability Zones.
    - RDS Proxy reduces failover time by up to 66% by managing failovers transparently.
    - It supports IAM authentication enforcement and securely stores credentials in AWS Secrets Manager.
    - Amazon ElastiCache provides managed Redis and Memcached caching services to reduce database load for read-intensive workloads.
    - Caches are in-memory databases offering high performance and low latency, enabling stateless applications by storing state externally.
    - Cache hit and miss strategies optimize data retrieval, but require careful cache invalidation to maintain data consistency.
    - Redis supports multi-availability zones, replication, persistence, and advanced data structures, while Memcached offers sharding and multi-threading but lacks replication and high availability.
    - Caching is generally safe but may lead to eventual consistency and stale data.
    - Lazy Loading (Cache-Aside) caches data only when requested, optimizing read performance.
    - Write Through updates the cache immediately when the database is updated, preventing stale data but incurring write penalties.
    - Cache Eviction and Time-to-Live (TTL) strategies manage cache size and data freshness effectively.
    - Amazon MemoryDB for Redis is a Redis-compatible, durable, in-memory database service.
    - Unlike Redis, which is primarily a cache with some durability, MemoryDB functions as a durable database with a Redis-compatible API.
    - MemoryDB offers ultra-fast performance, handling over 160 million requests per second, with Multi-AZ transaction logs for data durability.
    - It scales seamlessly from tens of gigabytes to hundreds of terabytes, ideal for web and mobile applications, online gaming, media streaming, and microservices requiring fast, durable in-memory data access.

- Amazon DynamoDB : managed NoSQL key-value + document DB
    - Serverless, scalable, fast AF (single-digit ms latency)
    - Use partition key (and optional sort key) for schema
    - Supports on-demand & provisioned throughput
    - DAX = in-memory cache, microsecond reads
    - DynamoDB Streams : Triggers on item-level changes (used w/ Lambda)
    - Global Tables for multi-region sync
    - lsi is querying for info from partion key and sort but gsi is query for info from other attributes
    - Transactions
    - Supports ACID transactions
    - Use TransactWriteItems and TransactGetItems
    - Max 25 operations per transaction
        - TTL (Time to Live)
    - Automatically deletes items past a certain timestamp
    - Doesn’t delete immediately — eventually consistent
    - Good for expiring session data, temp data
        - What is UnprocessedKeys?
    - When you use BatchGetItem or BatchWriteItem, DynamoDB may not process all requested items in a single request due to:
        - Throttling (hitting RCU/WCU limits)
        - Item size limits or request limits
        - Temporary internal issues
    - Those unprocessed items are returned in the UnprocessedKeys response object.
    - Retry strategy : Use exponential backoff + jitter(randomized delay)

- Amazon ElastiCache : fully managed in-memory data store
    - Redis → caching + pub/sub + leaderboard + session store, Ideal for frequently accessed, complex structured data (like leaderboard-style rankings or sorted sets)
    - Memcached → simple, fast caching, scalable, no persistence, does NOT support encryption at rest (only in transit).
    - Boosts performance, Reduces DB strain, Used in real-time apps
    - Use ElastiCache when data is ephemeral (cache)

- Amazon MemoryDB : managed in-memory DB, Redis-compatible
    - Use MemoryDB when you need in-memory performance + full durability
    - Multi-AZ cluster with synchronous writes
    - Stores data in in-memory nodes and durably replicates across AZs

# DEVELOPER TOOLS

- AWS Amplify : AWS’s toolkit + hosting platform for building full-stack apps FAST
    - You push code → Amplify builds → Deploys to global CDN → HTTPS + custom domain ready
    - Host web/mobile apps
    - Add backend features like auth, API, DB, storage
    - Support CI/CD + custom domains

- AWS CodePipeline :
    - CodePipeline = AWS’s fully managed CI/CD service
    - It automates the build → test → deploy steps for your code
    - [ Source ] → [ Build ] → [ Test (optional) ] → [ Deploy ]
    - Supports manual approvals

- AWS CodeBuild : CodeBuild = AWS’s fully managed build service
    - It compiles your source code, runs tests, produces build artifacts (like zip, jar, docker image).
    - Supports environment variables
    - Pricing is based on build duration
    - Can build Docker images & push to ECR
    - Integrates with CodePipeline, GitHub, CodeCommit

- AWS CodeDeploy : AWS’s fully managed deployment service
    - It automates deploying your app to:
        - EC2 instances
        - On-prem servers
        - Lambda functions
        - ECS services
    - Deployment Types :
        - In-place
        - Blue/Green
        - Canary
        - Linear
        - All-at-once
    - Add pre/post hooks via Lambda functions for: Smoke testing, Logging, Feature flags
    - In an AWS CodeDeploy in-place deployment, the hook run order is: ApplicationStop → BeforeInstall → AfterInstall → ApplicationStart → ValidateService.

- AWS CodeArtifact : fully managed artifact repository service
    - It stores build artifacts like:
        - Libraries
        - Packages
        - Dependencies

- AWS CodeGuru : AI-powered code reviewer + performance profiler

- AWS X-Ray : distributed tracing service, Helps you visualize, trace, and debug complex requests as they pass through multiple AWS services.
    - Service Map → Visual flow of services + latencies
    - Trace List → Every invocation/request with status
    - Annotations & Metadata → Add custom tags to traces (e.g. user ID, order ID)
    - Analytics → Filter traces (e.g. show all 5xx errors in last 1hr)

# MANAGEMENT AND GOVERNANCE

- AWS AppConfig : AWS service for managing & deploying application configuration safely
    - Lets you:
        - Manage app configs separately from code
        - Deploy configs gradually (think: canary, linear, all-at-once)
        - Monitor for errors & rollback if things break
    - Configs stored in hosted profile / S3 / SSM / SecretsManager

- AWS CloudTrail : service that records all AWS API calls & actions across your account
    - Stores logs in S3, sends to CloudWatch Logs

- AWS CloudFormation : Infrastructure as Code (IaC) service for AWS
    - Stacks : A deployed instance of a template (the real infra)
    - Change Sets : Preview updates to a stack before applying them
    - Rollback : Auto-rollback if deployment fails
    - Nested Stacks : Break large templates into smaller reusable ones
    - Drift Detection : See if someone changed the infra outside the template
    - StackSets : Deploy infra across multiple accounts/regions

        | Section      | What it does                                             |
        | ------------ | -------------------------------------------------------- |
        | `Parameters` | Take user input (e.g., region, name)                     |
        | `Resources`  | Define actual AWS resources                              |
        | `Outputs`    | Export info like bucket name, ARNs, etc.                 |
        | `Mappings`   | Region-specific values                                   |
        | `Conditions` | Create resources only if condition is met                |
        | `Metadata`   | Extra info, not used by engine                           |
        | `Transform`  | Include macros like `AWS::Serverless-2016-10-31` for SAM |

    - Useful Functions (aka "Intrinsics")
      | Function | Description |
      | ------------------------ | -------------------------------------- |
      | `!Ref` | Reference to a parameter/resource |
      | `!GetAtt` | Get attribute (like ARN) of a resource |
      | `!Sub` | String substitution (`${}`) |
      | `!Join` | Join strings |
      | `!If`, `!Equals`, `!Not` | Conditions |
      | `!ImportValue` | Cross-stack output import |
      | `!FindInMap` | Lookup in `Mappings` |

- Amazon CloudWatch : AWS’s monitoring + observability service
    - Metrics : Tracks performance (CPU, memory, request count, latency)
    - Logs : Collects logs from Lambda, ECS, EC2, apps, etc
    - Alarms : Sends alerts if metrics breach thresholds
    - Dashboards : Custom visualizations of metrics
    - Insights : Query logs (like SQL) for debugging
    - Events (now EventBridge) : Triggers on events like state changes

- Amazon CloudWatch Logs : centralized service to collect, store, monitor, and analyze log files from AWS resources + your apps
    - Log Group : Logical container for logs (e.g. /aws/lambda/my-func)
    - Log Stream : Sequence of logs from one resource instance (e.g. one Lambda instance)
    - Log Event : One single log line or message ({"timestamp": ..., "message": ...})
    - You can:
        - Export logs to S3 (long-term storage)
        - Stream logs to Elasticsearch/OpenSearch for advanced analytics
        - Forward logs to Lambda for custom processing
    - Retention is configurable (1 day → forever)
    - IAM required to read/write/stream logs

- AWS Systems Manager Parameter Store : Fully managed AWS service for storing config data + secrets (optional).
    - Stores key-value pairs for:
        - App configs (env vars, ARNs, URLs)
        - DB strings, API keys (via SecureString)
        - Passwords (via SecureString)

- AWS Cloud Development Kit : It lets you define AWS infrastructure using real programming languages (not YAML or JSON).

# NETWORK AND CONTENT DELIVERY

- Amazon CloudFront : AWS’s Content Delivery Network (CDN)
    - Can serve from S3, EC2, ALB, API Gateway, or custom origin
    - Custom error pages (404.html etc)

- Elastic Load Balancing (ELB) : AWS service that automatically distributes incoming traffic across multiple targets (EC2, containers, IPs, Lambda)
    - ALB is used for HTTP/HTTPS, path/host-based routing
    - NLB is for ultra-low-latency TCP/UDP use cases
    - ELB supports EC2, ECS, Lambda, IPs as targets
    - Health checks remove bad targets automatically
    - NLB supports TLS pass-through, static IPs, cross-zone load balancing
- Amazon Route 53 : AWS’s scalable and highly available Domain Name System (DNS) + health checking + domain registration service, Routing policies, Traffic control, Global failover.
    - Simple : Default — one record = one target
    - Weighted : A/B testing: 70% to us-east-1, 30% to us-west-2
    - Latency-based : Route users to the lowest-latency AWS region
    - Failover : Primary → Secondary if Primary health check fails
    - Geolocation : Route based on user location (e.g. India → Mumbai, US → Ohio)
    - Geoproximity : Weighted + bias by distance (Traffic Flow only)
    - Multivalue : Return multiple healthy IPs (basic load balancing)

- Amazon Virtual Private Cloud (Amazon VPC) : your own private network inside AWS, Control ingress/egress with firewall rules with security grps and NACLs
    - CIDR Block : The IP range for your VPC (e.g., 10.0.0.0/16)
    - Subnet : Smaller slices inside your VPC (can be public or private)
    - Route Table : Controls traffic routing between subnets + internet
    - Internet Gateway (IGW) : Enables internet access for public subnets
    - NAT Gateway : Lets private subnets talk out to internet securely
    - Security Group : Firewall attached to resources (stateful)
    - Network ACL : Firewall at subnet level (stateless)
    - VPC Peering : Connect 2 VPCs together
    - Endpoints : Private access to AWS services inside VPC
        - Gateway : S3, DynamoDB (route table-based)
            - Interface : Other AWS services (ENI-based)

- AWS API Gateway : It’s a fully managed service that lets you:
    - Create, publish, secure, and monitor REST, WebSocket, and HTTP APIs
    - Basically, you put API Gateway in front of your services (like Lambda, EC2, DynamoDB, etc.)
    - It handles routing, security, throttling, caching, etc.
    - Enforce authentication, rate limiting, monitoring, caching, and throttling
    - Stage Variables = key-value pairs per stage, Only for REST APIs, NOT HTTP APIs, Useful for multi-environment deployments
    - Amazon API Gateway supports mock integrations, which allow you to:
    - Simulate backend behavior without actually calling any backend service
    - Configure request and response mapping templates to return predefined responses
    - This is perfect for integration testing, development, or staging environments

# SECURITY, IDENTITY AND COMPLIANCE

- AWS Certificate Manager (ACM) : manage SSL/TLS certs for free, Automatically renews them, Deploys them to services like CloudFront, ALB, API Gateway, ELB, etc,
    - Works with both public and private certificates
    - ACM checks that DNS record to verify domain ownership
    - if u want to load certs they have to be in us-east-1 even if ur app is in another region

- AWS IAM (Identity and Access Management) :
    - identity-based-policies ->
        - Users -> long-term creds(pass, access-keys), no permission by default
        - Groups -> collection of users, user can be in multiple grps, no nested grps
        - Policies -> JSON files that allow/deny actions on resources. types:aws managed, customer managed, inline. Explicit Deny > Allow & no allow = deny.
        - Roles -> Temporary crud via STS for AWS services, cross-account, federation
    - resource-based-policies -> resource policies for AWS services that defines what services can access what services & how regardless of who's using it.
    - IAM credentials report -> provides an account-level overview of all users and their credential statuses.
    - IAM access adviser -> offers user-level insights into granted service permissions and their last accessed times.
    - Shared Responsibility Model ->
        - AWS is responsible for infrastructure, global network security, service configuration, vulnerability analysis, and compliance.
        - Users are responsible for managing IAM components such as users, groups, roles, policies, and their monitoring.

- AWS Private Certificate Authority (Private CA) : create your own private certsf

- AWS WAF (Web Application Firewall) : protects your web apps and APIs from common attacks by filtering HTTP/S traffic at the edge or app layer.

- Amazon Cognito : Fully managed service for authenticating, authorizing, and managing users for your apps.
    - It’s your app’s built-in login system — no need to roll your own auth
    - Cognito can:
        - Sign up / sign in users
        - Handle password resets, MFA, email/phone verification
        - Integrate with third-party IdPs (Google, Facebook, SAML, OIDC)
        - Issue JWT tokens (ID, access, refresh)
    - Cognito Identity Pools : Give users access to AWS services — securely
    - Cognito User Pools : Manage your users like user sign up / sign in

- AWS STS (Security Token Service) : It gives you temporary security credentials (access key, secret key, session token) to IAM users, Federated users (e.g., Google, SAML, Cognito), Cross-account roles, Applications needing short-term AWS access
    - so:
        - You can securely access AWS resources
        - You don’t need to hard-code long-term IAM keys
    - AssumeRole : Assume a role from same or another account (most common!)
    - AssumeRoleWithWebIdentity = for Cognito/mobile apps
    - GetSessionToken = add MFA to sessions

- AWS KMS(Key Management Service) : A managed service that enables you to easily encrypt your data. KMS provides a highly available key storage, management, and auditing solution for you to encrypt data within your own applications and control the encryption of stored data across AWS services.
    - AWS-managed CMKs (free, automatic, limited control)(can’t modify their policies, share them across accounts, or see detailed usage logs)
    - Customer-managed CMKs
        - (full control, logging, rotation)
            - (Supports cross-account access)

- AWS Secrets Manager : Fully managed AWS service for storing, managing, and rotating secrets securely. Secrets = API keys, DB creds, tokens, OAuth secrets, private keys, etc.

# STORAGE

- Amazon Elastic Block Store (EBS) : block storage for EC2, Snapshots are backups of EBS volumes stored in S3
- Amazon Elastic File System (EFS) : fully managed NFS file system
- Amazon S3 (Simple Storage Service) : object storage for any data type
  _ Bucket : Top-level folder — where your files live (global namespace)
  _ Object : The actual file you store (has key + metadata + data)
  _ Key : File name inside the bucket (like img/logo.png)
  _ Region : Bucket is created in one AWS region \* Prefix : Path inside the key (like a folder structure)
    - Versioning — keep multiple versions of the same file
    - Lifecycle Rules — move files to IA/Glacier after N days
    - Event Notifications — trigger Lambda/SNS/SQS on upload/delete
    - Static Website Hosting — serve HTML/CSS/JS from your bucket
    - Presigned URLs — temporary access to private files
    - Multi-part Upload — break big files into chunks (e.g. 10 GB video)
    - Cross-Region Replication — copy files to another region auto-magically
    - Requester Pays — force downloader to pay the cost
    - An S3 Access Point is a custom endpoint with its own access policy that simplifies and controls access to an S3 bucket for specific users, applications, or VPCs.
- Amazon S3 Glacier : low-cost archival storage
