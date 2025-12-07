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

Create a Web Page
echo "<h1>Instance 1 - Hello from Load Balancer</h1>" | sudo tee /var/www/html/index.html

Test locally: curl http://localhost
4. Repeat on Instance-2

## 🎯 Step-3: Create Target Group

The Target Group contains the backend EC2 servers.
The Load Balancer forwards traffic to this group.

---

### 🟡 1. Open Target Groups
- Go to **EC2 Console**
- Left side menu → **Load Balancing**
- Click **Target Groups**
- Click **Create Target Group**

---

### 🟡 2. Configure Target Group
- Target Type: **Instance**
- Name: `my-web-tg`
- Protocol: **HTTP**
- Port: **80**
- VPC: Select your VPC

---

### 🟡 3. Health Checks
- Protocol: **HTTP**
- Path: `/`
(leave all other options default)

Click **Next**

---

### 🟡 4. Register Targets
You will now see both EC2 instances.

- Select **web-server-1**
- Select **web-server-2**
- Click **Include as pending below**
- Port: **80**

Scroll down → **Create Target Group**

---

### 🟢 Result
Your Target Group is ready.
It contains both Ubuntu servers listening on port 80.

## 🌐 Step-4: Create Application Load Balancer

The Load Balancer will receive traffic from users and forward it to the Target Group.

---

### 🟡 1. Open Load Balancers
- Go to **EC2 Console**
- Left side menu → **Load Balancing**
- Click **Load Balancers**
- Click **Create Load Balancer**

---

### 🟡 2. Select Type
Choose:
- **Application Load Balancer**

Click **Create**

---

### 🟡 3. Basic Settings
- Name: `my-web-alb`
- Scheme: **Internet-facing**
- IP Address Type: **IPv4**

---

### 🟡 4. Network Mapping
Select:
- VPC: **Your VPC**
- Availability Zones:
  - ap-south-1a
  - ap-south-1b
  - ap-south-1c

Select one subnet in each AZ.

---

### 🟡 5. Security Group
- Click **Create new security group**
- Name: `alb-sg`

Inbound Rule:
| Type | Port | Source     |
|------|------|------------|
| HTTP | 80   | 0.0.0.0/0  |

Save the security group and **select it**.

---

### 🟡 6. Listener Configuration
- Listener: **HTTP : 80**
- Forward to Target Group:
  - Select: `my-web-tg`

---

### 🟡 7. Create Load Balancer
Scroll down and click **Create Load Balancer**

It may take **1–2 minutes** to show status **Active**.

---

### 🟢 Result
You have created an **Application Load Balancer** pointin

## 🔐 Step-5: Configure Security Groups (Best Practice)

In a production architecture:
- EC2 instances should **not** be directly accessible on port 80 from the internet.
- Only the **Load Balancer** should be allowed to send HTTP traffic to EC2.

We will configure Security Groups accordingly.

---

### 🟡 1. Load Balancer SG (alb-sg)

Inbound Rules:
| Type | Port | Source     |
|------|------|------------|
| HTTP | 80   | 0.0.0.0/0  |

This allows users on the internet to access the Load Balancer.

---

### 🟡 2. Instance SG (launch-wizard-X or your SG)

Edit inbound rules:
- Remove: `HTTP 80 → 0.0.0.0/0`
- Add new rule:

| Type | Port | Source          |
|------|------|------------------|
| HTTP | 80   | **alb-sg**       |

This means:
- EC2 can receive HTTP traffic **only from Load Balancer**
- Direct browser access to instance public IP will be blocked (expected behavior)

---

### 🟢 Result

Final security f

User → Internet → Load Balancer (HTTP 80) → alb-sg → EC2 Instance (HTTP 80)


Direct:

## 🧪 Step-6: Testing & Validation

Now test the full setup and verify that the Load Balancer is balancing traffic between both EC2 servers.

---

### 🟡 1. Wait for Health Checks
Go to:
- EC2 Console
- Load Balancing → Target Groups
- Click `my-web-tg`
- Open **Targets** tab

You should see:

| Instance     | Status  |
|--------------|----------|
| web-server-1 | Healthy  |
| web-server-2 | Healthy  |

If status shows **unhealthy**, wait 30-60 seconds.

---

### 🟡 2. Find the Load Balancer DNS

Go to:
- EC2 Console
- Load Balancing → Load Balancers
- Select: `my-web-alb`
- Copy **DNS name**

Example: my-web-alb-1234567890.ap-south-1.elb.amazonaws.com


You will see:

- `Instance 1 - Hello from Load Balancer`
- `Instance 2 - Hello from Load Balancer`

Every new refresh will show content from different instances.
This confirms **Round-Robin Load Balancing**.

---

### 🟡 5. CLI Test (Optional)

You can test from inside EC2 as well:

```bash
curl http://my-web-alb-1234567890.ap-south-1.elb.amazonaws.com

User → Load Balancer → Target Group → EC2 Instance 1
                                      → EC2 Instance 2









