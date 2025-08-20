# AWS EC2 Deep Dive 🖥️☁️

## 📌 What is EC2?

EC2 (Elastic Compute Cloud) is a core AWS service that allows users to rent virtual servers (instances) on demand. These instances are virtual machines (VMs) — like Linux or Windows servers.

💡 **Azure equivalent**: Virtual Machines (VMs)

## ⚙️ Key Concepts

### 🧱 Instance Components
- **vCPU & RAM**: Compute and memory allocated
- **Volumes**: Attached storage (EBS)
- **Snapshots**: Backups of volumes
- **AMIs (Amazon Machine Images)**: Preconfigured OS images, also used to clone servers

## 🚀 Launching EC2 Instances

### 🪟 Windows Instance Launch
1. Go to **EC2 > Instances** → Click **Launch Instance**
2. **Name**: Windows-Server
3. **Application & OS Image**: Choose *Microsoft Windows Server 2016 Base*
4. **Instance Type**: For test purposes, use *t2.medium* (⚠️ Not Free Tier)
5. **Key Pair**:
   - Create a new one → *Test-KeyPair*
   - File format:
     - `.pem`: For Linux/UNIX SSH
     - `.ppk`: For Windows via PuTTY
   - 🔥 **Important**: If `.pem` is lost, it cannot be recovered.
6. **Network Settings**:
   - VPC, Subnet
   - Enable Public IP if needed
   - Security Group: Use a restrictive rule in production; for learning, use *Allow All*
7. **Storage**: Customize root volume size
8. Launch and wait until **2/2 checks passed**

### 🐧 Linux Instance Launch
- Same steps as Windows but choose *Amazon Linux 2* as AMI.
- Requires port **22 (SSH)** open in security group
- Use `.pem` key with terminal:
  ```bash
  chmod 400 keypair.pem
  ssh -i keypair.pem ec2-user@<Public-IP>
  ```

## 🧠 EC2 Instance Types

Instance types define compute, memory, storage, and networking capacity.  
Check the official list: [AWS EC2 Instance Types](https://aws.amazon.com/ec2/instance-types)

### 🔸 Categories
- **General Purpose**: Balanced (e.g., *m5*, *t3*)
  - *M7i*: Intel
  - *M7g*: Graviton (ARM)
  - *M7a*: AMD
  - Choose based on compatibility & cost-efficiency
- **Compute Optimized**: High-performance CPUs (*c6i*, *c7g*)
- **Memory Optimized**: RAM-heavy workloads (*r5*, *x2idn*)
- **Storage Optimized**: High local disk I/O (*i4i*, *d3en*)
- **Accelerated Computing**: ML, AI, graphics (*p5*, *inf2*)
- **HPC Optimized**: High-Performance Computing

## 💵 EC2 Pricing & Types

### 1. On-Demand Instances
- Charged by the second/minute
- No upfront payment
- Flexible, but expensive for long-term use

### 2. Spot Instances
- Use spare AWS capacity at up to **90% discount**
- Can be interrupted by AWS with **2 minutes’ notice**
- Best for:
  - Batch processing
  - Fault-tolerant, flexible workloads
- Not suitable for critical or stateful services
- **Launch Spot Fleet**:
  1. Go to **EC2 > Spot Requests**
  2. Click **Create Fleet**
  3. Define:
     - Target capacity
     - Instance types
     - AMI
     - Pricing and max bid
     - Launch settings (key, VPC, SG)
  4. Launch and monitor your fleet

### 3. Reserved Instances (RIs)
- Commit to **1 or 3 years**
- Choose:
  - All Upfront
  - Partial Upfront
  - No Upfront
- Up to **75% cheaper** than On-Demand

### 4. Savings Plans
- More flexible than RIs. Pay for a commitment to usage ($/hour) across instance types.
- Example: Commit to $50/hour usage for 1 year, get discounts on any compute usage (EC2, Fargate, Lambda).
- **Steps**:
  1. Go to **Billing > Savings Plans**
  2. Click **Create Savings Plan**
  3. Choose type:
     - *Compute Savings Plan* (most flexible)
     - *EC2 Instance Savings Plan*
  4. Set term, hourly commitment, payment option
  5. Purchase

## 📊 AWS Pricing Calculator
- URL: [AWS Pricing Calculator](https://calculator.aws.amazon.com)
- Estimate pricing for:
  - Instance type
  - Region
  - OS
  - AMI
  - EBS volume
  - Load balancer
  - Data transfer
  - Savings plan/RI preferences

## 📌 EC2 FAQs & Interview Tips

❓ **Are key pairs global or regional?**  
→ Key pairs are **region-specific**

❓ **Can I recover lost `.pem` file?**  
→ **No**. You must create a new key pair and reconfigure instance access

❓ **What happens to public IP on stop/start?**  
→ Public IP **changes** unless using Elastic IP

❓ **Can I login to a private EC2 from another EC2?**  
→ **Yes**, if they are in the same VPC and Security Group allows it