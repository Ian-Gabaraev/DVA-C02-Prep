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
