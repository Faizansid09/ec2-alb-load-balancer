# ec2-alb-load-balancer
Self-healing, load-balanced website on AWS using EC2 + ALB with zero-downtime failover.
# 🌐 High-Availability Website on AWS (EC2 + ALB)

## 📖 Project Overview
This project demonstrates a **fault-tolerant, load-balanced web application** deployed entirely on AWS. I deployed a static HTML website across two EC2 instances and placed an Application Load Balancer (ALB) in front of them to distribute traffic automatically. 

**The Challenge:** Build a system where if one server crashes, the website *still works* without any manual intervention.

## 🏗️ Architecture Diagram
User (Browser)
│
▼
Application Load Balancer (web-alb)
│
▼
Target Group (my-web-tg)
│
├──────► EC2 Instance 1 (My-Web-Server-1) [ap-south-1a]
│ ├─ Instance ID: i-0804d82a9146e284f
│ └─ Private IP: 172.31.x.x
│
└──────► EC2 Instance 2 (My-Web-Server-2) [ap-south-1b]
├─ Instance ID: i-0e3a7617b6623f145
└─ Private IP: 172.31.0.35


## 🛠️ Tech Stack
- **Compute:** AWS EC2 (Amazon Linux 2023, `t2.micro`)
- **Web Server:** Apache (httpd)
- **Load Balancing:** AWS Application Load Balancer (Layer 7)
- **Networking:** VPC, Subnets (Multi-AZ), Security Groups
- **Automation:** User-data bash scripts for bootstrap configuration
- **Metadata:** IMDSv2 (Instance Metadata Service v2) for dynamic IP/ID retrieval
- **Monitoring:** Target Group Health Checks

## ⚙️ Key Features Implemented
1. **Zero-Config Automation:** Used EC2 User Data to automatically install Apache and deploy the HTML page—zero manual SSH setup required for initial deployment.
2. **Dynamic Identification:** Each EC2 instance queries the AWS Metadata Service (IMDSv2) to display its unique **Instance ID** and **Private IP** directly on the webpage.
3. **Traffic Distribution:** The ALB uses a Round-Robin algorithm to split incoming requests between the 2 instances.
4. **Self-Healing (Health Checks):** Configured Target Group Health Checks (HTTP:80). When an instance fails, it is automatically detached from the rotation.

## 🧪 High Availability Testing (Results)
### Scenario 1: Normal Operation
- **Action:** Refreshed the ALB DNS name.
- **Result:** The page alternated between *"Hello from Server 1"* and *"Hello from Server 2"*. 
- **Screenshot:** `[Paste Screenshot showing alternating Server 1 / Server 2 here]`

### Scenario 2: Simulated Server Failure
- **Action:** Manually stopped `My-Web-Server-1` via the AWS Console.
- **Result:** The Target Group health check immediately flagged it as `Unhealthy`. The ALB routed 100% of the traffic to `Server 2`. **Zero downtime experienced by the end-user.**
- **Screenshot:** `[Paste Screenshot of Target Group showing one Unhealthy here]`

### Scenario 3: Auto-Recovery (Resume)
- **Action:** Restarted (resumed) the stopped instance.
- **Result:** After 60 seconds, the Target Group automatically marked it as `Healthy` again and the ALB resumed sending traffic to both servers.

## 💡 What I Learned
- **IMDSv2 vs IMDSv1:** How to securely fetch instance metadata using a PUT request token.
- **Importance of AZs (Availability Zones):** Deploying instances across `ap-south-1a` and `ap-south-1b` protects against a single data center failure.
- **The Power of Health Checks:** Infrastructure can be self-healing if you configure it correctly; you don't need to wake up at 3 AM to reboot a server.

## 🚀 How to Reproduce (Quick Setup)
1. Launch 2 Amazon Linux 2023 EC2 instances with a Security Group allowing HTTP (80) and SSH (22).
2. Paste the provided User-Data script during launch.
3. Create a Target Group and register both instances.
4. Create an Internet-facing ALB and attach the Target Group.
5. Access the ALB DNS name in your browser!
