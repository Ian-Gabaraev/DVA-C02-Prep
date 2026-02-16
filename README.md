# AWS DVA-C02 Study Notes

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