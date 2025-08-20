# AWS and DevOps Lecture Notes - Session 06-08-2025

## Overview
This session covered:
- **EC2 (Elastic Compute Cloud)**: Core concepts, instance launching, and key pair management.
- **Instance Types**: Categories and their use cases.
- **On-Demand vs. Spot Instances**: Differences and Spot Fleet launch steps.
- **AWS Pricing Calculator**: Properties used for cost estimation.
- **Savings Plans**: Overview and steps to configure.
- **Reserved Instances**: Payment options and use cases.

---

## Recap: Session 05-08-2025
- Configured a VPC with three subnets and launched instances (Linux and Windows).
- Demonstrated internal communication using private IPs and external access via public IPs.
- Introduced Elastic IPs for static public IP addresses.

---

## 1. EC2 (Elastic Compute Cloud)

### Overview
- **EC2** provides scalable virtual servers (instances) in the cloud, supporting Linux and Windows operating systems.
- **Key Components**:
  - **Volumes**: Storage attached to instances (e.g., Elastic Block Store - EBS).
  - **Snapshots**: Point-in-time backups of EBS volumes.
  - **AMIs (Amazon Machine Images)**: Templates to clone or launch instances with pre-configured settings.
  - **Configuration Needs**: Define vCPUs, RAM, storage, and network bandwidth based on application requirements.
- **Comparison**: Similar to Virtual Machines (VMs) in Azure, with minimal differences in configuration.

### Launching an EC2 Instance (Linux and Windows)
1. **Steps for Windows Instance**:
   - **Go to EC2 > Instances > Launch Instances**.
   - **Name**: `WindowsServer`
   - **AMI**: Microsoft Windows Server 2016 Base (Free Tier eligible).
   - **Instance Type**: `t2.medium` (not Free Tier; use `t2.micro` for Free Tier to avoid charges).
   - **Key Pair**:
     - Create a new key pair: `Test-KeyPair`.
     - Type: RSA (`.pem` for Windows, convertible to `.ppk` for Linux SSH with PuTTYgen).
     - **Important**: Key pairs are **region-specific** (check the region in the AWS Console’s top corner).
     - **Critical Note**: If the `.pem` file is lost, it cannot be retrieved from AWS. Always store it securely.
   - **Network Settings**:
     - Select the existing VPC and subnet.
     - Enable or disable public IP based on requirements.
     - **Security Group**: Use a custom security group allowing all ports (not recommended for production; configure specific ports like RDP [port 3389] for Windows).
   - **Storage**: Configure as needed (default or increase based on requirements).
   - **Launch**: Wait for `2/2 checks passed` to confirm the instance is running.
2. **Steps for Linux Instance**:
   - Similar process as Windows, but faster to launch.
   - Use Amazon Linux 2023 AMI, `t2.micro` (Free Tier), and configure SSH (port 22) in the security group.
   - Refer to Session 4 README for detailed Linux launch steps.
3. **Best Practices**:
   - Use Free Tier instance types (`t2.micro`, `t3.micro`) to avoid charges unless testing higher configurations.
   - Configure security groups to allow only necessary ports (e.g., 22 for SSH, 3389 for RDP).

---

## 2. EC2 Instance Types

### Overview
- **Instance Types** define the compute, memory, storage, and network capacity of an EC2 instance.
- **Key Parameters**:
  - vCPUs (virtual CPUs)
  - Memory (RAM in GB)
  - Instance Storage (GB)
  - Network Bandwidth (Gbps)
  - EBS Bandwidth (Gbps)
- **Reference**: [AWS EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)

### Categories and Use Cases
1. **General Purpose**:
   - Examples: `M8g`, `M7g` (AWS Graviton processors), `M7i` (Intel processors), `M7a` (AMD processors).
   - Use Case: Balanced compute, memory, and networking for web servers, small databases, or development environments.
   - **Note**: Check application compatibility with Graviton (ARM-based) vs. Intel/AMD processors. Most organizations use Intel-based instances (e.g., `M5.large`, `M5.xlarge`, `M5.2xlarge`).
2. **Compute Optimized**:
   - Examples: `C6i` (Intel-based).
   - Use Case: High-performance computing tasks like batch processing or media encoding.
3. **Memory Optimized**:
   - Use Case: Memory-intensive applications like in-memory databases (e.g., Redis, SAP HANA).
4. **Accelerated Computing**:
   - Use Case: Machine learning (ML), artificial intelligence (AI), gaming, or graphic-intensive applications (e.g., UI/UX design).
5. **Storage Optimized**:
   - Use Case: High-performance storage needs like data warehousing or big data analytics.
6. **HPC Optimized**:
   - Use Case: High-performance computing tasks like scientific simulations or financial modeling.

### Best Practice
- Choose instance types based on client requirements for vCPUs, memory, and storage.
- Intel processors are widely used due to broad application compatibility.

---

## 3. On-Demand vs. Spot Instances

### On-Demand Instances
- **Definition**: Dedicated instances allocated to you until terminated.
- **Use Case**: Stable, predictable workloads requiring uninterrupted access.
- **Cost**: Full price, no discounts.

### Spot Instances
- **Definition**: Allows you to bid on unused EC2 capacity at a discounted rate (up to 90% off On-Demand prices).
- **Key Characteristics**:
  - Instances can be **terminated** by AWS with a 2-minute warning if capacity is needed for On-Demand users.
  - Ideal for fault-tolerant, stateless, or flexible workloads (e.g., batch processing, CI/CD pipelines, or big data analytics).
- **Risk**: If another user bids higher or AWS reclaims capacity, your Spot Instance may be interrupted.
- **Reference**: [AWS Spot Instances](https://aws.amazon.com/ec2/spot/)

### Steps to Launch a Spot Fleet
A **Spot Fleet** is a collection of Spot Instances (and optionally On-Demand Instances) that AWS manages to meet your target capacity.

1. **Go to EC2 > Spot Requests > Request Spot Instances**.
2. **Define Target Capacity**:
   - Specify the total number of instances or vCPUs needed.
3. **Select Instance Types**:
   - Choose multiple instance types (e.g., `t3.micro`, `t2.micro`) for flexibility.
   - AWS will select the most cost-effective instances based on Spot pricing.
4. **Set Allocation Strategy**:
   - Options: `Lowest Price`, `Diversified` (spread across instance types), or `Capacity Optimized` (prioritizes capacity availability).
5. **Configure Bid Price**:
   - Set the maximum price you’re willing to pay per instance hour (or use the default, which is the On-Demand price).
6. **Network Settings**:
   - Select the VPC, subnets, and security groups.
   - Enable public IPs if needed.
7. **Key Pair**: Use an existing key pair for SSH/RDP access.
8. **Launch Configuration**:
   - Specify the AMI (e.g., Amazon Linux 2023 or Windows Server).
   - Add user data (optional) for instance initialization scripts.
9. **Submit Request**:
   - Review and submit the Spot Fleet request.
   - AWS will provision instances when capacity is available at or below your bid price.
10. **Monitor**:
    - Check **EC2 > Spot Requests** to monitor the status of your Spot Fleet.
    - Instances may be terminated if Spot prices exceed your bid or capacity is reclaimed.

### Best Practices
- Use Spot Instances for non-critical, interruptible workloads.
- Diversify instance types and Availability Zones to reduce interruption risk.
- Monitor Spot Instance interruptions via AWS CloudWatch or Spot Fleet events.

---

## 4. AWS Pricing Calculator

### Overview
- The **AWS Pricing Calculator** helps estimate monthly AWS costs based on your configuration.
- **Reference**: [AWS Pricing Calculator](https://calculator.aws/)

### Properties Configured
1. **Service Selection**:
   - Choose services (e.g., EC2, EBS, S3).
2. **Region**: Select the AWS region (e.g., `us-east-1`).
3. **Instance Configuration**:
   - **Instance Type**: e.g., `t2.micro`, `t3.medium`.
   - **Operating System**: Linux, Windows, etc.
   - **Quantity**: Number of instances.
   - **Usage Hours**: Hours per day/month (e.g., 24/7 or 8 hours/day).
4. **Storage**:
   - EBS volume type (e.g., General Purpose SSD - gp3).
   - Storage size (e.g., 8 GB for Free Tier).
5. **Billing Options**:
   - On-Demand, Spot, Reserved Instances, or Savings Plans.
6. **Additional Services**:
   - Data transfer costs (e.g., internet egress/ingress).
   - Elastic IPs (charged when not associated with running instances).
7. **Output**:
   - Estimated monthly cost.
   - Breakdown by service, region, and billing option.

### Best Practices
- Use the calculator to compare On-Demand, Spot, and Reserved Instances/Savings Plans costs.
- Factor in data transfer and storage costs for accurate estimates.

---

## 5. Savings Plans

### Overview
- **Savings Plans** offer significant discounts (up to 72%) compared to On-Demand pricing in exchange for a commitment to use a consistent amount of compute resources (measured in $/hour) over a 1- or 3-year term.
- **Types**:
  - **Compute Savings Plans**: Flexible across instance types, sizes, and regions (e.g., EC2, Fargate, Lambda).
  - **EC2 Instance Savings Plans**: Specific to EC2 instances in a particular region and instance family (e.g., `t3` or `m5`).
- **Reference**: [AWS Savings Plans](https://aws.amazon.com/savingsplans/)

### Steps to Configure a Savings Plan
1. **Go to AWS Cost Management > Savings Plans**.
2. **Choose Plan Type**:
   - Select **Compute Savings Plans** for flexibility or **EC2 Instance Savings Plans** for specific instance families.
3. **Set Commitment**:
   - Specify the hourly commitment (e.g., $10/hour).
   - Choose a 1-year or 3-year term.
4. **Payment Option**:
   - **All Upfront**: Pay the entire amount upfront for maximum savings.
   - **Partial Upfront**: Pay a portion upfront and the rest monthly.
   - **No Upfront**: Pay monthly with slightly lower savings.
5. **Review and Purchase**:
   - Review the estimated savings and coverage.
   - Confirm and purchase the Savings Plan.
6. **Monitor**:
   - Use AWS Cost Explorer to track Savings Plans utilization and coverage.

### Benefits
- Significant cost savings compared to On-Demand pricing.
- Flexibility to change instance types (Compute Savings Plans) or stay within an instance family (EC2 Instance Savings Plans).

---

## 6. Reserved Instances

### Overview
- **Reserved Instances (RIs)** provide discounts (up to 75%) for committing to use specific EC2 instance types in a region for a 1- or 3-year term.
- **Use Case**: Predictable, steady-state workloads.
- **Reference**: [AWS Reserved Instances](https://aws.amazon.com/ec2/pricing/reserved-instances/)

### Payment Options
1. **All Upfront**: Pay the full amount upfront for the highest discount.
2. **Partial Upfront**: Pay a portion upfront and the rest monthly.
3. **No Upfront**: Pay monthly with a lower discount.

### Key Differences from Savings Plans
- RIs are specific to instance type, family, and region, while Compute Savings Plans offer more flexibility.
- RIs are ideal for workloads with fixed instance requirements.

---

## Summary of Steps
1. **Launched Instances**:
   - Windows: `t2.medium`, Microsoft Windows Server 2016, region-specific key pair.
   - Linux: Refer to Session 4 for steps (Amazon Linux, `t2.micro`, SSH port 22).
2. **Instance Types**: Selected based on vCPUs, memory, storage, and bandwidth needs.
3. **Spot Fleet**:
   - Requested Spot Instances with diversified instance types and allocation strategies.
4. **Pricing Calculator**: Configured instance types, storage, and billing options to estimate costs.
5. **Savings Plans and Reserved Instances**:
   - Configured for cost optimization with flexible or fixed commitments.

---

## Key Takeaways
- **EC2**: Core service for launching virtual servers with customizable configurations (AMIs, volumes, snapshots).
- **Instance Types**: Choose based on workload needs (e.g., General Purpose, Compute Optimized, Memory Optimized).
- **Spot Instances**: Cost-effective for interruptible workloads; use Spot Fleets for scalability.
- **Pricing Calculator**: Essential for estimating costs across services and billing options.
- **Savings Plans and Reserved Instances**: Optimize costs for long-term, predictable workloads.
- **Best Practices**:
  - Secure key pairs and never lose `.pem` files.
  - Use Free Tier instances for learning to avoid charges.
  - Configure security groups to allow only necessary ports for production.