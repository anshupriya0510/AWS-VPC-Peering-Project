# AWS VPC Peering Project | Secure Cross-VPC Communication

## 📌 Project Overview

This project demonstrates how to establish private network communication between two separate Amazon Virtual Private Clouds (VPCs) using **AWS VPC Peering**.

Two VPCs are configured with separate CIDR blocks, private subnets, route tables, and security groups. A Bastion Host provides secure SSH access to the private EC2 instances. Connectivity is verified by testing ICMP traffic between the application instances in both directions.

## 🎯 Objectives

- Create two isolated VPCs in AWS.
- Configure public and private subnets.
- Establish a VPC Peering connection.
- Configure route tables for cross-VPC communication.
- Implement security groups to control network access.
- Access private EC2 instances through a Bastion Host.
- Validate connectivity using `ping`.

## 🏗️ Architecture

```text
                 Developer / Windows PC
                           |
                       SSH (22)
                           |
                    Internet Gateway
                           |
                 +--------------------+
                 |   VPC A            |
                 |   10.10.0.0/16     |
                 |                    |
                 |  Public Subnet     |
                 |  10.10.1.0/24      |
                 |  Bastion Host       |
                 |  10.10.1.10         |
                 |         |           |
                 |  Private Subnet     |
                 |  10.10.2.0/24      |
                 |  App-A: 10.10.2.10 |
                 +---------+----------+
                           |
                    VPC Peering
                           |
                 +---------+----------+
                 |   VPC B            |
                 |   10.20.0.0/16     |
                 |                    |
                 |  Private Subnet    |
                 |  10.20.1.0/24      |
                 |  App-B: 10.20.1.10 |
                 +--------------------+
```

**Note:** The diagram is a logical overview of the lab. The Bastion Host is in VPC A's public subnet, while App-A and App-B are in private subnets.

## 🛠️ AWS Services and Technologies

- **Amazon VPC** — isolated virtual networks
- **Amazon EC2** — Bastion Host and application instances
- **VPC Peering** — private communication between VPCs
- **Subnets** — public and private network segmentation
- **Route Tables** — routing between VPC CIDR blocks
- **Internet Gateway** — internet connectivity for the Bastion Host
- **Security Groups** — instance-level traffic control
- **Network ACLs** — subnet-level traffic control
- **Amazon Linux 2023** — EC2 operating system
- **SSH and ICMP** — remote access and connectivity testing

## 🌐 Network Configuration

| Resource | Name | CIDR / Private IP |
|---|---|---|
| VPC A | `ewb-vpc-a` | `10.10.0.0/16` |
| VPC A public subnet | `ewb-a-public` | `10.10.1.0/24` |
| VPC A private subnet | `ewb-a-private` | `10.10.2.0/24` |
| VPC B | `ewb-vpc-b` | `10.20.0.0/16` |
| VPC B private subnet | `ewb-b-private` | `10.20.1.0/24` |
| Bastion Host | `ewb-bastion` | `10.10.1.10` |
| App-A | `ewb-app-a` | `10.10.2.10` |
| App-B | `ewb-app-b` | `10.20.1.10` |
| Peering connection | `ewb-a-b-peer` | VPC A ↔ VPC B |

## ⚙️ Implementation Steps

### 1. Create the VPCs

Create two VPCs with non-overlapping CIDR ranges:

- VPC A: `10.10.0.0/16`
- VPC B: `10.20.0.0/16`

Non-overlapping CIDRs allow routes between the networks without address conflicts.

### 2. Create the Subnets

Configure the following subnets:

- VPC A public subnet: `10.10.1.0/24`
- VPC A private subnet: `10.10.2.0/24`
- VPC B private subnet: `10.20.1.0/24`

### 3. Configure the Internet Gateway

Create `ewb-igw-a` and attach it to VPC A.

Configure the public route table with:

| Destination | Target |
|---|---|
| `10.10.0.0/16` | Local |
| `0.0.0.0/0` | Internet Gateway |

### 4. Establish VPC Peering

Create the peering connection `ewb-a-b-peer` between VPC A and VPC B, then accept the connection request.

Verify that its status is **Active**.

### 5. Configure Route Tables

Add the peering routes to the relevant route tables.

**VPC A route tables:**

| Destination | Target |
|---|---|
| `10.20.0.0/16` | VPC Peering Connection |

**VPC B route table:**

| Destination | Target |
|---|---|
| `10.10.0.0/16` | VPC Peering Connection |

Keep the local routes and the existing public Internet Gateway route where applicable.

### 6. Configure Security Groups

Configure the Bastion Host security group to permit SSH on TCP port 22 from your own public IP address.

For the application instances, permit only the necessary SSH and ICMP traffic from the intended source addresses. Keep application instances private and avoid opening SSH to the entire internet.

### 7. Launch EC2 Instances

Launch three Amazon Linux 2023 instances:

- `ewb-bastion` in the VPC A public subnet
- `ewb-app-a` in the VPC A private subnet
- `ewb-app-b` in the VPC B private subnet

Assign the intended private IP addresses and attach the appropriate security groups.

### 8. Connect Through the Bastion Host

Use SSH from the local computer to access the Bastion Host, then use an SSH jump connection to reach the private instances.

Example PowerShell command for App-A:

```powershell
ssh -o IdentitiesOnly=yes -i "$HOME\Downloads\ewb-lab.pem" -o "ProxyCommand=ssh -o IdentitiesOnly=yes -i C:\Users\YOUR_USERNAME\Downloads\ewb-lab.pem -W %h:%p ec2-user@BASTION_PUBLIC_IP" ec2-user@10.10.2.10
```

Replace `YOUR_USERNAME` and `BASTION_PUBLIC_IP` with your Windows username and current Bastion Host public IP.

**Security note:** Keep the private key on your local computer. Do not upload the `.pem` file to GitHub or copy it to the Bastion Host.

## 🧪 Connectivity Testing

### Test 1: App-A to App-B

Run this command from the App-A terminal:

```bash
ping -c 4 10.20.1.10
```

**Observed result:** 4 packets transmitted, 4 received, 0% packet loss.

### Test 2: App-B to App-A

Run this command from the App-B terminal:

```bash
ping -c 4 10.10.2.10
```

**Observed result:** 4 packets transmitted, 4 received, 0% packet loss.

These tests demonstrate successful private IP connectivity between the two VPCs in both directions.

## 📸 Screenshots

Create a `screenshots/` directory and add your actual AWS console and terminal screenshots.

```text
screenshots/
├── 01-vpc-a.png
├── 02-vpc-b.png
├── 03-vpc-peering-active.png
├── 04-route-table-vpc-a.png
├── 05-route-table-vpc-b.png
├── 06-ec2-instances.png
├── 07-ping-app-a-to-app-b.png
└── 08-ping-app-b-to-app-a.png
```

Embed the screenshots in this README using Markdown, for example:

```markdown
![VPC Peering Connection](screenshots/03-vpc-peering-active.png)

![App-A to App-B Connectivity](screenshots/07-ping-app-a-to-app-b.png)

![App-B to App-A Connectivity](screenshots/08-ping-app-b-to-app-a.png)
```

Make sure the image filenames match the files you upload.

## 🔐 Security Best Practices

- Restrict SSH access to trusted public IP addresses.
- Keep application instances in private subnets.
- Use security groups with least-privilege rules.
- Never commit private keys, credentials, or AWS access keys.
- Review AWS billing and stop EC2 instances when they are not needed.
- Remember that stopping instances does not eliminate all charges; EBS storage and public IPv4 addresses may continue to incur costs.

## 📚 Key Learnings

- Creating and configuring custom VPCs and subnets
- Establishing VPC Peering connections
- Configuring route tables for cross-VPC routing
- Managing security groups and Network ACLs
- Connecting securely to private instances through a Bastion Host
- Troubleshooting SSH connectivity
- Verifying network communication using ICMP and `ping`

## 🚀 Future Improvements

- Automate infrastructure provisioning using Terraform.
- Add VPC Flow Logs for network troubleshooting.
- Configure CloudWatch monitoring and alarms.
- Explore AWS Systems Manager Session Manager for private instance access.
- Implement a more comprehensive network architecture with high availability.



*This project was built for hands-on learning and demonstration of AWS networking, secure access, and private communication between VPCs.*
