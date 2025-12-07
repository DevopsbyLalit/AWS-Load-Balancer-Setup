# AWS-Load-Balancer-Setup
AWS load 

## 🛠️ Prerequisites

Before you start, make sure you have:

### ✔ AWS Account
You need an active AWS account with IAM user access.

### ✔ Region Selection
Select a region close to you.  
Example: **Asia Pacific (Mumbai) – ap-south-1**

### ✔ IAM Permissions (Minimum)
The user should have permissions for:
- EC2
- VPC
- IAM
- Load Balancer
- Security Groups

### ✔ Basic Knowledge (Recommended)
- Launching EC2 instances
- SSH login into EC2
- Linux commands (basic)
- Security Group rules

---

### 💡 Architecture Summary (Quick)
We will create:

- **2 Ubuntu EC2 Instances**
- Install **Apache2** on both
- Create a **Target Group**
- Create an **Application Load Balancer**
- Configure **Security Groups**
- Test **Load Balancer DNS**

## 🚀 Step-1: Launch Two EC2 Instances

We will launch **two Ubuntu servers**.  
Both servers will run Apache and serve the same webpage.

---

### 🟡 1. Open EC2 Console
- Go to **AWS Console**
- Search: **EC2**
- Click **Instances → Launch Instances**

---

### 🟡 2. Instance Details
- Name: `web-server-1` (first instance)
- AMI: **Ubuntu Server 22.04 LTS**
- Instance type: `t2.micro` (Free Tier)

---

### 🟡 3. Key Pair
- Create a new key pair or select existing
- Type: RSA
- `.pem` file will download

---

### 🟡 4. Network Settings
- VPC: **Default / Your chosen VPC**
- Subnet: **ap-south-1a**
- Auto-assign public IP: **Enable**
- Security Group:
  - Allow **SSH (22)**
  - Allow **HTTP (80)**

Click **Launch**

---

### 🟡 5. Launch Second Instance
Repeat the same steps:

- Name: `web-server-2`
- Subnet: **ap-south-1b**
- Same key pair & security group
- Launch the instance

---

### 🟢 Result
You now have **two Ubuntu EC2 instances** running in **two different Availability Zones**.

Example:
- `web-server-1` → ap-south-1a
- `web-server-2` → ap-south-1b

## 🔥 Step-2: Install Apache on Both EC2 Servers

Now connect to both EC2 instances and install Apache.

---

### 🟡 1. SSH into Instance-1

Copy Public IP from EC2 console and connect:

```bash
ssh -i your-key.pem ubuntu@PUBLIC_IP_OF_INSTANCE_1 / crate a puuty server

Update Packages & Install Apache

sudo apt update -y
sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2



