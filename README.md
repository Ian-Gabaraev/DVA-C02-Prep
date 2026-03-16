# AWS DVA-C02 Study Notes

## Table of Contents

| Section | Topics |
|---------|--------|
| [AWS Global Infrastructure](#aws-global-infrastructure) | Regions, AZs, Edge Locations |
| [IAM](#iam-identity--access-management) | Users, Groups, Policies, Roles |
| [EC2](#ec2-elastic-compute-cloud) | Instance Types, Purchasing, Security Groups |
| [AMI](#ami-amazon-machine-image) | Custom AMIs, Cross-region |
| [EBS](#ebs-elastic-block-store) | Volume Types, Snapshots, Multi-Attach |
| [EFS](#efs-elastic-file-system) | Performance Modes, Storage Tiers |
| [Storage Comparison](#ebs-vs-efs-vs-instance-store) | EBS vs EFS vs Instance Store |
| [ELB & ASG](#elb--asg-load-balancing--auto-scaling) | ALB, NLB, GLB, Health Checks, Scaling |
| [RDS & Aurora](#rds-relational-database-service) | Read Replicas, Multi-AZ, Aurora, Proxy |
| [ElastiCache](#aws-elasticache) | Redis vs Memcached, Caching Strategies |
| [S3](#s3-simple-storage-service) | Storage Classes, Security, Replication, Performance |
| [Lambda](#aws-lambda) | Invocations, Concurrency, Layers, VPC |
| [API Gateway](#api-gateway) | Endpoints, Integrations, Security, Caching |
| [DynamoDB](#dynamodb) | Capacity, Indexes, Streams, DAX |
| [SQS](#sqs-simple-queue-service) | Standard/FIFO, Visibility, DLQ |
| [SNS](#sns-simple-notification-service) | Fan-out, Filtering, FIFO |
| [Kinesis](#kinesis) | Streams, Firehose, Analytics |
| [Step Functions](#step-functions) | State Machines, Workflows |
| [Containers](#ecs-fargate--ecr-containers) | ECS, Fargate, ECR |
| [CloudFormation](#cloudformation) | Templates, Functions, Stacks |
| [SAM](#aws-sam-serverless-application-model) | Serverless Templates, CLI |
| [CI/CD](#cicd-codecommit-codebuild-codedeploy-codepipeline) | CodeCommit, CodeBuild, CodeDeploy, CodePipeline |
| [CloudWatch](#cloudwatch) | Metrics, Logs, Alarms |
| [X-Ray](#x-ray) | Distributed Tracing, Sampling |
| [Cognito](#cognito) | User Pools, Identity Pools |
| [KMS](#kms-key-management-service) | Encryption, Key Types, Envelope Encryption |
| [Secrets & Parameters](#secrets-manager--ssm-parameter-store) | Secrets Manager, SSM Parameter Store |
| [EventBridge](#eventbridge) | Event Bus, Rules, Targets |
| [Elastic Beanstalk](#elastic-beanstalk) | Deployment Policies, Extensions |
| [Self-Exam Questions](#self-exam-questions) | 100+ questions across all DVA-C02 topics |

**[Take Interactive Quiz](quiz.html)** — Test your knowledge with an interactive quiz!

---

## AWS Global Infrastructure

### Regions

A **Region** is a geographic area with multiple data centers (e.g., `us-east-1`, `eu-west-2`).

| Consideration | Description |
|---------------|-------------|
| **Compliance** | Data may need to stay in specific countries |
| **Latency** | Deploy closer to users for better performance |
| **Service availability** | Not all services available in all regions |
| **Pricing** | Varies by region |

### Availability Zones (AZs)

Each Region has **2-6 AZs** (e.g., `us-east-1a`, `us-east-1b`). Each AZ = one or more discrete data centers with independent power, networking, and connectivity.

| Key Point | Detail |
|-----------|--------|
| **Isolation** | AZs are physically separated (disaster protection) |
| **Low latency** | Connected via high-bandwidth, low-latency networking |
| **HA design** | Distribute resources across AZs for fault tolerance |

### Edge Locations & Global Services

| Concept | Description |
|---------|-------------|
| **Edge Locations** | CDN endpoints for CloudFront (200+ worldwide) |
| **Global services** | IAM, Route 53, CloudFront, WAF (not region-specific) |
| **Regional services** | EC2, RDS, EBS, etc. (bound to a region) |

> 💡 **Exam tip:** Know which services are global vs regional. IAM is global; EC2 and EBS are regional; EBS is AZ-specific.

---

## IAM (Identity & Access Management)

> **Global service** — not region-specific

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Users** | Individual identities, can belong to multiple groups |
| **Groups** | Collections of users (cannot nest groups) |
| **Policies** | JSON documents defining permissions |
| **Inline Policy** | Policy attached directly to a user (no group needed) |

### Policy Structure

```json
{
  "Version": "2012-10-17",    // Policy language version
  "Statement": [{
    "Sid": "StatementId",     // Optional identifier
    "Effect": "Allow|Deny",
    "Principal": "arn:...",   // Account/user/role this applies to
    "Action": ["s3:Get*"],    // API actions
    "Resource": ["arn:..."]   // Target resources
  }]
}
```

### Roles

- Permissions for **services**, not users
- Attach policies to roles → assign roles to services (e.g., EC2)
- The service receiving the role = **trusted entity**

### Security Tools

| Tool | Purpose |
|------|---------|
| **Credentials Report** | CSV of all users + credential status |
| **Access Advisor** | Shows service access history per user |

---

## EC2 (Elastic Compute Cloud)

**EC2 encompasses:** Instances • EBS (drives) • ELB (load balancing) • ASG (auto-scaling)

### Configuration Options

- **OS:** Linux, Windows, MacOS
- **Compute:** CPU cores, RAM
- **Storage:** Network (EBS/EFS) or Hardware (Instance Store)
- **Network:** Card speed, public/private IP

**User Data** — Bootstrap script that runs at launch with root privileges. Use for updates, software installation, config.

---

### Security Groups

Acts as a **firewall** for EC2 instances.

| Rule Type | Default | Description |
|-----------|---------|-------------|
| **Inbound** | Blocked | Controls incoming traffic |
| **Outbound** | Allowed | Controls outgoing traffic |

> ⚠️ **Timeout = Security Group issue** — If you can't connect (SSH/HTTP/HTTPS), check SG first

**Key points:**

- Locked to **region + VPC**
- SGs can reference other SGs (e.g., allow traffic from instances with SG2)

### Common Ports

| Port | Protocol |
|------|----------|
| 22 | SSH / SFTP |
| 21 | FTP |
| 80 | HTTP |
| 443 | HTTPS |
| 3389 | RDP (Windows) |

---

### Instance Types

**Naming:** `m5.2xlarge` → **m** (class) + **5** (generation) + **2xlarge** (size)

| Type | Prefix | Use Case |
|------|--------|----------|
| **General Purpose** | t3, m5 | Balanced workloads |
| **Compute Optimized** | c5 | Batch processing, high-performance computing |
| **Memory Optimized** | r5 | In-memory databases, caching |
| **Storage Optimized** | i3 | High IOPS, data warehousing |
| **GPU** | p3 | ML/AI, graphics |
| **FPGA** | f1 | Custom hardware acceleration |

### Purchasing Options

| Option | Discount | Commitment | Best For |
|--------|----------|------------|----------|
| **On-Demand** | None | None | Short, unpredictable workloads |
| **Reserved** | Up to 72% | 1-3 years | Steady-state workloads |
| **Savings Plan** | Up to 72% | $/hour for 1-3 years | Flexible long workloads |
| **Spot** | Up to 90% | None (can be interrupted) | Batch jobs, fault-tolerant |
| **Dedicated Host** | Varies | Optional 1-3 year | Licensing, compliance |
| **Dedicated Instance** | Varies | None | Compliance, isolation |
| **Capacity Reservation** | None | Pay regardless of use | Guaranteed availability |

**On-Demand:** Linux/Windows billed per second (after 1st min), other OS per hour

**Reserved Instances:** Reserve specific attributes (type, region, OS). Pay upfront = more discount. Can sell on AWS Marketplace.

**Savings Plan:** Commit to $/hour spend, locked to instance family + region (e.g., m5 in us-east-1). Excess usage = On-Demand pricing.

**Spot:** Cheapest option. Lose instance when spot price > your bid. Never use for critical workloads.

**Dedicated Host vs Instance:**

- **Host** — Full server control, see sockets/cores (for BYOL licensing)
- **Instance** — Dedicated hardware, no host visibility, may share with same account

---

## AMI (Amazon Machine Image)

> **Region-specific** — must copy to use in another region

AMI = Pre-configured EC2 template (OS + software + config)

| AMI Type | Description |
|----------|-------------|
| **Public** | AWS-provided (Amazon Linux, Ubuntu, etc.) |
| **AWS Marketplace** | Third-party, often pre-configured software |
| **Custom** | Your own, built from an EC2 instance |

**Creating a Custom AMI:**

1. Launch EC2 → configure/install software
2. Stop instance (for data integrity)
3. Create AMI → creates EBS snapshots automatically
4. Launch new instances from your AMI

> 💡 AMIs speed up boot time since software is pre-baked, not installed via User Data

---

## EBS (Elastic Block Store)

Network-attached storage for EC2 — like a USB stick over the network.

**Key characteristics:**

- Bound to **one AZ**
- Attached to **one instance** at a time (except io1/io2 Multi-Attach)
- Network drive = some latency
- Persists independently of instance lifecycle

### EBS Snapshots

Backup mechanism for EBS volumes — can restore to any AZ.

| Feature | Description |
|---------|-------------|
| **Cross-AZ restore** | Snapshot in us-east-1a → restore in us-east-1b |
| **Recycle Bin** | Deleted snapshots recoverable (configurable retention) |
| **Fast Snapshot Restore** | No latency on first use, but expensive |

### EC2 Instance Store

Hardware-attached storage (physically on the host) — **not network-based**.

| Pros | Cons |
|------|------|
| Extremely high IOPS (millions) | **Ephemeral** — data lost on stop/terminate/hardware failure |
| Low latency (direct attached) | Cannot detach and reattach |
| Included in instance cost | Size tied to instance type |

**Use cases:** Buffer, cache, scratch data, temporary content

> ⚠️ **You are responsible for backups/replication** — AWS won't recover this data

### Delete on Termination

| Volume | Default Behavior |
|--------|------------------|
| Root EBS | **Deleted** on termination |
| Additional EBS | **Preserved** on termination |

Can be changed via console or CLI at launch time.

---

### EBS Volume Types

| Type | Category | IOPS | Throughput | Size | Boot? |
|------|----------|------|------------|------|-------|
| **gp3** | General SSD | 3,000–16,000 | 125–1,000 MiB/s | 1 GiB–16 TiB | ✅ |
| **gp2** | General SSD | 3 IOPS/GiB (max 16,000) | Linked to IOPS | 1 GiB–16 TiB | ✅ |
| **io2 Block Express** | Provisioned IOPS | Up to 256,000 | 4,000 MiB/s | 4 GiB–64 TiB | ✅ |
| **io1** | Provisioned IOPS | Up to 64,000 | 1,000 MiB/s | 4 GiB–16 TiB | ✅ |
| **st1** | Throughput HDD | Max 500 | 500 MiB/s | 125 GiB–16 TiB | ❌ |
| **sc1** | Cold HDD | Max 250 | 250 MiB/s | 125 GiB–16 TiB | ❌ |

> 💡 **Only SSD types (gp2/gp3/io1/io2) can be boot volumes**

**gp3 vs gp2:** gp3 allows independent IOPS/throughput scaling; gp2 links IOPS to size

**Provisioned IOPS (io1/io2):** For sustained IOPS needs — databases, critical apps. io2 Block Express offers sub-millisecond latency.

### EBS Multi-Attach

- **io1/io2 only** — attach same volume to multiple EC2 in same AZ
- Up to **16 instances** simultaneously
- Use case: clustered applications requiring shared storage

---

## EFS (Elastic File System)

Managed NFS that can be mounted on **multiple EC2 across multiple AZs**.

| Feature | Value |
|---------|-------|
| **Compatibility** | Linux only (POSIX) |
| **Scaling** | Automatic, up to petabytes |
| **Throughput** | Up to 10+ GB/s |
| **Pricing** | Pay per GB used |

### Performance Modes

| Mode | Use Case |
|------|----------|
| **General Purpose** | Latency-sensitive (web servers, CMS) |
| **Max I/O** | Higher latency, highly parallel (big data) |

### Throughput Modes

| Mode | Description |
|------|-------------|
| **Bursting** | Scales with storage size |
| **Provisioned** | Fixed throughput regardless of size |
| **Elastic** | Auto-scales based on workload (recommended) |

### Storage Tiers

| Tier | Cost | Access |
|------|------|--------|
| **Standard** | Higher | Frequent |
| **Infrequent Access (IA)** | Lower storage, pay per retrieval | Occasional |
| **Archive** | ~50% cheaper | Rare |

> 💡 Use **lifecycle policies** to auto-move files between tiers

### Availability

| Option | Description |
|--------|-------------|
| **Standard (Multi-AZ)** | Production, HA |
| **One Zone** | Dev/backup, cheaper, single AZ |

---

## EBS vs EFS vs Instance Store

| Feature | EBS | EFS | Instance Store |
|---------|-----|-----|----------------|
| **Attach to** | 1 instance (io1/io2: multi) | 100s of instances | 1 instance |
| **AZ scope** | Single AZ | Multi-AZ | Single AZ |
| **Persistence** | Persists | Persists | Ephemeral |
| **Use case** | Boot volumes, databases | Shared content, web serving | Cache, temp data |
| **Cost** | Per provisioned GB | Per used GB | Included |

## ELB & ASG (Load Balancing & Auto Scaling)

> **Terminology:** ELB (Elastic Load Balancing) is the **service name**, not a load balancer type. The actual LB types are ALB, NLB, GLB, and CLB.

### OSI Model Quick Reference

| Layer | Name | Protocol/Example | AWS LB |
|-------|------|------------------|--------|
| 7 | Application | HTTP, HTTPS, WebSocket | ALB |
| 4 | Transport | TCP, UDP, TLS | NLB |
| 3 | Network | IP, ICMP | GLB |
| 2 | Data Link | Ethernet, MAC | — |
| 1 | Physical | Cables, signals | — |

---

### Load Balancer Types

| Type | Layer | Protocols | Use Case |
|------|-------|-----------|----------|
| **ALB** | 7 | HTTP, HTTPS, WebSocket | Web apps, microservices |
| **NLB** | 4 | TCP, UDP, TLS | Extreme performance, static IP |
| **GLB** | 3 | IP (GENEVE) | Firewalls, packet inspection |
| **CLB** | 4/7 | HTTP, HTTPS, TCP, SSL | Legacy (avoid) |

> ⚠️ **CLB** = Classic Load Balancer, sometimes called "Classic ELB" — adds to the ELB naming confusion. Avoid for new projects.

---

### ELB Health Checks

LB periodically pings targets to verify they're healthy.

| Setting | Description |
|---------|-------------|
| **Protocol** | HTTP, HTTPS, TCP |
| **Path** | e.g., `/health` (HTTP/HTTPS only) |
| **Interval** | Time between checks (default: 30s) |
| **Threshold** | Consecutive successes/failures to change state |
| **Timeout** | Time to wait for response |

> ⚠️ **ELB does NOT terminate unhealthy targets** — it only stops routing traffic to them

### ASG + ELB Health Checks

ASG can use ELB health status to decide when to terminate/replace instances.

| Health Check Type | Default | Termination Trigger |
|-------------------|---------|---------------------|
| **EC2** | ✅ Yes | Instance stopped, impaired, or terminated |
| **ELB** | ❌ No | Target fails LB health check |

> 💡 Enable ELB health checks on ASG for automatic replacement of app-level failures

---

### Application Load Balancer (ALB)

Layer 7 (HTTP) — routes to **target groups**:

| Target Type | Example |
|-------------|--------|
| EC2 instances | i-0123... |
| Lambda functions | my-function |
| Private IPs | On-prem servers |

**Routing rules based on:**

- URL path (`/api/*`, `/images/*`)
- Hostname (`api.example.com`)
- Query strings (`?platform=mobile`)
- HTTP headers

**Key points:**

- Fixed DNS hostname (no static IP)
- Client IP in `X-Forwarded-For` header
- WebSocket support

---

### Network Load Balancer (NLB)

Layer 4 (TCP/UDP) — **highest performance** LB.

| Feature | Value |
|---------|-------|
| Performance | Millions of requests/sec |
| Latency | ~100ms (vs ~400ms ALB) |
| Static IP | One per AZ |

**Target groups:** EC2 instances, Private IPs, ALB (NLB → ALB combo)

**NLB provides:**

- Static hostname
- Static IP (one per AZ)
- Elastic IP support

> 💡 **When to use NLB:** Gaming servers, IoT backends, financial trading platforms — anywhere you need ultra-low latency, millions of requests/sec, or must whitelist a static IP for clients/firewalls.

### ALB vs NLB Routing

| Routing By | ALB | NLB |
|------------|-----|-----|
| URL path | ✅ | ❌ |
| Hostname | ✅ | ❌ |
| Query strings | ✅ | ❌ |
| HTTP headers | ✅ | ❌ |
| Port | ✅ | ✅ |

> NLB = Layer 4 (sees packets, not HTTP). ALB = Layer 7 (sees HTTP content). **Content-based routing → ALB. Static IP + performance → NLB.**

---

### Gateway Load Balancer (GLB)

Layer 3 (IP) — for **network appliances** (firewalls, IDS, packet inspection).

**Flow:** Traffic → GLB → Security appliances → GLB → Your app

| Feature | Detail |
|---------|--------|
| Protocol | GENEVE (port 6081) |
| Use case | Third-party virtual appliances |
| Layer | 3 (Network) |

> GENEVE encapsulates packets in UDP for cross-host VM/container communication

---

### Sticky Sessions (Session Affinity)

Same client always routed to same target instance.

| Cookie Type | Who Creates | Cookie Name |
|-------------|-------------|-------------|
| **Duration-based** | ALB | `AWSALB` (reserved) |
| **Application-based (LB)** | ALB | `AWSALBAPP` (reserved) |
| **Application-based (App)** | Your app | Custom (e.g., `SESSIONID`) |

> ⚠️ `AWSALB*` names are **AWS-reserved** — cannot be used by your app

> 💡 Use for stateful apps; avoid if possible (prefer stateless + external session store)

---

### Cross-Zone Load Balancing

Distributes traffic evenly across all targets in all AZs, regardless of AZ distribution.

| LB Type | Default | Cost |
|---------|---------|------|
| **ALB** | Enabled | Free |
| **NLB** | Disabled | Charged |
| **GLB** | Disabled | Charged |

> Without cross-zone: If AZ-1 has 2 instances and AZ-2 has 8, each AZ gets 50% of traffic (unfair distribution)

---

### SSL/TLS & SNI

**SSL Termination:** LB decrypts HTTPS traffic, forwards HTTP to targets (offloads CPU from instances).

| Concept | Description |
|---------|-------------|
| **SSL Certificate** | Loaded on LB via ACM (AWS Certificate Manager) |
| **SNI (Server Name Indication)** | Allows multiple SSL certs on one LB — client indicates hostname, LB selects correct cert |

**SNI Support:**

- ✅ ALB, NLB (multiple certs)
- ❌ CLB (one cert only)

> 💡 Use **ACM** for free, auto-renewing public certificates

---

### Connection Draining / Deregistration Delay

Time allowed for in-flight requests to complete when a target is deregistering or unhealthy.

| Setting | Default | Range |
|---------|---------|-------|
| **Deregistration Delay** | 300s (5 min) | 0–3600s |

> 💡 Set to **0** for short-lived requests; increase for long uploads/connections

---

## Auto Scaling Group (ASG)

Automatically adjusts EC2 capacity to match demand. **ASG is free** — you pay only for instances.

### Capacity Settings

| Setting | Description |
|---------|-------------|
| **Minimum** | Never go below this |
| **Desired** | Target number of instances |
| **Maximum** | Never exceed this |

### Launch Template

Defines what to launch:

| Setting | Example |
|---------|---------|
| AMI | ami-0123456789 |
| Instance Type | t3.micro |
| IAM Role | MyEC2Role |
| Security Groups | sg-web |
| User Data | Bootstrap script |
| Key Pair | my-key |
| EBS Volumes | gp3, 20 GiB |

> 💡 CloudWatch alarms can trigger ASG scale-out/in based on metrics (CPU, RAM, custom)

### ASG Scaling Policies

| Policy | Description | Example |
|--------|-------------|---------|
| **Target Tracking** | Maintain a target metric value | Keep avg CPU at 40% |
| **Step Scaling** | Scale based on threshold ranges | CPU > 70% → +2, CPU < 30% → -1 |
| **Scheduled** | Scale at specific times | Add 3 instances every Friday 5PM |
| **Predictive** | ML-based forecasting | Pre-scale for predicted daily peaks |

- **Predictive Scaling** — ML analyzes historical load patterns, pre-provisions capacity ahead of predicted spikes. Great for recurring patterns (daily/weekly cycles).

### Scaling Metrics

| Metric | Best For |
|--------|----------|
| **CPUUtilization** | Compute-bound apps |
| **RequestCountPerTarget** | Web servers behind ALB |
| **NetworkIn/Out** | Network-bound apps |
| **Custom (CloudWatch)** | App-specific (queue depth, etc.) |

---

### ASG Cooldown

Prevents rapid successive scaling actions. Default: **300 seconds**.

> 💡 Use shorter cooldown with faster-booting AMIs; longer for slow startup apps

---

### ASG Instance Refresh

Rolling update when you change Launch Template — replaces instances gradually.

| Setting | Description |
|---------|-------------|
| **Min Healthy %** | % of instances that must stay running (e.g., 90%) |
| **Warm-up** | Time before new instance counts as healthy |

> 💡 Enables zero-downtime deployments for Launch Template changes

## RDS (Relational Database Service)

Managed relational database — AWS handles patching, backups, scaling, HA, monitoring.

| Feature | Included |
|---------|----------|
| OS/DB patching | ✅ |
| Automated backups | ✅ |
| Multi-AZ failover | ✅ |
| Read replicas | ✅ (up to 15) |
| Encryption (at-rest & in-flight) | ✅ |
| Performance Insights | ✅ |

It supports MySQL, Postgres, Oracle, MariaDB, MS SQL Server, Aurora.

> ⚠️ **No SSH access** to the underlying instance

### Storage Auto Scaling

RDS automatically increases storage when running low. Set `MaxStorageThreshold` to cap it.

---

### Read Replicas vs Multi-AZ

| Feature | Read Replicas | Multi-AZ |
|---------|---------------|----------|
| **Purpose** | Read scaling | Disaster recovery |
| **Replication** | ASYNC (eventually consistent) | SYNC (immediate) |
| **Readable?** | ✅ Yes | ❌ Standby only |
| **Cross-region?** | ✅ Yes (with cost) | ❌ Same region |
| **Failover** | Manual (promote to standalone) | Automatic |
| **Max count** | 15 | 1 standby |

**Cost:** Same-region RR replication = free. Cross-region = network charges.

**Multi-AZ setup:** Enable in console → snapshot taken → restored to standby AZ → sync begins. **Zero downtime.**

> 💡 Read replicas can also be Multi-AZ (common exam question)

---

### Amazon Aurora

AWS-built relational DB, compatible with **MySQL** and **PostgreSQL**.

| Feature | Value |
|---------|-------|
| Performance | 5x MySQL, 3x PostgreSQL |
| Storage | Auto-scales 10 GB → 128 TiB |
| Replicas | Up to 15 (faster replication than RDS) |
| Failover | < 30 seconds |
| Copies | 6 copies across 3 AZs |
| Cost | ~20% more than RDS |

**Self-healing:** Corrupted data blocks repaired via peer-to-peer replication.

### Aurora Endpoints

| Endpoint | Purpose |
|----------|--------|
| **Writer Endpoint** | Always points to current master (for writes) |
| **Reader Endpoint** | Load-balanced across all read replicas |
| **Custom Endpoint** | Route to specific subset of instances |

> 💡 Use Writer for writes, Reader for reads — endpoints auto-update on failover

---

### RDS & Aurora Security

| Layer | Implementation |
|-------|----------------|
| **At-rest encryption** | KMS key at launch (encrypts master + replicas + snapshots) |
| **In-flight encryption** | TLS by default (use AWS TLS root certs) |
| **Authentication** | Username/password OR IAM DB authentication |
| **Network** | Security groups control access |

> To encrypt an unencrypted DB: snapshot → copy with encryption → restore

---

### RDS Proxy

Serverless connection pooler in front of RDS/Aurora.

| Benefit | Description |
|---------|-------------|
| **Connection pooling** | Reduces DB load from many connections |
| **Failover** | Reduces failover time by 66% |
| **IAM auth** | Enforce IAM authentication |
| **VPC only** | Never publicly accessible |

> 💡 Great for Lambda → RDS (Lambda opens many short-lived connections)

---

---

## AWS ElastiCache

Managed in-memory caching — **Redis** or **Memcached**.

### Redis vs Memcached

| Feature | Redis | Memcached |
|---------|-------|----------|
| **Multi-AZ** | ✅ | ❌ |
| **Auto Failover** | ✅ | ❌ |
| **Replication** | ✅ | ❌ |
| **Persistence** | ✅ | ❌ |
| **Backup & Restore** | ✅ | ✅ |
| **Data structures** | Complex (lists, sets, sorted sets) | Simple key-value |
| **Sharding** | Cluster mode | Multi-node |

> 💡 **Exam tip:** Use **Redis** for HA, persistence, complex data. Use **Memcached** for simple caching, multi-threaded, horizontal scaling.

---

### Caching Considerations

| Question | Consider |
|----------|----------|
| **Safe to cache?** | What if stale data causes security/business issues? |
| **Effective?** | Best for slow-changing, frequently-read data |
| **Structure fit?** | Key-value lookups work best; complex joins may not |
| **TTL strategy?** | How long before data expires? |

---

### Caching Design Patterns

#### Lazy Loading (Cache-Aside)

```
App → Cache (miss?) → DB → Cache → App
```

| Pros | Cons |
|------|------|
| Only requested data cached | Cache miss = 3 network calls |
| Node failure not fatal | Stale data possible |
| Simple to implement | Must handle cache invalidation |

---

#### Write-Through

```
App → DB + Cache (write both)
```

| Pros | Cons |
|------|------|
| Cache always current | Write penalty (2 writes) |
| No stale data | Cache churn (data may never be read) |
| | Missing data until first write |

> 💡 Combine **Write-Through + Lazy Loading** for best results

---

#### TTL (Time-To-Live)

Set expiration on cached items. Balance between:
- **Short TTL** — Fresh data, more cache misses
- **Long TTL** — Fewer misses, risk of stale data

---

#### Write-Behind (Write-Back)

```
App → Cache → (async) → DB
```

| Pros | Cons |
|------|------|
| Fast writes (async to DB) | Data loss risk if cache fails |
| Reduces DB load | Complex to implement |
| Good for write-heavy workloads | Eventually consistent |

---

#### Read-Through

```
App → Cache (auto-fetches from DB on miss)
```

Cache sits between app and DB. On miss, cache itself fetches from DB and stores. Simpler app logic, but requires cache to understand DB.

---

### ElastiCache Use Cases

| Use Case | Pattern |
|----------|--------|
| Session storage | Redis with TTL |
| Database query caching | Lazy Loading + TTL |
| Real-time leaderboards | Redis Sorted Sets |
| Pub/Sub messaging | Redis Pub/Sub |
| Rate limiting | Redis counters with TTL |

---

## S3 (Simple Storage Service)

Object storage with **unlimited storage**, highly durable (99.999999999% — 11 9s).

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Bucket** | Container for objects, globally unique name |
| **Object** | File + metadata, identified by key (full path) |
| **Key** | Full path including "folders" (e.g., `images/2024/photo.jpg`) |
| **Max object size** | 5 TB (use multipart upload for >100 MB, required >5 GB) |

### Storage Classes

| Class | Durability | Availability | Use Case |
|-------|------------|--------------|----------|
| **S3 Standard** | 11 9s | 99.99% | Frequently accessed data |
| **S3 Intelligent-Tiering** | 11 9s | 99.9% | Unknown/changing access patterns |
| **S3 Standard-IA** | 11 9s | 99.9% | Infrequent access, rapid retrieval |
| **S3 One Zone-IA** | 11 9s | 99.5% | Infrequent, non-critical, reproducible |
| **S3 Glacier Instant** | 11 9s | 99.9% | Archive, millisecond retrieval |
| **S3 Glacier Flexible** | 11 9s | 99.99% | Archive, minutes to hours retrieval |
| **S3 Glacier Deep Archive** | 11 9s | 99.99% | Long-term archive, 12-48 hour retrieval |

> 💡 Use **Lifecycle Policies** to automatically transition objects between classes

### S3 Security

| Layer | Mechanism |
|-------|-----------|
| **User-based** | IAM policies |
| **Resource-based** | Bucket policies (JSON), Object ACLs, Bucket ACLs |
| **Encryption** | SSE-S3, SSE-KMS, SSE-C, client-side |

**Bucket Policy Structure:**

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::my-bucket/*"
  }]
}
```

### S3 Encryption

| Type | Key Management | Use Case |
|------|----------------|----------|
| **SSE-S3** | AWS managed | Default encryption |
| **SSE-KMS** | KMS key | Audit trail, fine control |
| **SSE-C** | Customer-provided | Full key control |
| **Client-side** | Encrypt before upload | Maximum control |

> 💡 **SSE-KMS** has API call limits (quota). For high throughput, consider SSE-S3.

### S3 Versioning

- Enable at bucket level
- Protects against unintentional deletes (delete marker, not actual delete)
- Once enabled, can only be suspended (not disabled)
- `null` version ID for objects uploaded before versioning

### S3 Replication

| Type | Description |
|------|-------------|
| **CRR** (Cross-Region) | Compliance, lower latency, replication across accounts |
| **SRR** (Same-Region) | Log aggregation, live replication between prod/test |

**Requirements:** Versioning enabled on both buckets, proper IAM permissions

> ⚠️ Only new objects replicated after enabling. Use **S3 Batch Replication** for existing objects.

### S3 Event Notifications

Trigger actions on bucket events (PUT, DELETE, etc.):

| Target | Use Case |
|--------|----------|
| **SNS** | Fan-out to multiple subscribers |
| **SQS** | Queue for processing |
| **Lambda** | Real-time processing |
| **EventBridge** | Advanced filtering, multiple destinations |

### S3 Performance

| Feature | Description |
|---------|-------------|
| **Multi-part upload** | Parallelize uploads, recommended >100 MB |
| **Transfer Acceleration** | Use CloudFront edge locations for faster uploads |
| **Byte-range fetches** | Parallelize downloads by requesting byte ranges |
| **S3 Select / Glacier Select** | Retrieve subset of data using SQL |

**Baseline:** 3,500 PUT/COPY/POST/DELETE and 5,500 GET/HEAD requests per second **per prefix**.

### S3 Pre-signed URLs

Temporary access to private objects without changing bucket policy.

```bash
aws s3 presign s3://bucket/object --expires-in 3600
```

| Parameter | Description |
|-----------|-------------|
| **Expires** | 1 second to 7 days (default: 1 hour) |
| **Permissions** | Inherits permissions of the user who generated it |

---

## AWS Lambda

Serverless compute — run code without managing servers.

### Key Limits

| Limit | Value |
|-------|-------|
| **Memory** | 128 MB – 10,240 MB (10 GB) |
| **Timeout** | Max 15 minutes (900 seconds) |
| **Environment variables** | 4 KB total |
| **/tmp storage** | 512 MB – 10,240 MB |
| **Deployment package** | 50 MB zipped, 250 MB unzipped |
| **Concurrent executions** | 1,000 default (can increase) |
| **Layers** | Up to 5 per function |

> 💡 CPU scales proportionally with memory. More memory = more CPU = faster execution.

### Lambda Invocation Types

| Type | Behavior | Retries | Examples |
|------|----------|---------|----------|
| **Synchronous** | Caller waits for response | None (caller handles) | API Gateway, SDK |
| **Asynchronous** | Fire and forget | 2 retries (3 total) | S3, SNS, EventBridge |
| **Event Source Mapping** | Lambda polls source | Depends on source | SQS, Kinesis, DynamoDB Streams |

### Asynchronous Invocation

```
Event → Lambda (internal queue) → [Retry 1] → [Retry 2] → DLQ/Destination
```

| Setting | Description |
|---------|-------------|
| **Retries** | 2 retries with exponential backoff |
| **DLQ** | Dead Letter Queue (SQS or SNS) for failed events |
| **Destinations** | Route success/failure to SQS, SNS, Lambda, or EventBridge |

> 💡 **Destinations** are preferred over DLQ — more features, supports success events

### Event Source Mapping

Lambda polls from:

| Source | Behavior |
|--------|----------|
| **SQS** | Batch processing, long polling |
| **SQS FIFO** | Lambda scales to # of message groups |
| **Kinesis/DynamoDB Streams** | Process in order per shard |

**Error Handling:**

- Entire batch fails if one record fails
- Options: discard, retry, split batch, send to DLQ/destination

### Lambda in VPC

By default, Lambda runs in AWS-managed VPC (has internet). To access private resources:

1. Configure VPC, subnets, security groups
2. Lambda creates ENIs in your subnets
3. Use NAT Gateway for internet access from private subnet

> ⚠️ Lambda in VPC has no internet unless you have NAT Gateway

### Lambda Concurrency

| Type | Description |
|------|-------------|
| **Unreserved** | Shared pool, up to account limit |
| **Reserved** | Guaranteed minimum for a function |
| **Provisioned** | Pre-initialized instances, no cold start |

**Cold Start:** First invocation initializes execution environment (can add seconds). Provisioned concurrency eliminates cold starts.

### Lambda Layers

Share code/dependencies across functions:

```
Function → Layer 1 (libs) → Layer 2 (common code)
```

- Up to 5 layers per function
- Total unzipped size < 250 MB
- Use for: common libraries, custom runtimes

### Lambda@Edge / CloudFront Functions

| Type | Location | Max Duration | Use Case |
|------|----------|--------------|----------|
| **CloudFront Functions** | Edge locations | < 1 ms | Simple request/response manipulation |
| **Lambda@Edge** | Regional edge cache | 5-30 seconds | Complex logic, external calls |

---

## API Gateway

Managed API service — create, publish, secure, and monitor APIs.

### Endpoint Types

| Type | Description |
|------|-------------|
| **Edge-optimized** | Routed through CloudFront (default) |
| **Regional** | For clients in same region |
| **Private** | Accessible only from VPC via VPC endpoint |

### API Types

| Type | Features | Cost |
|------|----------|------|
| **REST API** | Full features (caching, API keys, usage plans, request validation) | Higher |
| **HTTP API** | Simpler, faster, JWT auth only | ~70% cheaper |
| **WebSocket API** | Real-time two-way communication | Per message |

### Integration Types

| Type | Description |
|------|-------------|
| **Lambda Proxy** | Request passed as-is to Lambda, Lambda returns full response |
| **Lambda Custom** | Transform request/response with mapping templates |
| **HTTP Proxy** | Pass through to HTTP endpoint |
| **HTTP Custom** | Transform with mapping templates |
| **AWS Service** | Direct integration with AWS services |
| **Mock** | Return response without backend |

> 💡 **Lambda Proxy** is most common — simplest setup, Lambda controls response format

### API Gateway Security

| Method | Description |
|--------|-------------|
| **IAM** | AWS Sig v4, good for internal/AWS clients |
| **Lambda Authorizer** | Custom auth logic (JWT, OAuth, etc.) |
| **Cognito User Pools** | JWT validation with Cognito |
| **API Keys + Usage Plans** | Rate limiting per client |

### Stages and Deployment

| Concept | Description |
|---------|-------------|
| **Stage** | Named reference to deployment (dev, prod, v1) |
| **Stage Variables** | Key-value pairs, like environment variables |
| **Canary Deployment** | Route % of traffic to new deployment |

### Throttling

| Limit | Value |
|-------|-------|
| **Account limit** | 10,000 requests/second |
| **Per-stage limit** | Configurable |
| **Per-client (Usage Plans)** | API key-based throttling |

> **429 Too Many Requests** when throttled. Client should retry with exponential backoff.

### Caching

- Cache responses at stage level
- TTL: 0-3600 seconds (default: 300)
- Cache size: 0.5 GB – 237 GB
- Cache key: method + resource path (can include headers/query params)

> 💡 Reduce backend calls, improve latency. Invalidate with `Cache-Control: max-age=0` header.

---

## DynamoDB

Fully managed NoSQL database — millisecond latency at any scale.

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Table** | Collection of items |
| **Item** | Row (max 400 KB) |
| **Attribute** | Column (nested up to 32 levels) |
| **Primary Key** | Partition key (required) + optional sort key |

### Primary Key Options

| Type | Components | Use Case |
|------|------------|----------|
| **Partition key** | Single attribute | Unique identifier |
| **Composite** | Partition + Sort key | One-to-many relationships |

> 💡 Choose partition key with high cardinality for even distribution

### Capacity Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| **Provisioned** | Set RCU/WCU, auto-scaling available | Predictable workloads |
| **On-Demand** | Pay per request | Unpredictable, new tables |

**Throughput units:**

| Unit | Capacity |
|------|----------|
| **1 RCU** | 1 strongly consistent read/sec (4 KB) OR 2 eventually consistent |
| **1 WCU** | 1 write/sec (1 KB) |

### Read Consistency

| Type | Description |
|------|-------------|
| **Eventually consistent** | Default, might return stale data |
| **Strongly consistent** | Returns most recent data, uses 2x RCU |

### Secondary Indexes

| Type | Partition Key | Sort Key | When Created | Throughput |
|------|---------------|----------|--------------|------------|
| **LSI** | Same as table | Different | Table creation only | Shares table's |
| **GSI** | Different | Different | Anytime | Separate (provision separately) |

> ⚠️ **GSI throttling** can throttle main table writes. Provision GSI capacity carefully.

### DynamoDB Streams

Ordered stream of item modifications (insert, update, delete).

| View Type | Content |
|-----------|---------|
| **KEYS_ONLY** | Just the key attributes |
| **NEW_IMAGE** | Item after modification |
| **OLD_IMAGE** | Item before modification |
| **NEW_AND_OLD_IMAGES** | Both images |

**Use cases:** Trigger Lambda, replicate to other tables, analytics

### DynamoDB Operations

| Operation | Description | Cost |
|-----------|-------------|------|
| **GetItem** | Single item by primary key | Uses RCU |
| **Query** | Items by partition key + optional sort key | Efficient, uses RCU |
| **Scan** | Entire table | Expensive, avoid in production |
| **BatchGetItem** | Up to 100 items | Parallel GetItem |
| **BatchWriteItem** | Up to 25 PutItem/DeleteItem | Parallel writes |

### Conditional Writes

```python
# Optimistic locking example
response = table.update_item(
    Key={'pk': 'item1'},
    UpdateExpression='SET #v = :newval, version = version + :inc',
    ConditionExpression='version = :expectedVersion',
    ExpressionAttributeValues={':expectedVersion': 1, ':newval': 'updated', ':inc': 1}
)
```

> 💡 Use for optimistic concurrency control — no locking overhead

### DynamoDB Accelerator (DAX)

In-memory cache for DynamoDB — microsecond latency.

| Feature | Value |
|---------|-------|
| **Latency** | Microseconds (vs milliseconds) |
| **Cache** | Item cache + query cache |
| **Compatibility** | Drop-in replacement (same API) |

> Use case: Read-heavy workloads, hot keys

### Global Tables

Multi-region, multi-active replication.

| Feature | Description |
|---------|-------------|
| **Active-Active** | Read/write in any region |
| **Replication** | Sub-second across regions |
| **Requirement** | DynamoDB Streams must be enabled |

### TTL (Time-To-Live)

Auto-delete expired items (no WCU cost).

```
Set TTL attribute → Store expiry timestamp (epoch) → DynamoDB deletes after expiry
```

---

## SQS (Simple Queue Service)

Fully managed message queue — decouple applications.

### Queue Types

| Type | Throughput | Ordering | Delivery |
|------|------------|----------|----------|
| **Standard** | Unlimited | Best-effort | At-least-once |
| **FIFO** | 300 msg/s (3000 batched) | Strict | Exactly-once |

### Key Settings

| Setting | Default | Description |
|---------|---------|-------------|
| **Visibility Timeout** | 30 seconds | Time message is hidden after receive |
| **Message Retention** | 4 days | Max: 14 days |
| **Max Message Size** | 256 KB | Use S3 for larger payloads |
| **Delay Queue** | 0 seconds | Delay before message is visible |
| **Long Polling** | Disabled | Wait for messages (reduces API calls) |

### Visibility Timeout

```
Receive → Message hidden → Process → Delete
                 ↓
         (If timeout expires before delete)
                 ↓
         Message reappears in queue
```

> 💡 If processing takes longer than visibility timeout, call `ChangeMessageVisibility`

### Dead Letter Queue (DLQ)

Messages that fail processing after `maxReceiveCount` go to DLQ.

| Setting | Description |
|---------|-------------|
| **maxReceiveCount** | # of receives before sending to DLQ |
| **Redrive** | Move DLQ messages back to main queue |

### FIFO Queues

| Feature | Description |
|---------|-------------|
| **MessageGroupId** | Messages in same group processed in order |
| **MessageDeduplicationId** | Prevent duplicates within 5-minute window |
| **Naming** | Queue name must end with `.fifo` |

### SQS + Lambda

Lambda polls SQS and processes batches:

| Setting | Description |
|---------|-------------|
| **Batch size** | 1-10 messages per invocation |
| **Batch window** | Time to wait for batch to fill |
| **Concurrency** | One invocation per message group (FIFO) |

---

## SNS (Simple Notification Service)

Pub/sub messaging — push to multiple subscribers.

### Subscribers

| Type | Use Case |
|------|----------|
| **SQS** | Queue for processing |
| **Lambda** | Serverless processing |
| **HTTP/S** | Webhook endpoints |
| **Email/SMS** | User notifications |
| **Kinesis Data Firehose** | Stream to S3, Redshift |

### Fan-Out Pattern

```
Producer → SNS Topic → SQS Queue 1 → Consumer 1
                    → SQS Queue 2 → Consumer 2
                    → Lambda → Process
```

> 💡 Decouple, parallel processing, different consumption rates

### Message Filtering

Filter messages per subscriber using **filter policies**:

```json
{
  "eventType": ["order_placed"],
  "store": [{"prefix": "us-"}]
}
```

### FIFO Topics

- Order guaranteed per message group
- Subscribers must be SQS FIFO queues
- Topic name must end with `.fifo`

---

## Kinesis

Real-time streaming data at scale.

### Kinesis Services

| Service | Purpose |
|---------|---------|
| **Kinesis Data Streams** | Collect and process real-time data |
| **Kinesis Data Firehose** | Load streams into AWS data stores |
| **Kinesis Data Analytics** | SQL/Flink analytics on streams |
| **Kinesis Video Streams** | Stream video for analytics |

### Kinesis Data Streams

| Concept | Description |
|---------|-------------|
| **Shard** | Unit of capacity (1 MB/s in, 2 MB/s out) |
| **Partition Key** | Determines which shard receives record |
| **Sequence Number** | Unique ID per record within shard |
| **Retention** | 1-365 days (default: 24 hours) |

**Capacity:**

| Direction | Per Shard |
|-----------|-----------|
| **Write** | 1 MB/s or 1,000 records/s |
| **Read** | 2 MB/s (shared by all consumers) |

### Consumer Types

| Type | Description |
|------|-------------|
| **Shared** | Multiple consumers share 2 MB/s per shard |
| **Enhanced Fan-Out** | 2 MB/s per consumer per shard (push model) |

### Kinesis Data Firehose

Near real-time delivery (60-900 second buffer) to:

| Destination | Description |
|-------------|-------------|
| **S3** | Most common |
| **Redshift** | Via S3 copy |
| **OpenSearch** | Search/analytics |
| **HTTP endpoint** | Custom destinations |

> 💡 Firehose = managed, auto-scaling, no capacity planning. Streams = more control, real-time.

### Streams vs Firehose

| Feature | Data Streams | Data Firehose |
|---------|--------------|---------------|
| **Latency** | ~200 ms | 60-900 seconds |
| **Capacity** | Provision shards | Auto-scaling |
| **Data retention** | 1-365 days | No storage |
| **Consumer** | Custom (Lambda, apps) | Built-in destinations |
| **Data transformation** | External | Built-in Lambda |

---

## Step Functions

Orchestrate Lambda functions and AWS services with visual workflows.

### Key Concepts

| Concept | Description |
|---------|-------------|
| **State Machine** | Workflow definition (JSON/YAML) |
| **State** | Individual step in workflow |
| **Execution** | Running instance of state machine |
| **Task** | Unit of work (Lambda, AWS service, HTTP) |

### State Types

| State | Description |
|-------|-------------|
| **Task** | Execute work (Lambda, AWS API) |
| **Choice** | Branch based on condition |
| **Parallel** | Execute branches in parallel |
| **Map** | Iterate over array |
| **Wait** | Delay execution |
| **Pass** | Pass input to output, inject data |
| **Succeed/Fail** | End execution |

### Workflow Types

| Type | Max Duration | Pricing | Use Case |
|------|--------------|---------|----------|
| **Standard** | 1 year | Per state transition | Long-running, auditing |
| **Express** | 5 minutes | Per execution + duration | High-volume, event processing |

### Error Handling

| Mechanism | Description |
|-----------|-------------|
| **Retry** | Retry failed states with backoff |
| **Catch** | Handle errors, transition to fallback |

```json
"Retry": [{
  "ErrorEquals": ["States.TaskFailed"],
  "MaxAttempts": 3,
  "IntervalSeconds": 1,
  "BackoffRate": 2.0
}],
"Catch": [{
  "ErrorEquals": ["States.ALL"],
  "Next": "HandleError"
}]
```

### Service Integrations

| Pattern | Description |
|---------|-------------|
| **Request Response** | Call service, wait for response |
| **Run a Job (.sync)** | Wait for job completion (Batch, ECS, Glue) |
| **Wait for Callback** | Pause until external callback (Human approval) |

---

## ECS, Fargate & ECR (Containers)

### ECS (Elastic Container Service)

Container orchestration on AWS.

| Launch Type | Description |
|-------------|-------------|
| **EC2** | You manage EC2 instances, more control |
| **Fargate** | Serverless, AWS manages infrastructure |

### ECS Concepts

| Concept | Description |
|---------|-------------|
| **Task Definition** | Blueprint for containers (image, CPU, memory, ports) |
| **Task** | Running instance of Task Definition |
| **Service** | Maintains desired count of tasks, load balancing |
| **Cluster** | Logical grouping of tasks/services |

### Task Definition Settings

| Setting | Description |
|---------|-------------|
| **Image** | Docker image (from ECR or public) |
| **CPU/Memory** | Resource allocation |
| **Port Mappings** | Container port to host port |
| **Environment** | Variables, secrets from SSM/Secrets Manager |
| **IAM Role** | Task role (permissions for containers) |
| **Logging** | CloudWatch Logs integration |

### Fargate

| Feature | Description |
|---------|-------------|
| **Serverless** | No EC2 management |
| **Pricing** | Per vCPU + memory per second |
| **Scaling** | Auto-scaling on CPU/memory metrics |

### ECR (Elastic Container Registry)

Private Docker registry:

| Feature | Description |
|---------|-------------|
| **Encryption** | Images encrypted at rest |
| **Scanning** | Vulnerability scanning |
| **Lifecycle Policies** | Auto-delete old images |
| **Cross-region** | Replicate to other regions |

### ECS IAM Roles

| Role | Purpose |
|------|---------|
| **Task Execution Role** | Pulls images from ECR, sends logs to CloudWatch |
| **Task Role** | Permissions for the application running in container |

> 💡 **Task Role** = what container can do. **Execution Role** = what ECS agent can do.

### ECS + Load Balancing

| Feature | Description |
|---------|-------------|
| **ALB** | Dynamic port mapping, path-based routing |
| **NLB** | High throughput, static IP |
| **Service Discovery** | Route 53 DNS for service-to-service |

---

## CloudFormation

Infrastructure as Code — define AWS resources in templates.

### Template Structure

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: String
Parameters: # Input values
Resources: # AWS resources (REQUIRED)
Outputs: # Export values
Mappings: # Static variables
Conditions: # Conditional resource creation
```

### Intrinsic Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `!Ref` | Reference resource/parameter | `!Ref MyBucket` |
| `!GetAtt` | Get resource attribute | `!GetAtt MyBucket.Arn` |
| `!Sub` | String substitution | `!Sub "arn:aws:s3:::${BucketName}"` |
| `!Join` | Join strings | `!Join ["-", [a, b, c]]` → "a-b-c" |
| `!If` | Conditional value | `!If [Prod, m5.large, t3.micro]` |
| `!ImportValue` | Import from another stack | `!ImportValue VPCId` |
| `!FindInMap` | Lookup in Mappings | `!FindInMap [RegionMap, !Ref 'AWS::Region', AMI]` |

### Pseudo Parameters

| Parameter | Value |
|-----------|-------|
| `AWS::AccountId` | Account ID |
| `AWS::Region` | Current region |
| `AWS::StackName` | Stack name |
| `AWS::StackId` | Stack ID |
| `AWS::NoValue` | Remove property conditionally |

### Cross-Stack References

**Stack A (export):**
```yaml
Outputs:
  VPCId:
    Value: !Ref MyVPC
    Export:
      Name: SharedVPC
```

**Stack B (import):**
```yaml
VpcId: !ImportValue SharedVPC
```

### Nested Stacks

Reusable components embedded in parent stack:

```yaml
Resources:
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/mybucket/network.yaml
```

> 💡 **Nested** = component reuse. **Cross-stack** = share values between independent stacks.

### Change Sets

Preview changes before executing:

```bash
aws cloudformation create-change-set --stack-name MyStack --template-body file://template.yaml
aws cloudformation describe-change-set --change-set-name MyChangeSet
aws cloudformation execute-change-set --change-set-name MyChangeSet
```

### Drift Detection

Detect if actual resources differ from template definition.

---

## AWS SAM (Serverless Application Model)

Simplified CloudFormation for serverless.

### SAM Template

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31  # SAM transform

Globals:
  Function:
    Timeout: 30

Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: index.handler
      Runtime: python3.9
      CodeUri: ./src
      Events:
        Api:
          Type: Api
          Properties:
            Path: /hello
            Method: GET
```

### SAM Resource Types

| Type | Creates |
|------|---------|
| `AWS::Serverless::Function` | Lambda + execution role |
| `AWS::Serverless::Api` | API Gateway REST API |
| `AWS::Serverless::HttpApi` | API Gateway HTTP API |
| `AWS::Serverless::SimpleTable` | DynamoDB table |
| `AWS::Serverless::LayerVersion` | Lambda Layer |

### SAM CLI Commands

| Command | Description |
|---------|-------------|
| `sam init` | Initialize new project |
| `sam build` | Build and package |
| `sam local invoke` | Test locally |
| `sam local start-api` | Local API Gateway |
| `sam deploy --guided` | Interactive deployment |
| `sam sync` | Fast sync for development |

### SAM Policy Templates

Built-in policies for common patterns:

```yaml
Policies:
  - S3ReadPolicy:
      BucketName: !Ref MyBucket
  - DynamoDBCrudPolicy:
      TableName: !Ref MyTable
```

---

## CI/CD: CodeCommit, CodeBuild, CodeDeploy, CodePipeline

### CodeCommit

AWS Git repository hosting.

| Feature | Description |
|---------|-------------|
| **Auth** | HTTPS (Git credentials), SSH (keys), IAM roles |
| **Triggers** | Lambda, SNS on repository events |
| **Notifications** | CloudWatch Events/EventBridge |

### CodeBuild

Managed build service — compile, test, produce artifacts.

**buildspec.yml:**

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      python: 3.9
  pre_build:
    commands:
      - pip install -r requirements.txt
  build:
    commands:
      - python -m pytest
      - sam build
  post_build:
    commands:
      - sam package --s3-bucket $BUCKET

artifacts:
  files:
    - template.yaml
    - '**/*'

cache:
  paths:
    - '/root/.cache/pip/**/*'
```

| Section | Purpose |
|---------|---------|
| **phases** | install, pre_build, build, post_build |
| **artifacts** | Files to output |
| **cache** | Speed up builds |
| **env** | Environment variables |

### CodeDeploy

Automated deployment to EC2, Lambda, ECS.

**appspec.yml (EC2):**

```yaml
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html
hooks:
  BeforeInstall:
    - location: scripts/install_dependencies.sh
  AfterInstall:
    - location: scripts/start_server.sh
```

**Lifecycle Hooks (EC2):**

```
ApplicationStop → DownloadBundle → BeforeInstall → Install → AfterInstall → ApplicationStart → ValidateService
```

### Deployment Types

| Platform | Types | Description |
|----------|-------|-------------|
| **EC2** | In-Place, Blue/Green | Rolling update or swap target groups |
| **Lambda** | AllAtOnce, Canary, Linear | Traffic shifting |
| **ECS** | Blue/Green | Traffic shifting with ALB |

**Lambda deployment:**

| Type | Description |
|------|-------------|
| **AllAtOnce** | Immediate shift to new version |
| **Canary** | x% for n minutes, then 100% |
| **Linear** | x% every n minutes |

### CodePipeline

Orchestrate CI/CD workflow:

```
Source → Build → Test → Deploy
        ↓
   [Manual Approval]
```

| Feature | Description |
|---------|-------------|
| **Stages** | Sequential groups of actions |
| **Actions** | Individual tasks (source, build, deploy) |
| **Artifacts** | Files passed between stages (stored in S3) |
| **Manual Approval** | Human gate between stages |

---

## CloudWatch

Monitoring, logging, and alarms.

### CloudWatch Metrics

| Concept | Description |
|---------|-------------|
| **Namespace** | Container for metrics (e.g., AWS/EC2) |
| **Dimension** | Attribute of metric (InstanceId, AutoScalingGroupName) |
| **Resolution** | Standard (1 min) or High-res (1 sec) |
| **Custom Metrics** | Your own metrics via PutMetricData API |

**EC2 Default Metrics:**
- CPU, Network, Disk (read/write operations)
- **NOT included:** Memory, disk space (need CloudWatch Agent)

### CloudWatch Alarms

| State | Description |
|-------|-------------|
| **OK** | Metric within threshold |
| **ALARM** | Metric breached threshold |
| **INSUFFICIENT_DATA** | Not enough data points |

**Actions:** SNS notification, Auto Scaling, EC2 actions (stop, terminate, reboot)

### CloudWatch Logs

| Concept | Description |
|---------|-------------|
| **Log Group** | Collection of log streams (e.g., per application) |
| **Log Stream** | Sequence of events from same source |
| **Retention** | Never expire by default, configure 1 day to 10 years |
| **Metric Filters** | Extract metrics from log data |
| **Subscription Filters** | Stream logs to Lambda, Kinesis, OpenSearch |

### CloudWatch Logs Insights

Query logs with SQL-like syntax:

```sql
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20
```

### CloudWatch Agent

Install on EC2/on-premises for:

- **Custom metrics:** Memory, disk, swap, custom
- **Log collection:** Push logs to CloudWatch Logs

### CloudWatch Container Insights

Monitoring for ECS, EKS, Kubernetes — metrics per container, task, service.

---

## X-Ray

Distributed tracing for debugging and performance analysis.

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Trace** | End-to-end request journey |
| **Segment** | Work done by one service |
| **Subsegment** | Granular breakdown (HTTP calls, DB queries) |
| **Annotations** | Indexed key-value pairs (searchable) |
| **Metadata** | Non-indexed key-value pairs |

### X-Ray Integration

| Service | Setup |
|---------|-------|
| **Lambda** | Enable active tracing |
| **API Gateway** | Enable tracing in stage settings |
| **EC2/ECS** | Install X-Ray daemon + SDK |
| **Elastic Beanstalk** | Extension configuration |

### X-Ray Daemon

Runs on EC2/ECS, buffers and sends trace data to X-Ray API.

```
App (X-Ray SDK) → UDP port 2000 → X-Ray Daemon → X-Ray API
```

### X-Ray Sampling

Control volume of requests traced:

| Setting | Description |
|---------|-------------|
| **Reservoir** | Fixed # requests per second traced |
| **Rate** | Percentage of additional requests traced |

Default: 1 request/sec + 5% additional

### X-Ray APIs

| API | Used By |
|-----|---------|
| **PutTraceSegments** | App/SDK uploads segments |
| **GetTraceSummaries** | Get list of traces |
| **BatchGetTraces** | Get full trace details |

---

## Cognito

User identity and access management.

### Cognito User Pools (CUP)

**Authentication** — Sign-up, sign-in, returns JWT tokens.

| Feature | Description |
|---------|-------------|
| **Sign-up/Sign-in** | Email, phone, username |
| **MFA** | SMS, TOTP |
| **Social login** | Google, Facebook, SAML, OIDC |
| **Hosted UI** | Pre-built login pages |
| **Triggers** | Lambda on auth events |

**JWT Tokens:**
- **ID Token:** User identity/attributes
- **Access Token:** API authorization
- **Refresh Token:** Get new tokens

### Cognito Identity Pools (Federated Identities)

**Authorization** — Exchange tokens for temporary AWS credentials.

```
[User] → [CUP/Social] → [ID Token] → [Identity Pool] → [Temp AWS Credentials]
```

| Feature | Description |
|---------|-------------|
| **Federation** | CUP, Google, Facebook, SAML, OpenID |
| **IAM Roles** | Map users to authenticated/unauthenticated roles |
| **Fine-grained** | Policy variables for row-level access |

### User Pools vs Identity Pools

| Feature | User Pools | Identity Pools |
|---------|------------|----------------|
| **Purpose** | Authentication | Authorization |
| **Returns** | JWT tokens | AWS credentials |
| **Use with** | API Gateway, ALB | AWS SDK (S3, DynamoDB) |

---

## KMS (Key Management Service)

Managed encryption keys.

### Key Types

| Type | Managed By | Cost | Rotation |
|------|------------|------|----------|
| **AWS Owned** | AWS | Free | Varies |
| **AWS Managed** | AWS | Free | Auto yearly |
| **Customer Managed** | You | $/month + $/API call | Optional/yearly |

### KMS API Operations

| API | Purpose |
|-----|---------|
| **Encrypt** | Encrypt data up to 4 KB |
| **Decrypt** | Decrypt data |
| **GenerateDataKey** | Returns plaintext + encrypted data key |
| **GenerateDataKeyWithoutPlaintext** | Returns only encrypted data key |

### Envelope Encryption

For data > 4 KB:

```
1. GenerateDataKey → plaintext DEK + encrypted DEK
2. Encrypt data with plaintext DEK
3. Store encrypted DEK with encrypted data
4. Decrypt: Use KMS to decrypt DEK → use DEK to decrypt data
```

### KMS Key Policies

| Policy Type | Description |
|-------------|-------------|
| **Default** | Created automatically, grants access to root user |
| **Custom** | Define who can access key, required for cross-account |

### Encryption Context

Additional authenticated data for extra security:

```python
kms.encrypt(
    KeyId='alias/my-key',
    Plaintext=data,
    EncryptionContext={'department': 'engineering'}
)
```

> Decryption must include same encryption context

---

## Secrets Manager & SSM Parameter Store

### Secrets Manager

| Feature | Description |
|---------|-------------|
| **Purpose** | Store secrets (passwords, API keys, tokens) |
| **Rotation** | Automatic rotation with Lambda |
| **Integration** | RDS, Redshift, DocumentDB automatic rotation |
| **Cost** | Per secret + per API call |

### SSM Parameter Store

| Feature | Description |
|---------|-------------|
| **Purpose** | Configuration and secrets |
| **Types** | String, StringList, SecureString (encrypted) |
| **Hierarchy** | `/app/prod/db-connection` |
| **Cost** | Free (standard) or paid (advanced) |

### When to Use Which

| Use Case | Service |
|----------|---------|
| **Secrets with rotation** | Secrets Manager |
| **RDS/database credentials** | Secrets Manager |
| **Configuration values** | Parameter Store |
| **Cost-sensitive** | Parameter Store |
| **Simple secrets without rotation** | Parameter Store (SecureString) |

---

## EventBridge

Serverless event bus — route events to targets.

### Event Sources

| Source | Examples |
|--------|----------|
| **AWS Services** | EC2, S3, CodePipeline state changes |
| **Custom Apps** | Your applications via PutEvents API |
| **SaaS Partners** | Zendesk, Datadog, Auth0 |
| **Scheduled** | Cron expressions |

### Event Rules

| Type | Description |
|------|-------------|
| **Event Pattern** | Match events by pattern (source, detail-type, etc.) |
| **Schedule** | Cron or rate expression |

### Event Targets

Lambda, SQS, SNS, Step Functions, Kinesis, ECS Tasks, CodePipeline, EC2 Actions, API Gateway, EventBridge in another account/region...

### Event Pattern Example

```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["stopped", "terminated"]
  }
}
```

### Schema Registry

- Discover/store event schemas
- Generate code bindings
- Versioning

---

## Elastic Beanstalk

PaaS for deploying web applications.

### Deployment Policies

| Policy | Downtime | Description |
|--------|----------|-------------|
| **All at once** | Yes | Fastest, brief outage |
| **Rolling** | No | Deploy batch by batch |
| **Rolling with additional batch** | No | Maintain capacity during deployment |
| **Immutable** | No | New ASG, swap when healthy |
| **Blue/Green** | No | Create new environment, swap URL |

### Beanstalk Extensions

`.ebextensions/*.config` files customize environment:

```yaml
option_settings:
  aws:elasticbeanstalk:application:environment:
    MY_ENV_VAR: value

packages:
  yum:
    git: []

container_commands:
  01_migrate:
    command: "python manage.py migrate"
    leader_only: true
```

### Lifecycle Policy

Limit stored application versions (max 1000):

- Delete based on age or count
- Option to preserve source bundle in S3

---

## Self-Exam Questions

*Click to reveal answers. Includes key DVA-C02 topics beyond the notes above.*

### AWS Global Infrastructure

<details>
<summary>Is IAM a global or regional service?</summary>

> ✅ **Global** — IAM users, groups, roles, and policies are not region-specific.

</details>

<details>
<summary>Is EBS regional or AZ-specific?</summary>

> ✅ **AZ-specific** — EBS volumes are bound to a single Availability Zone.

</details>

<details>
<summary>How many AZs does a Region typically have?</summary>

> ✅ **2-6 AZs** per Region.

</details>

### IAM

<details>
<summary>Can an IAM group contain another group?</summary>

> ✅ **No** — Groups can only contain users, not other groups.

</details>

<details>
<summary>What are IAM Roles used for?</summary>

> ✅ **Services**, not users. Roles grant permissions to AWS services (e.g., EC2, Lambda) to perform actions.

</details>

### EC2

<details>
<summary>You're trying to SSH into your EC2 and getting a timeout. What's the most likely issue?</summary>

> ✅ **Security Group** — Timeout = 100% a security group issue. Check inbound rules for port 22.

</details>

<details>
<summary>Which EC2 purchasing option offers up to 90% discount but can be interrupted?</summary>

> ✅ **Spot Instances** — Cheapest option, but AWS can reclaim when spot price exceeds your bid.

</details>

<details>
<summary>What's the difference between Dedicated Host and Dedicated Instance?</summary>

> ✅ **Dedicated Host** — Full server control, see sockets/cores (for BYOL licensing)
>
> ✅ **Dedicated Instance** — Dedicated hardware, no host visibility

</details>

### Storage (EBS, EFS, Instance Store)

<details>
<summary>What happens to Instance Store data when you stop an EC2 instance?</summary>

> ✅ **Data is lost** — Instance Store is ephemeral. Data is lost on stop, terminate, or hardware failure.

</details>

<details>
<summary>Which EBS volume types can be used as boot volumes?</summary>

> ✅ **SSD types only** — gp2, gp3, io1, io2. HDD types (st1, sc1) cannot be boot volumes.

</details>

<details>
<summary>What is the max IOPS for gp3?</summary>

> ✅ **16,000 IOPS** — Can be provisioned independently of volume size.

</details>

<details>
<summary>Can you attach an EBS volume to multiple EC2 instances?</summary>

> ✅ **Only io1/io2** with Multi-Attach — up to 16 instances, same AZ only.

</details>

<details>
<summary>EFS is compatible with which operating systems?</summary>

> ✅ **Linux only** — EFS is POSIX-compliant, not compatible with Windows.

</details>

### AMI

<details>
<summary>Are AMIs region-specific or global?</summary>

> ✅ **Region-specific** — Must copy an AMI to use it in another region.

</details>

### ELB & ASG

<details>
<summary>What does ELB stand for and is it a load balancer type?</summary>

> ✅ **Elastic Load Balancing** — It's the service name, not a LB type. Actual types are ALB, NLB, GLB, CLB.

</details>

<details>
<summary>Which load balancer provides a static IP address?</summary>

> ✅ **NLB** — Network Load Balancer provides one static IP per AZ. ALB only provides a static DNS hostname.

</details>

<details>
<summary>NLB operates at which OSI layer? ALB?</summary>

> ✅ **NLB** — Layer 4 (Transport: TCP, UDP)
>
> ✅ **ALB** — Layer 7 (Application: HTTP, HTTPS)

</details>

<details>
<summary>Will ELB terminate an unhealthy target?</summary>

> ✅ **No** — ELB only stops routing traffic. ASG with ELB health checks enabled will terminate/replace unhealthy instances.

</details>

<details>
<summary>Is Cross-Zone Load Balancing enabled by default for ALB? NLB?</summary>

> ✅ **ALB** — Enabled by default (free)
>
> ✅ **NLB** — Disabled by default (charged if enabled)

</details>

<details>
<summary>What is the default ASG cooldown period?</summary>

> ✅ **300 seconds (5 minutes)** — Prevents rapid successive scaling actions.

</details>

<details>
<summary>What scaling policy uses ML to predict load patterns?</summary>

> ✅ **Predictive Scaling** — Analyzes historical patterns and pre-provisions capacity.

</details>

### RDS & Aurora

<details>
<summary>Read Replicas use sync or async replication?</summary>

> ✅ **ASYNC** — Data is eventually consistent across read replicas.

</details>

<details>
<summary>Multi-AZ uses sync or async replication?</summary>

> ✅ **SYNC** — Changes are immediately replicated to standby for disaster recovery.

</details>

<details>
<summary>Can you read from a Multi-AZ standby database?</summary>

> ✅ **No** — Standby is only for failover. Use Read Replicas for read scaling.

</details>

<details>
<summary>How many Read Replicas can RDS have? Aurora?</summary>

> ✅ Both can have up to **15 Read Replicas**.

</details>

<details>
<summary>What's the failover time for Aurora?</summary>

> ✅ **Less than 30 seconds**.

</details>

<details>
<summary>How do you encrypt an existing unencrypted RDS database?</summary>

> ✅ **Snapshot → Copy with encryption → Restore** from encrypted snapshot.

</details>

<details>
<summary>What is RDS Proxy and when should you use it?</summary>

> ✅ Serverless connection pooler. Use with **Lambda** to reduce DB connections (Lambda opens many short-lived connections).

</details>

<details>
<summary>Is RDS Proxy publicly accessible?</summary>

> ✅ **No** — It lives inside your VPC only, never publicly accessible.

</details>

### Lambda

<details>
<summary>What is the maximum Lambda execution timeout?</summary>

> ✅ **15 minutes (900 seconds)**.

</details>

<details>
<summary>What is the maximum Lambda memory allocation?</summary>

> ✅ **10,240 MB (10 GB)**. CPU scales proportionally with memory.

</details>

<details>
<summary>What is the /tmp directory size limit in Lambda?</summary>

> ✅ **10,240 MB (10 GB)** — Use for temporary file processing.

</details>

<details>
<summary>What happens if Lambda runs out of memory?</summary>

> ✅ Execution fails with **"Process exited before completing request"** or **OutOfMemoryError**.

</details>

<details>
<summary>What are Lambda Layers used for?</summary>

> ✅ Share code/dependencies across multiple functions. Up to **5 layers** per function.

</details>

<details>
<summary>How do you give Lambda access to resources in a VPC?</summary>

> ✅ Configure VPC settings (subnets + security groups). Lambda creates ENIs in your VPC.

</details>

<details>
<summary>What's the difference between synchronous and asynchronous Lambda invocation?</summary>

> ✅ **Sync** — Caller waits for response (API Gateway, SDK invoke)
>
> ✅ **Async** — Caller doesn't wait, Lambda handles retries (S3, SNS, EventBridge)

</details>

<details>
<summary>How many retries does Lambda do for async invocations?</summary>

> ✅ **2 retries** (3 total attempts). Failed events can go to DLQ or on-failure destination.

</details>

### API Gateway

<details>
<summary>What are the three API Gateway endpoint types?</summary>

> ✅ **Edge-optimized** (CloudFront), **Regional**, **Private** (VPC only)

</details>

<details>
<summary>What is the API Gateway default timeout?</summary>

> ✅ **29 seconds** — Cannot exceed this even if Lambda timeout is higher.

</details>

<details>
<summary>How do you handle CORS in API Gateway?</summary>

> ✅ Enable CORS on the resource/method. API Gateway adds `Access-Control-Allow-Origin` headers.

</details>

<details>
<summary>What's the difference between REST API and HTTP API in API Gateway?</summary>

> ✅ **HTTP API** — Cheaper, faster, simpler (JWT auth, Lambda proxy)
>
> ✅ **REST API** — Full features (caching, request validation, usage plans, API keys)

</details>

<details>
<summary>How do you implement rate limiting in API Gateway?</summary>

> ✅ **Usage Plans + API Keys** — Set throttling limits per client.

</details>

### DynamoDB

<details>
<summary>What are the two capacity modes in DynamoDB?</summary>

> ✅ **Provisioned** (set RCU/WCU) and **On-Demand** (pay per request).

</details>

<details>
<summary>What is the maximum item size in DynamoDB?</summary>

> ✅ **400 KB** per item.

</details>

<details>
<summary>What's the difference between Query and Scan?</summary>

> ✅ **Query** — Efficient, uses partition key (and optionally sort key)
>
> ✅ **Scan** — Reads entire table, expensive, use sparingly

</details>

<details>
<summary>What are DynamoDB Streams used for?</summary>

> ✅ Capture item-level changes (insert, update, delete). Trigger Lambda, replicate data, etc.

</details>

<details>
<summary>What is a GSI vs LSI in DynamoDB?</summary>

> ✅ **GSI** — Different partition key, can be added anytime, has own throughput
>
> ✅ **LSI** — Same partition key, must be created at table creation, shares table throughput

</details>

<details>
<summary>How do you implement optimistic locking in DynamoDB?</summary>

> ✅ Use **conditional writes** with a version attribute. Write fails if version doesn't match.

</details>

### S3

<details>
<summary>What is the maximum object size in S3?</summary>

> ✅ **5 TB**. Use multipart upload for objects > 100 MB (required > 5 GB).

</details>

<details>
<summary>What is S3 Transfer Acceleration?</summary>

> ✅ Uses CloudFront edge locations to speed up uploads over long distances.

</details>

<details>
<summary>What's the difference between S3 Standard-IA and S3 One Zone-IA?</summary>

> ✅ **Standard-IA** — Multi-AZ, for infrequent access
>
> ✅ **One Zone-IA** — Single AZ, cheaper, data lost if AZ fails

</details>

<details>
<summary>What is S3 Object Lock?</summary>

> ✅ WORM model (Write Once Read Many). Prevents object deletion/modification for retention period.

</details>

<details>
<summary>How do you enable versioning on an S3 bucket?</summary>

> ✅ Enable at bucket level. Once enabled, can only be suspended (not disabled). Protects against accidental deletes.

</details>

### SQS & SNS

<details>
<summary>What is the default visibility timeout for SQS?</summary>

> ✅ **30 seconds** — Time a message is hidden after being read.

</details>

<details>
<summary>What is the maximum retention period for SQS messages?</summary>

> ✅ **14 days** (default: 4 days).

</details>

<details>
<summary>What's the difference between Standard and FIFO SQS queues?</summary>

> ✅ **Standard** — Unlimited throughput, at-least-once delivery, best-effort ordering
>
> ✅ **FIFO** — 300 msg/s (3000 with batching), exactly-once, strict ordering

</details>

<details>
<summary>What is a Dead Letter Queue (DLQ)?</summary>

> ✅ Queue for messages that failed processing after max retries. Helps debug failures.

</details>

<details>
<summary>What's the difference between SQS and SNS?</summary>

> ✅ **SQS** — Queue, pull-based, messages persist until processed
>
> ✅ **SNS** — Pub/sub, push-based, messages sent immediately to all subscribers

</details>

<details>
<summary>What is the SNS + SQS fan-out pattern?</summary>

> ✅ SNS topic pushes to multiple SQS queues. Decouples publishers from consumers, enables parallel processing.

</details>

### CI/CD (CodeCommit, CodeBuild, CodeDeploy, CodePipeline)

<details>
<summary>What is the buildspec.yml file?</summary>

> ✅ CodeBuild configuration file. Defines build phases (install, pre_build, build, post_build) and artifacts.

</details>

<details>
<summary>What is the appspec.yml/appspec.yaml file?</summary>

> ✅ CodeDeploy configuration. Defines deployment lifecycle hooks and file mappings.

</details>

<details>
<summary>What deployment types does CodeDeploy support for EC2?</summary>

> ✅ **In-place** (rolling) and **Blue/Green** (traffic shift to new instances).

</details>

<details>
<summary>What deployment types does CodeDeploy support for Lambda?</summary>

> ✅ **AllAtOnce**, **Canary** (x% then 100%), **Linear** (x% every n minutes).

</details>

### CloudFormation & SAM

<details>
<summary>What is the intrinsic function to reference another resource in CloudFormation?</summary>

> ✅ `!Ref` or `Ref:` — Returns the physical ID of the resource.

</details>

<details>
<summary>What does !GetAtt do in CloudFormation?</summary>

> ✅ Gets an attribute from a resource (e.g., `!GetAtt MyBucket.Arn`).

</details>

<details>
<summary>What is AWS SAM?</summary>

> ✅ **Serverless Application Model** — Simplified CloudFormation for serverless (Lambda, API Gateway, DynamoDB).

</details>

<details>
<summary>What command packages and deploys a SAM application?</summary>

> ✅ `sam build` → `sam deploy` (or `sam deploy --guided` for interactive).

</details>

### CloudWatch & X-Ray

<details>
<summary>What is the minimum resolution for CloudWatch custom metrics?</summary>

> ✅ **1 second** (high-resolution). Standard is 1 minute.

</details>

<details>
<summary>How long are CloudWatch Logs retained by default?</summary>

> ✅ **Forever** (never expire). Must set retention policy to auto-delete.

</details>

<details>
<summary>What is X-Ray used for?</summary>

> ✅ **Distributed tracing** — Visualize requests as they travel through your application. Debug latency issues.

</details>

<details>
<summary>What is the X-Ray daemon?</summary>

> ✅ Runs on EC2/ECS, collects trace data from SDK and sends to X-Ray service. Lambda has it built-in.

</details>

<details>
<summary>What are X-Ray segments and subsegments?</summary>

> ✅ **Segment** — Work done by a service/resource
>
> ✅ **Subsegment** — Granular breakdown (e.g., external HTTP call, DB query)

</details>

### Cognito

<details>
<summary>What's the difference between Cognito User Pools and Identity Pools?</summary>

> ✅ **User Pools** — Authentication (sign-up, sign-in, get JWT tokens)
>
> ✅ **Identity Pools** — Authorization (exchange tokens for temporary AWS credentials)

</details>

<details>
<summary>How do you authenticate API Gateway with Cognito?</summary>

> ✅ Use **Cognito User Pool Authorizer** — Validates JWT tokens from User Pool.

</details>

### KMS & Encryption

<details>
<summary>What are the two types of KMS keys?</summary>

> ✅ **AWS managed** (aws/service-name, free) and **Customer managed** (you control rotation, policies).

</details>

<details>
<summary>What is envelope encryption?</summary>

> ✅ Data encrypted with **data key**, data key encrypted with **KMS key**. Used for large data.

</details>

<details>
<summary>What is the GenerateDataKey API?</summary>

> ✅ Returns a plaintext data key + encrypted copy. Use plaintext to encrypt data, store encrypted key with data.

</details>

### EventBridge

<details>
<summary>What is EventBridge (formerly CloudWatch Events)?</summary>

> ✅ Serverless event bus. Route events from AWS services, SaaS, custom apps to targets (Lambda, SQS, etc.).

</details>

<details>
<summary>What is an EventBridge rule?</summary>

> ✅ Matches incoming events (by pattern or schedule) and routes to target(s).

</details>

### ElastiCache

<details>
<summary>What's the difference between Redis and Memcached in ElastiCache?</summary>

> ✅ **Redis** — Multi-AZ, replication, persistence, complex data types
>
> ✅ **Memcached** — Simple key-value, multi-threaded, no persistence, horizontal scaling

</details>

<details>
<summary>What is Lazy Loading (Cache-Aside) pattern?</summary>

> ✅ App checks cache first → on miss, fetches from DB → stores in cache → returns. Only requested data is cached.

</details>

<details>
<summary>What is Write-Through caching?</summary>

> ✅ Write to cache AND DB on every update. Cache always current, but write penalty and cache churn.

</details>

<details>
<summary>What is the main drawback of Lazy Loading?</summary>

> ✅ **Cache miss = 3 network calls** (check cache, query DB, write cache). Also, data can become stale.

</details>

<details>
<summary>When would you use Redis over Memcached?</summary>

> ✅ When you need: Multi-AZ, persistence, complex data structures (sorted sets, lists), pub/sub, or backup/restore.

</details>

<details>
<summary>What is TTL in caching?</summary>

> ✅ **Time-To-Live** — Automatic expiration of cached items. Balance freshness vs cache hit rate.

</details>
