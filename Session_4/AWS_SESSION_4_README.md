# AWS & DevOps Notes

## 📅 Last Session Summary: 01-08-25 (Session-3)

- **Region**
- **Availability Zones**
- **AWS Services**

---

## 📘 Session Date: 04-08-2025 (Session-4)

### 1️⃣ EC2 (Elastic Compute Cloud)

#### 🚀 Launching an Instance (Linux)

- **Instance Name**: `LinuxTestServer`
- **Quick Start**: Amazon Linux
- **AMI**: Amazon Linux 2023 Kernel 6.1
- **Instance Type**: `t3.micro` (Free tier eligible)
- **Key Pair (Login)**:
  - Create key pair and download it (`.pem`)
  - Types: RSA or ED25519
  - Region-specific service

#### 🔐 SSH Access

- `.pem` file: Used with **OpenSSH**
- `.ppk` file: Convert `.pem` using **PuTTYgen** (For use with **PuTTY**)
- Connect via:
  - **Git Bash** or **PuTTY** (For Linux server)

##### PuTTY Steps:

1. Open **PuTTY**
2. In **Host Name**, enter **Public IPv4 address**
3. Go to `Connection → SSH → Auth`
4. Browse and select the `.ppk` key
5. Click **Open**
6. Login as: `ec2-user`
7. Run:
   ```bash
   sudo su
   cd /
   yum update -y
   ```

---

### 🪟 Launching an Instance (Windows)

- **Instance Name**: `WindowsServer`
- **AMI**: Microsoft Windows Server 2016 Base (Free tier)
- **Instance Type**: `t3.small` (Do **not** use `t3.micro` – it lags)
- **Key Pair**: Use the same `.pem` file created earlier

#### 🖥️ RDP Connection

1. In **Instances**, click **Connect** → **RDP Client**
2. Upload `.pem` file
3. Decrypt password
4. Open **Remote Desktop Connection**
5. Enter:
   - **Public IP**
   - **Username**: Administrator
   - **Password**: Decrypted password
6. Click **Yes** to connect

> **Note**: Always **terminate instances** when not in use.  
> **Stopping** an instance still **incurs charges**.

---

### 2️⃣ VPC (Virtual Private Cloud)

> Until now, we used the **default VPC**.

#### ❓ Why Custom VPC?

- Default VPC is required to launch instances.
- Think of VPC like your **home network**.
- Without the internet (gateway), **external devices can’t access your PC**.
- For organizations, custom VPCs offer **security and network control**.

#### 🔧 Prerequisites for VPC Creation

- Decide on the **IP address range**
- Decide:
  - How many IPs are needed?
  - How many subnets are needed?

#### 🧮 CIDR Range Rules

- VPC subnet range: `/16` to `/24`
  - `/8` not supported (1.6M IPs is too large)
- Example:
  - `/16` → ~65,536 IPs
  - `/24` → 256 IPs

---

### 🛠️ Creating a VPC

- Go to **VPC** → Click **Create VPC**
- **Name**: `TestingVPC`
- **CIDR block**: `10.0.0.0/16`
- Error if using `/8`

Use this tool to help: [Subnet Calculator](https://subnet-calculator.com)

---

### 🧱 Subnet Creation

Divide `10.0.0.0/16` into:

| Subnet Name | CIDR Block     | IPs     |
|-------------|----------------|---------|
| Subnet-1    | `10.0.1.0/24`  | 256     |
| Subnet-2    | `10.0.2.0/24`  | 256     |
| Subnet-3    | `10.0.3.0/24`  | 256     |

> Note: Out of 256 IPs in each subnet, **5 are reserved** by AWS  
> So ~**251 usable** IPs per subnet

**Availability Zone Example**: `us-east-1a`

> ❓ *Can we create ~255 subnets from a /16 block?*  
> Yes, approximately.

---

### 🔐 Launching Instance with Security Group

- Create a **Security Group**
- Allow access from **0.0.0.0/0** (for all IPs) – ⚠️ **Use with caution**
- For **controlled access**, restrict this IP range

---

### 🌐 Internet Access Setup

By default, a new VPC has **no internet access**.

#### ✅ Steps to Enable Internet:

1. **Create an Internet Gateway**
2. **Attach** the gateway to your VPC
3. Update **Route Tables**:
   - Add route: `0.0.0.0/0` → Internet Gateway
4. Update **Subnet Associations**:
   - Change from **“Without explicit association”** to **explicitly associate** your subnets

---

### ✅ Summary

| Step | Task                                  |
|------|----------------------------------------|
| 1    | Create VPC: `10.0.0.0/16`              |
| 2    | Create Subnets:                       |
|      | - `10.0.1.0/24`                       |
|      | - `10.0.2.0/24`                       |
|      | - `10.0.3.0/24`                       |
| 3    | Create Internet Gateway                |
| 4    | Attach Internet Gateway to VPC        |
| 5    | Update Route Table with:              |
|      | - Destination: `0.0.0.0/0`            |
|      | - Target: Internet Gateway            |
| 6    | Update Subnet Association: Explicit   |

---

> ✅ Now, your instances will be accessible using their **Public IP address** 🎉
