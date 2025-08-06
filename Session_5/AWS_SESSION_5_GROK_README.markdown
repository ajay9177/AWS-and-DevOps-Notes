# AWS and DevOps Lecture Notes - Session 05-08-2025

## Overview
This session focused on:
- **VPC and Subnets**: Configuring and accessing EC2 instances within a VPC across multiple subnets.
- **Public and Private IPs**: Understanding internal and external communication between servers.
- **Elastic IPs**: Managing static public IP addresses for EC2 instances.
- **Scenario**: Accessing servers with and without public IPs within the same VPC, including Linux-to-Linux and Windows-to-Linux communication.

---

## Recap: Session 04-08-2025
- Created a VPC with three subnets in different Availability Zones.
- Launched an EC2 instance in the VPC.
- Enabled connectivity by attaching an Internet Gateway and configuring route tables.

---

## 1. Scenario Setup: Accessing Servers in a VPC

### Objective
- **VPC Configuration**: A VPC with three subnets (Subnet-1, Subnet-2, Subnet-3).
- **Instances**:
  - **Server1** (Subnet-1): Amazon Linux, `t3.micro`, public IP enabled, in `us-east-1c`.
  - **Server2** (Subnet-2): Amazon Linux, `t3.micro`, public IP disabled.
  - **Server3** (Subnet-3): Windows Server, public IP enabled.
- **Goal**: Access Server1 (public IP) and Server2 (private IP only) from outside the VPC and from Server3 (Windows instance).

### Launching Server1 (Linux with Public IP)
1. **Steps**:
   - **Name**: `Server1`
   - **AMI**: Amazon Linux 2023 (Free Tier)
   - **Instance Type**: `t3.micro`
   - **VPC and Subnet**: Select the existing VPC and Subnet-1 (`us-east-1c`).
   - **Public IP**: Enable auto-assign public IP.
   - **Security Group**: Use an existing security group allowing all traffic (`0.0.0.0/0` for SSH, port 22).
   - **Key Pair**: Use the previously created key pair (e.g., `.pem` file).
   - **Launch**: Wait for the instance to reach the `Running` state (3/3 checks passed).
2. **Connect**:
   - Use PuTTY or Git Bash to SSH into Server1 using the public IP and `.pem`/`.ppk` file (refer to Session 4 README for SSH steps).

### Launching Server2 (Linux without Public IP)
1. **Steps**:
   - **Name**: `Server2`
   - **AMI**: Amazon Linux 2023 (Free Tier)
   - **Instance Type**: `t3.micro`
   - **VPC and Subnet**: Select the existing VPC and Subnet-2.
   - **Public IP**: Disable auto-assign public IP.
   - **Security Group**: Use the same security group allowing all traffic.
   - **Key Pair**: Use the same key pair as Server1.
   - **Launch**: Confirm successful launch. Note that no public IP is assigned.
2. **Issue**: Attempting to SSH to Server2 using its private IP (e.g., `10.0.2.x`) from outside the VPC results in a **connection timed out** error because Server2 lacks a public IP and is only accessible within the VPC.

### Accessing Server2 from Server1
1. **Concept**: Since Server1 has a public IP and is in the same VPC as Server2, you can SSH from Server1 to Server2 using Server2’s private IP.
2. **Steps**:
   - SSH into Server1 using its public IP (refer to Session 4 README for SSH steps).
   - Copy the `.pem` key pair file to Server1:
     ```bash
     vim keypair.pem
     # Paste the contents of the .pem file
     # Save and exit (:wq)
     chmod 400 keypair.pem
     ```
   - SSH from Server1 to Server2 using Server2’s private IP (e.g., `10.0.2.25`):
     ```bash
     ssh -i keypair.pem ec2-user@10.0.2.25
     ```
3. **Outcome**: Successfully log in to Server2 from Server1 because both instances are in the same VPC, allowing internal communication via private IPs.

### Launching Server3 (Windows with Public IP)
1. **Steps**:
   - **Name**: `Server3`
   - **AMI**: Microsoft Windows Server 2016 Base (Free Tier)
   - **Instance Type**: `t3.small` (avoid `t3.micro` due to performance issues).
   - **VPC and Subnet**: Select the existing VPC and Subnet-3.
   - **Public IP**: Enable auto-assign public IP.
   - **Security Group**: Use the same security group allowing all traffic (including RDP, port 3389).
   - **Key Pair**: Use the same key pair as before.
   - **Launch**: Wait 3–4 minutes for the Windows instance to be ready.
2. **Connect**: Use Remote Desktop Protocol (RDP) to log in (refer to Session 4 README for RDP steps).

### Accessing Server1 and Server2 from Server3 (Windows)
1. **Objective**: Log in to Server1 and Server2 from Server3 using their private IPs, as all instances are in the same VPC.
2. **Steps**:
   - RDP into Server3 using its public IP (refer to Session 4 README).
   - Install **PuTTY** on Server3 (download and install manually or via a package manager).
   - Copy the `.ppk` file (converted from the `.pem` file using PuTTYgen) to Server3’s desktop.
   - Open PuTTY on Server3:
     - For **Server1**: Enter Server1’s private IP (e.g., `10.0.1.x`), select the `.ppk` file under **Connection > SSH > Auth**, and click **Open**. Accept the dialog and log in as `ec2-user`.
     - For **Server2**: Repeat the process using Server2’s private IP (e.g., `10.0.2.x`).
3. **Outcome**: Successfully log in to both Server1 and Server2 from Server3 using their private IPs, as internal communication within the same VPC is possible regardless of subnet.

### Key Concept: Internal vs. External Communication
- **Analogy**: In a home network, devices connected to a router via LAN cables can communicate using private IPs without internet access. Similarly, servers in the same VPC communicate internally using private IPs.
- **Public IP**: Used for external access (e.g., from your local machine to Server1).
- **Private IP**: Used for internal communication within the VPC (e.g., Server3 to Server1/Server2).
- **Best Practice**: In companies, most servers use private IPs for internal communication, with only a few having public IPs for external access (e.g., bastion hosts).

---

## 2. Elastic IPs

### Scenario
- **Action**: Terminate Server2 (no public IP) and Server3 (Windows). Stop Server1 (public IP enabled).
- **Observation**:
  - When Server1 is stopped, its public IP is released, but its private IP remains unchanged.
  - Upon restarting Server1, a new public IP is assigned (dynamic IP behavior).
- **Issue**: If an application or domain is mapped to the public IP, it will break when the IP changes after a stop/start.

### Solution: Elastic IP
1. **Concept**: An Elastic IP is a static public IP address that remains associated with an instance until explicitly disassociated. It is chargeable when not associated with a running instance.
2. **Steps**:
   - Go to **Elastic IPs** in the AWS Management Console.
   - Click **Allocate Elastic IP address** and then **Allocate**.
   - Associate the Elastic IP with Server1:
     - Go to **Actions > Associate Elastic IP address**.
     - Select Server1 and click **Associate**.
   - **Verification**: Stop and restart Server1. The Elastic IP remains unchanged, unlike the dynamic public IP.
3. **Important Notes**:
   - If Server1 is terminated, the Elastic IP is disassociated and must be released manually from the Elastic IPs section to avoid charges.
   - Private IPs are retained until the instance is terminated, even if stopped or rebooted.
   - Elastic IPs are useful for applications requiring consistent public IP addresses (e.g., when mapped to a domain).

---

## Summary of Steps
1. **Launched Instances**:
   - Server1: Linux, Subnet-1, public IP enabled.
   - Server2: Linux, Subnet-2, public IP disabled.
   - Server3: Windows, Subnet-3, public IP enabled.
2. **Accessing Servers**:
   - SSH to Server1 (public IP) from outside the VPC.
   - SSH to Server2 (private IP) from Server1 within the VPC.
   - RDP to Server3, then use PuTTY to SSH to Server1 and Server2 using private IPs.
3. **Elastic IP**:
   - Allocated and associated an Elastic IP to Server1 to maintain a static public IP.
   - Terminated Server2 and Server3, stopped Server1 to observe IP behavior.

---

## Key Takeaways
- **Public vs. Private IPs**:
  - Public IPs enable external access; private IPs enable internal VPC communication.
  - Use a bastion host (e.g., Server1 or Server3) with a public IP to access private IP instances.
- **Elastic IPs**: Provide static public IPs for consistent external access, especially for domain mappings.
- **VPC Communication**: Instances in the same VPC can communicate via private IPs, regardless of subnets, as long as security groups and route tables are configured correctly.
- **Best Practices**:
  - Minimize public IP usage for security.
  - Use Elastic IPs for critical applications requiring static public IPs.
  - Terminate unused instances and release unassociated Elastic IPs to avoid charges.