# AWS & DevOps Notes

## 🗕️ Last Session Summary: 04-08-25 (Session-4)

- **VPC**
- **3 Subnets in different Availability Zones**
- **Launched instance in that VPC**
- **Connected to instance by attaching internet gateway in that VPC**

---

## 📘 Session Date: 05-08-2025 (Session-5)

### Scenario Overview:

We are exploring VPC networking in more detail.

Create a VPC with 3 subnets:

- Subnet 1 → EC2 instance (Server1) with **public IP enabled**
- Subnet 2 → EC2 instance (Server2) with **no public IP**

#### Step-by-Step:

1. **Launch Server1:**

   - Amazon Linux, t3.micro
   - Subnet1 (e.g., us-east-1c), enable public IP
   - Security Group: Allow All (Inbound & Outbound)
   - Key pair: Use previously created
   - Launch and wait until running
   - SSH into it using instructions from Session 4

2. **Launch Server2:**

   - Same AMI & instance type as Server1
   - Subnet2, **disable** auto-assign public IP
   - Security Group: Use same as Server1
   - After launching, verify there's **no public IP**

3. **Try to SSH into Server2:**

   - Attempting direct SSH from your local machine using **private IP** results in **timeout**
   - Server2 is not reachable from the internet (as expected)

4. **Access Server2 from Server1 (inside VPC):**

   - On Server1:
     ```sh
     vim keypair.pem  # Paste your private key here
     chmod 400 keypair.pem
     ssh -i keypair.pem ec2-user@<Server2-private-ip>
     ```
   - This works because they are in the same VPC

---

### Internal Access Real-World Analogy:

Just like connecting devices to the same router at home — they communicate using private IPs over LAN even if there's no internet.

---

### Additional Lab:

5. **Launch Windows Instance (Server3) in Subnet3:**

   - Enable public IP
   - Same VPC
   - Security Group: Allow All
   - Install PuTTY on Windows instance
   - Copy `.ppk` file to desktop

6. **Login from Server3 (Windows) to Server1 & Server2 using PuTTY:**

   - Use **private IPs** only
   - Select key pair in PuTTY and open connection

Outcome: Internal communication is possible between all three servers inside the same VPC despite being in different subnets.

---

### Elastic IP Concept:

7. **Stop Server1:**
   - Notice: **Public IP is released**, but **private IP remains**
   - Restarting → New **public IP** (dynamic)

> Problem: If a domain is mapped to this public IP, we must manually update it after every restart.

**Solution:** Use Elastic IP

#### Allocate & Attach Elastic IP:

1. Go to Elastic IP in AWS Console
2. Click `Allocate` → AWS assigns a static IP
3. Go to `Actions` → `Associate`
4. Choose your stopped instance (Server1)
5. Click `Associate` → Done!

> Now, even if you **stop/start**, the public IP won’t change.

**Note:** Elastic IP is **chargeable** when not associated with a running instance.

---

## ✅ Summary:

- Direct SSH to private IPs doesn’t work externally
- You can SSH from **inside** the VPC (via another instance)
- Use **Elastic IP** if you want a persistent public IP across restarts
- Companies prefer **private IP communication** internally
- Common approach: One bastion/public EC2 instance → jump into other private servers

