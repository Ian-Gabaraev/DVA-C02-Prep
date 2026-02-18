# AWS DVA-C02 Study Notes

## Table of Contents

| Section | Topics |
|---------|--------|
| [IAM](#iam-identity--access-management) | Users, Groups, Policies, Roles |
| [EC2](#ec2-elastic-compute-cloud) | Instance Types, Purchasing, Security Groups |
| [AMI](#ami-amazon-machine-image) | Custom AMIs, Cross-region |
| [EBS](#ebs-elastic-block-store) | Volume Types, Snapshots, Multi-Attach |
| [EFS](#efs-elastic-file-system) | Performance Modes, Storage Tiers |
| [Storage Comparison](#ebs-vs-efs-vs-instance-store) | EBS vs EFS vs Instance Store |
| [ELB & ASG](#elb--asg-load-balancing--auto-scaling) | ALB, NLB, GLB, Health Checks, Scaling |

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

- Classic Load Balancer (old) - HTTP, HTTPS, TCP and SSL
- Application Load Balancer - HTTP, HTTPS, WebSocket
- Network Load Balancer - TCP, TLS, UDP
- Gateway Load Balancer - IP Protocol

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