# IAM

It is a `global` service.

There are `groups` and `users`. A `group` cannot contain another `group`.

But `users` can be in multiple `groups`.

Both `users` and `groups` can be assigned `JSON` documents containing permissions and those are called `policies`.

A `user` does not need to be in a `group`. You can attach an `inline policy` to them instead.

Every policy follows the same structure:

`Version` - it is a date usually. Reflects the policy language version.

`Sid` - statement ID.

`Effect` - it is either `Allow` or `Deny` .

`Principal` - refers to the `account` or `user` or `role` to which the `policy` is applied.

`Action` - list of actions that are allowed or denied.

`Resource` - list of resource to which `action` applies.

## Roles

`Roles` are `permissions` meant to be used by `services`, not `users`.
For example, you give an `EC2` instance a `role` so that it can perform some actions in `aws`.

`Permissions` contain `policies`.

To create a `role`, you select a `service` to grant it to, and then you attach `policies` to it. The `service` that is granted a `role` is also called a `trusted entity`.

## IAM Security Tools

`Credentials Report` - it lists all your users and the status of their credentials, download as a CSV.

`Access Advisor` aka `Last Accessed` (new name) - it shows the services that a user has accessed and when was the last time they accessed it.

# EC2

Elastic Compute Cloud `EC2`. It is not just one service, it contains multiple concepts:

- Virtual Machines (`EC2`)
- Virtual Drives (`EBS`)
- Load Balancing (`ELB`)
- Scaling (`ASG`)

What options do you have?

- Operating systems: `MacOS`, `Linux`, `Windows`
- Compute: `RAM` and `CPU` cores
- Storage:
  - Network attached (`EBS` or `EFS`)
  - Hardware (`EC2 Instance Store`)
- Network card: speed of the card, IP address

You can also `bootstrap` your EC2 instance by using `User Data`. That is usually a script that runs when an `EC2` instance starts. Useful to install updates, software, anything. It runs with `root` credentials.

## Security groups (Firewall)

Each EC2 instance will have a `security group` which acts as a `firewall` and controls traffic to the instance.

A `security group` will have

- `inbound rules` - who on the internet can access the instance.
- `outbound rules` - what the instance can access on the internet.

*`A tip:` any time you try to connect to your EC2 instance, over HTTP, HTTPS or SSH, and you see a `timeout`, it is `100% a security group issue`.*

A `security group` is `locked to a specific region or VPC`. So if you switch a region, you will need a new `security group`.

By default, `all inbound traffic is blocked`, and `all outbound is allowed`.

A nice feature:

You have `EC2 A`. It has `SG1`. In its inbound rules it allows `SG2`.

Then you have `EC2 B`. It has `SG2` attached.

Now `EC2 B` can connect straight to `EC2 A`.

### Important ports

- `22` - SSH
- `21` - FTP
- `22` - SFTP
- `80` - HTTP
- `443` - HTTPS

## Instance types

A typical EC2 instance follows a naming convention. E.g.:

`m5.2xlarge`

- `m` - instance class
- `5` - generation
- `2xlarge` - size, defines the memory, the CPU

What types are there

- `General purpose`: `t3`, `m5` - `balance` between compute, memory and network

- `Compute optimized`: `c5` - high CPU to memory ratio, for `compute intensive applications`, like `batch processing` workloads

- `Memory optimized`: `r5` - high memory to CPU ratio, for tasks that `process a lot of data in RAM`, like databases

- `Storage optimized`: `i3` - high storage throughput and IOPS, for `storage-intensive` tasks, like caches or file systems.

- GPU instances: `p3` - for machine learning and AI

- FPGA instances: `f1` - for custom hardware acceleration
