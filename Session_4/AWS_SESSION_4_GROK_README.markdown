# AWS and DevOps Lecture Notes - Session 04-08-2025

## Overview
This session covered the following key AWS concepts:
- **Regions** and **Availability Zones**
- **AWS Services**: Focus on EC2 (Elastic Compute Cloud) and VPC (Virtual Private Cloud)

---

## 1. EC2 (Elastic Compute Cloud)

### Launching an EC2 Instance (Linux)
1. **Steps to Launch a Linux Instance**:
   - **Name**: `LinuxTestServer`
   - **AMI**: Amazon Linux 2023 (Kernel 6.1, Free Tier)
   - **Instance Type**: `t3.micro` (Free Tier eligible)
   - **Key Pair**:
     - Create a new key pair (e.g., RSA or ED25519).
     - Download the `.pem` file (used for SSH access).
     - For use with PuTTY, convert `.pem` to `.ppk` using **PuTTYgen**:
       - Open PuTTYgen, load the `.pem` file, and save the private key as `.ppk`.
   - **Region-Specific**: Key pairs are region-specific.
   - **Launch**: Linux instances launch faster than Windows instances.
   - **Status**: Wait for `3/3 checks passed` to confirm the instance is running.

2. **Connecting to the Instance**:
   - **Public IPv4 Address**: Copy the instance’s public IPv4 address.
   - **Tools**: Use PuTTY (Windows) or Git Bash (any OS) for SSH.
   - **PuTTY Configuration**:
     - Enter the public IPv4 address in the **Host Name (or IP address)** field.
     - Navigate to **Connection > SSH > Auth**, browse, and select the `.ppk` file.
     - Log in with the username `ec2-user`.
   - **Commands**:
     ```bash
     sudo su
     cd /
     yum update -y
     ```

### Launching an EC2 Instance (Windows)
1. **Steps to Launch a Windows Instance**:
   - **Name**: `WindowsServer`
   - **AMI**: Microsoft Windows Server 2016 Base (Free Tier)
   - **Instance Type**: `t3.small` (avoid `t3.micro` due to performance issues).
   - **Key Pair**: Use the same key pair created for Linux or create a new one.
   - **Launch**: Windows instances take longer to launch compared to Linux.

2. **Connecting to the Instance**:
   - In the AWS Management Console, select the instance and click **Connect**.
   - Choose **RDP Client**.
   - Upload the `.pem` file to **Decrypt Password** in the AWS Management Console.
   - Open **Remote Desktop Connection**:
     - Enter the public IPv4 address.
     - Username: `Administrator`
     - Password: Decrypted password from AWS.
     - Click **Yes** to accept the certificate.

3. **Important Notes**:
   - **Terminate** instances when not in use to avoid charges.
   - **Stop** instances if you plan to use them later, but note that stopped instances may still incur minor charges (e.g., for storage).

---

## 2. VPC (Virtual Private Cloud)

### Introduction
- By default, AWS provides a **default VPC** for launching instances.
- A VPC is mandatory to launch any EC2 instance, as it provides network isolation and connectivity.
- Analogy: A PC without internet or a router cannot access external resources or be accessed remotely. Similarly, a VPC enables connectivity for AWS resources.

### Deleting and Creating a Custom VPC
1. **Why Delete Default VPC?**
   - To configure a custom VPC tailored to specific requirements (e.g., IP range, subnets).
   - **Note**: A VPC is required to launch instances.

2. **Prerequisites for Creating a VPC**:
   - **IP Address Range**: Choose an IPv4 CIDR block.
   - **Subnet Requirements**: Determine the number of subnets and IPs needed.
   - **Supported CIDR Range**: AWS supports `/16` to `/24` (e.g., `10.0.0.0/16` provides ~65,534 IPs, while `10.0.0.0/24` provides ~256 IPs).
   - **Unsupported CIDR**: `/8` (e.g., `10.0.0.0/8`, ~16 million IPs) is not supported by AWS.

3. **Steps to Create a VPC**:
   - **Name**: `TestingVPC`
   - **IPv4 CIDR**: `10.0.0.0/16` (~65,534 IPs).
   - Use a subnet calculator (e.g., subnet-calculator.com) to plan subnets.
   - **Create VPC**: If `/8` is selected, AWS will prompt to use `/16` or `/24`.

4. **Creating Subnets**:
   - **VPC**: Select the created VPC (`TestingVPC`).
   - **Subnet Names and CIDR Blocks**:
     - Subnet 1: `10.0.1.0/24` (~256 IPs, 251 usable after 5 reserved by AWS).
     - Subnet 2: `10.0.2.0/24` (~256 IPs, 251 usable).
     - Subnet 3: `10.0.3.0/24` (~256 IPs, 251 usable).
   - **Availability Zones**:
     - Subnets can be created in the same Availability Zone (e.g., `us-east-1a`) or distributed across multiple zones for high availability.
   - **Note**: Without subnets, instances cannot be launched.

5. **Question**: From a `/16` CIDR (~65,534 IPs), approximately 255 subnets of `/24` (~256 IPs each) can be created, accounting for reserved IPs per subnet.

### Security Groups
- **Purpose**: Control inbound and outbound traffic to instances.
- **Rule Example**:
   - Allow traffic from `0.0.0.0/0` (anywhere) for protocols like SSH (port 22) or RDP (port 3389).
   - **Caution**: Avoid using `0.0.0.0/0` for production environments; restrict access to specific IPs for security.

### Enabling Internet Access
1. **Issue**: A newly created VPC does not have internet access by default, preventing instance login.
2. **Solution**:
   - **Create an Internet Gateway**:
     - Name: `TestingIGW`
     - Attach it to the VPC (`TestingVPC`).
   - **Configure Route Table**:
     - Update the route table to include a route for `0.0.0.0/0` pointing to the Internet Gateway.
     - Associate subnets explicitly with the route table (move from implicit to explicit associations).

### Summary of VPC Setup
1. Create a VPC:
   - CIDR: `10.0.0.0/16`
2. Create subnets:
   - Subnet 1: `10.0.1.0/24`
   - Subnet 2: `10.0.2.0/24`
   - Subnet 3: `10.0.3.0/24`
3. Create an Internet Gateway and attach it to the VPC.
4. Update the route table:
   - Add route: `0.0.0.0/0` → Internet Gateway.
   - Explicitly associate subnets with the route table.

---

## Key Takeaways
- **EC2**: Launch Linux and Windows instances with appropriate AMIs and instance types. Use key pairs for secure access.
- **VPC**: Custom VPCs require subnets, an Internet Gateway, and route table configuration for connectivity.
- **Best Practices**:
  - Terminate or stop unused instances to manage costs.
  - Restrict security group rules for controlled access.
  - Plan IP ranges and subnets carefully using tools like subnet calculators.