+++
author = "Eric Ngigi"
title = "Terraform AWS Three Tier Architecture"
date = "2025-10-10"
description = "A Terraform AWS Three Tier Architecture Automation"
tags = [
    "Terraform",
    "IaC",
]
categories = [
    "terraform",
    "shell scripting",
]
series = ["Terraform Guide"]
aliases = ["terraform-guide-01"]
image = "terraform-banner-01.png"
+++

# **1. Overview**

This project implements a **Three-Tier Architecture on AWS**, designed to provide a secure and highly available environment for deploying modern web applications.

The architecture separates components into three logical layers: **Presentation (Web/App Tier)**, **Application (Logic Tier)**, and **Data (Database Tier)** to ensure clear separation of concerns, better security, and easier scalability.

The deployment is fully automated using **Terraform**, enabling consistent, repeatable infrastructure provisioning.

---

### **1.1 Objectives**

The goal of this architecture is to:

* Provide a **secure and isolated environment** for application and database workloads.
* Ensure **high availability** across multiple Availability Zones.
* Maintain **resilience** and fault tolerance by leveraging AWS managed services.
* Enable **infrastructure as code (IaC)** for repeatable and version-controlled deployments.

---

### **1.2 Architectural Layers**

#### **1.2.1 Presentation Layer (Web / Application Tier)**

* Consists of **EC2 instances** running the application or web server.
* Deployed in **private subnets** across multiple Availability Zones.
* Traffic is distributed using an **Application Load Balancer (ALB)** deployed in public subnets.
* The ALB routes HTTP and HTTPS traffic to the backend EC2 instances for processing.

#### **1.2.2 Application / Logic Layer**

* Handles core business logic and communicates with both the presentation and data tiers.
* Uses **Elastic File System (EFS)** for shared storage between instances (e.g., logs, application data, uploads).

#### **1.2.3 Data Layer**

* Uses **Amazon RDS (MySQL)** as the managed relational database engine.
* Deployed in **private subnets** with **Multi-AZ** replication for high availability.
* Database access is restricted to only the application layer via security groups.

---

### **1.3 Key AWS Components**

| **Component**                 | **Purpose**                                                               |
| ----------------------------- | ------------------------------------------------------------------------- |
| **VPC**                       | Provides the isolated network environment.                                |
| **Subnets**                   | Separate public and private tiers across Availability Zones.              |
| **Internet Gateway**          | Enables public internet access for external users.                        |
| **NAT Gateways**              | Allow private resources outbound access without direct internet exposure. |
| **Application Load Balancer** | Distributes traffic across EC2 instances in multiple AZs.                 |
| **EC2 Instances**             | Host the web/application layer.                                           |
| **EFS**                       | Provides shared storage across app instances.                             |
| **RDS (MySQL)**               | Managed relational database for persistent data storage.                  |
| **Security Groups**           | Control inbound and outbound network traffic between layers.              |

---

### **1.4 High-Level Features**

* **Multi-AZ Deployment:** Ensures redundancy and fault tolerance.
* **Secure Networking:** Implements VPC isolation and least privilege access via security groups.
* **Automated Provisioning:** Infrastructure managed using Terraform for consistency and versioning.
* **Centralized Storage:** Shared EFS mounted across app instances for consistent file access.
* **Monitoring & Logging:** Easily integrated with AWS CloudWatch for performance and health tracking.

---

### **1.5 Use Cases**

This architecture is suitable for:

* Web applications requiring a secure and scalable backend.
* Internal enterprise applications with strict access controls.
* Multi-tier deployments (frontend, backend, and database separation).
* Environments that require high availability and fault tolerance across AWS Availability Zones.

---

# **2. Architecture Diagram**

The following diagram illustrates the **Three-Tier AWS Architecture** implemented in this project. It depicts how each layer — **presentation**, **application**, and **data** — is logically and physically separated within the AWS environment to provide scalability, fault tolerance, and security.

---

### **2.1 Diagram**

![AWS Three Tier Architecture](terraform-aws-three-tier-architecture.svg)

---

### **2.2 Architectural Components**

The architecture is divided into multiple **Availability Zones (AZs)** to ensure **high availability** and **fault tolerance**. Each tier performs a specific role as outlined below:

---

#### **2.2.1 Networking Layer**

* **VPC (Virtual Private Cloud)**:
  Provides an isolated network environment for all deployed AWS resources.

* **Subnets**:

  * **Public Subnets** – Host the Application Load Balancer (ALB) and NAT Gateways.
  * **Private Subnets** – Host EC2 application servers and RDS instances.
  * Each subnet is distributed across **two Availability Zones** (e.g., `us-east-1a`, `us-east-1b`) for redundancy.

* **Internet Gateway**:
  Enables inbound and outbound internet access for resources in public subnets.

* **NAT Gateways**:
  Allow EC2 instances in private subnets to access the internet for software updates without exposing them publicly.

---

#### **2.2.2 Presentation Layer (Web Tier)**

* **Application Load Balancer (ALB)**:

  * Acts as the entry point for all HTTP and HTTPS traffic.
  * Deployed in **public subnets** for external accessibility.
  * Routes requests to the EC2 instances in the private subnets via target groups.
  * Supports both **HTTP (port 80)** and **HTTPS (port 443)** for secure communication.

* **Route 53 (DNS)**:

  * (Optional) Provides a custom domain name that maps to the ALB’s DNS.
  * Improves accessibility and branding of the deployed application.

---

#### **2.2.3 Application Layer (App Tier)**

* **EC2 Instances**:

  * Host the web or application logic (e.g., Nginx, Node.js, PHP, etc.).
  * Deployed across multiple private subnets for high availability.

* **EFS (Elastic File System)**:

  * Provides shared storage accessible by all EC2 instances in the app tier.
  * Commonly used for logs, session data, media uploads, or persistent application files.

* **EC2 Security Group**:

  * Allows inbound traffic **only from the ALB** (port 80/443).
  * Restricts outbound access to necessary services (e.g., database or NAT).

---

#### **2.2.4 Data Layer (Database Tier)**

* **Amazon RDS (MySQL)**:

  * Deployed in **private subnets** to ensure database isolation from the public internet.
  * Configured for **Multi-AZ** deployment for failover and data durability.
  * Includes **Primary** and **Replica** instances for read/write separation and redundancy.

* **RDS Security Group**:

  * Allows inbound connections **only from the Application Layer** security group.
  * Prevents direct public or administrative access from the internet.

---

#### **2.2.5 Storage Layer**

* **Amazon EFS (Elastic File System)**:

  * Accessible from all EC2 instances across Availability Zones.
  * Scales automatically to store application files without the need for manual provisioning.
  * Enforced by the **EFS Security Group** to allow only authorized NFS access from the app tier.

---

### **2.3 Traffic Flow Summary**

1. **External users** access the application through a **DNS record (Route 53)** or the **ALB DNS name**.
2. Requests reach the **Application Load Balancer** in the public subnet.
3. The ALB forwards traffic to **EC2 instances** in private subnets via **Target Groups**.
4. The **EC2 application servers** process requests, interact with the **RDS database** and **EFS** as needed.
5. **Outbound internet access** (for software updates or dependency downloads) is routed through the **NAT Gateways**.
6. All communication between layers is controlled using **Security Groups** and restricted by the principle of least privilege.

---

### **2.4 High Availability & Fault Tolerance**

* All tiers are deployed across **two Availability Zones** to ensure continuity in case of an AZ outage.
* **NAT Gateways**, **App Servers**, and **RDS instances** are distributed between zones.
* The **Application Load Balancer** automatically reroutes traffic to healthy targets during failures.

---

### **2.5 Security Considerations**

* No direct SSH or RDP access to EC2 instances from the internet.
* Communication between tiers is restricted via **dedicated Security Groups**.
* The **database and EFS** are deployed in private subnets with no internet exposure.
* HTTPS is recommended for all external traffic to ensure encryption in transit.

---

# **3. Components**

This section describes the core AWS infrastructure components that make up the three-tier architecture.
Each component is modular and defined using **Terraform** for ease of deployment, management, and scalability.

The architecture is divided into **five main categories**:

* **Networking**
* **Compute**
* **Storage**
* **Database**
* **Security**

---

### **3.1 Networking Components**

The networking layer provides the foundation for communication and isolation between resources. It defines the virtual network, subnets, routing, and internet access.

#### **3.1.1 VPC (Virtual Private Cloud)**

* Defines a logically isolated network within AWS.
* CIDR block: typically `10.0.0.0/16` (customizable through variables).
* Hosts all public and private subnets used by other resources.
* Acts as a boundary for all communication and routing.

#### **3.1.2 Subnets**

* **Public Subnets**

  * Host resources that require internet access such as the **Application Load Balancer** and **NAT Gateways**.
  * Associated with a route table that routes traffic to the **Internet Gateway**.

* **Private Subnets**

  * Host internal resources such as **EC2 application servers**, **RDS databases**, and **EFS**.
  * Access to the internet is provided via **NAT Gateways**, not directly.
  * Ensures that private resources remain isolated from public exposure.

#### **3.1.3 Internet Gateway**

* Enables inbound and outbound traffic between the VPC and the internet.
* Attached to the VPC and routes requests from public subnets.

#### **3.1.4 NAT Gateways**

* Deployed in each public subnet for redundancy.
* Allow instances in private subnets to access the internet for updates and package installations without direct exposure.

#### **3.1.5 Route Tables**

* Define how traffic is routed within the VPC.
* Public subnets route through the **Internet Gateway**, while private subnets route through **NAT Gateways**.
* Each subnet is explicitly associated with the appropriate route table.

---

### **3.2 Compute Components**

This layer hosts the application workload and provides compute capacity for the web or app tier.

#### **3.2.1 EC2 Instances (App Servers)**

* Run the web application or middleware logic.
* Deployed in **private subnets** for security.
* Typically configured with Nginx, Apache, Node.js, or PHP-FPM depending on the workload.
* Optionally managed by **Auto Scaling Groups** to maintain elasticity and availability.

#### **3.2.2 Application Load Balancer (ALB)**

* Distributes incoming traffic across multiple EC2 instances in different Availability Zones.
* Operates at **Layer 7 (HTTP/HTTPS)** for intelligent routing.
* Supports:

  * **HTTP (port 80)** for general access.
  * **HTTPS (port 443)** for encrypted communication.
* Integrated with **Target Groups** to register healthy EC2 instances dynamically.
* Provides **health checks** to detect and isolate unhealthy targets.

#### **3.2.3 Target Groups**

* Define the set of EC2 instances (targets) that receive traffic from the ALB.
* Each group listens on the defined application port (e.g., port 80).
* Health checks ensure requests are routed only to healthy targets.

---

### **3.3 Storage Components**

Storage components provide persistent and shared storage solutions for the application layer.

#### **3.3.1 Elastic File System (EFS)**

* Shared network file system accessible by all EC2 instances.
* Used for:

  * Application assets
  * Logs
  * Media uploads
  * Shared configuration files
* Scales automatically with demand.
* Accessible only within the VPC via NFS and secured with an **EFS Security Group**.

---

### **3.4 Database Components**

The database layer provides persistent, managed storage for application data.

#### **3.4.1 Amazon RDS (MySQL)**

* Provides a managed relational database service for the application.
* Deployed in **private subnets** to ensure isolation.
* Configured with:

  * **Primary Database Instance** in one Availability Zone.
  * **Read Replica** or **Multi-AZ Deployment** in another Availability Zone for redundancy and load balancing.
* Automatically handles:

  * Backups
  * Patching
  * Failover
  * Storage scaling

#### **3.4.2 Database Security**

* The RDS instance is associated with a dedicated **MySQL Security Group**.
* Inbound connections allowed only from the **App Server Security Group**.
* No direct internet access permitted.

---

### **3.5 Security Components**

Security is enforced at every layer using AWS-native mechanisms and Terraform-managed configurations.

#### **3.5.1 Security Groups**

Security groups control inbound and outbound traffic to AWS resources.

| **Security Group**     | **Purpose**                  | **Inbound Rules**             | **Outbound Rules** |
| ---------------------- | ---------------------------- | ----------------------------- | ------------------ |
| **EC2 Security Group** | Protects application servers | ALB (HTTP/HTTPS) only         | RDS, EFS, NAT      |
| **ALB Security Group** | Allows user-facing traffic   | HTTP/HTTPS from the internet  | Forward to EC2 SG  |
| **RDS Security Group** | Protects database instances  | App Server SG only            | None (default)     |
| **EFS Security Group** | Secures shared storage       | App Server SG (NFS port 2049) | None (default)     |

#### **3.5.2 Network Access Control Lists (NACLs)**

* Provide an additional layer of subnet-level security.
* Optional but recommended for environments requiring stricter ingress and egress filtering.

#### **3.5.3 IAM Roles and Policies**

* Used to grant EC2 instances permissions to access AWS services securely (e.g., S3, CloudWatch).
* Ensures that no hardcoded credentials are used within instances or configuration files.

---

### **3.6 Optional Components**

* **CloudWatch Monitoring** – For real-time monitoring, alarms, and performance insights.
* **Route 53** – For DNS and custom domain routing to the ALB.
* **Certificate Manager (ACM)** – For managing SSL/TLS certificates when using HTTPS.
* **S3 Buckets** – For centralized log storage or backups.

---

### **3.7 Summary of Component Interactions**

1. **External users** send requests via a DNS name (Route 53 or ALB URL).
2. The **ALB** receives the request and forwards it to EC2 **App Servers** in private subnets.
3. The **App Servers** process the requests, read/write data from the **RDS** database, and access shared files via **EFS**.
4. Any outbound connections (e.g., software updates) go through **NAT Gateways** in the public subnets.
5. **Security Groups** ensure that only permitted communication flows between layers.

---

# **4. Deployment Steps and Future Enhancements**

This section outlines how to deploy the three-tier AWS architecture using Terraform, along with planned improvements for better monitoring, backup, and system management.

---

## **4.1 Deployment Steps**

Follow these steps to provision the infrastructure and deploy your three-tier architecture on AWS.

### **4.1.1 Prerequisites**

Before deployment, ensure the following tools and credentials are set up:

* An **AWS account** with sufficient permissions (AdministratorAccess or equivalent IAM role).
* **Terraform** installed (version 1.6 or higher recommended).
* **AWS CLI** configured with credentials (`aws configure`).
* Access to Terraform modules and variable files defined within this repository.
* Recommended tools:

  * `tflint` for linting Terraform code.
  * `terraform-docs` for generating documentation.
  * `checkov` or `terrascan` for static security scanning.

---

### **4.1.2 Folder Structure**

The repository is organized as follows:

```
.
├── .terraform
│   ├── modules
│   │   └── modules.json
│   └── providers
│       └── registry.terraform.io
├── .terraform.lock.hcl
├── .terraform.log
├── cloud-init
│   ├── configuration
│   │   ├── zsh
│   │   └── zsh-home
│   └── shell-scripts
│       ├── bootstrap.sh
│       └── efs-utils.sh
├── main.tf
├── modules
│   ├── containers
│   │   ├── main.tf
│   │   ├── outputs.tf
│   │   └── variables.tf
│   ├── database
│   │   ├── main.tf
│   │   ├── outputs.tf
│   │   └── variables.tf
│   ├── instances
│   │   ├── main.tf
│   │   ├── outputs.tf
│   │   └── variables.tf
│   ├── management
│   │   ├── main.tf
│   │   ├── outputs.tf
│   │   └── variables.tf
│   ├── network
│   │   ├── main.tf
│   │   ├── outputs.tf
│   │   └── variables.tf
│   ├── notifications
│   │   ├── main.tf
│   │   ├── outputs.tf
│   │   └── variables.tf
│   ├── scaling
│   │   ├── main.tf
│   │   ├── outputs.tf
│   │   └── variables.tf
│   ├── security
│   │   ├── main.tf
│   │   ├── outputs.tf
│   │   └── variables.tf
│   ├── severless
│   │   ├── main.tf
│   │   ├── outputs.tf
│   │   └── variables.tf
│   └── storage
│       ├── main.tf
│       ├── outputs.tf
│       └── variables.tf
├── outputs.tf
├── provider.tf
├── terraform-aws-three-tier-architecture.svg
├── terraform.tfvars
└── variables.tf
```

---

### **4.1.3 Deployment Process**

1. **Clone the Repository**

   ```bash
   git clone https://github.com/erikngigi/terraform-aws-three-tier-architecture
   cd terraform-aws-three-tier-architecture
   ```

2. **Initialize Terraform**

   ```bash
   terraform init
   ```

   This step downloads the necessary Terraform providers and initializes the backend configuration.

3. **Validate the Configuration**

   ```bash
   terraform validate
   ```

   Ensures there are no syntax or configuration errors.

4. **Review the Execution Plan**

   ```bash
   terraform plan -var-file=dev.tfvars
   ```

   Displays all resources that will be created, modified, or destroyed.

5. **Deploy the Infrastructure**

   ```bash
   terraform apply -var-file=dev.tfvars
   ```

   Type `yes` to confirm and deploy the architecture.
   Terraform will provision VPC, subnets, security groups, EC2 instances, load balancer, EFS, and RDS.

6. **Verify the Deployment**

   * Navigate to the **AWS Management Console** → **EC2** → confirm instances are running.
   * Open the **Application Load Balancer DNS name** in a browser (Terraform output variable).
   * Confirm the application or placeholder page is accessible.

7. **Clean Up (Optional)**

   ```bash
   terraform destroy -var-file=dev.tfvars
   ```

   Destroys all infrastructure created by Terraform.

8. **Example of terraform.tfvars**
   Below is an example configuration for the terraform.tfvars file that defines environment-specific parameters: 
   
   ``````bash
   # Project details
    project_name = "project-A"

    # Network details
    vpc_cidr        = "172.0.0.0/16"
    target_alb_port = "80"

    # Security details
    ec2_ingress = {
      "HTTP traffic" = {
        description   = "HTTP"
        port          = 80
        protocol      = "tcp"
        sg_cidr_block = ["0.0.0.0/0"]
        }

    "HTTPS traffic" = {
        description   = "HTTPS"
        port          = 443
        protocol      = "tcp"
        sg_cidr_block = ["0.0.0.0/0"]
        }
    }

    ec2_egress = {
      "ALL" = {
        description   = "All outbound traffic"
        port          = 0
        protocol      = "-1"
        sg_cidr_block = ["0.0.0.0/0"]
        }
    }

    mysql_rds_ingress = {
      "MySQL" = {
        description = "Allow MySQL from EC2 security group"
        port        = 3306
        protocol    = "tcp"
        }
    }

    mysql_rds_egress = {
      "MySQL" = {
        description   = "Allow connections to MySQL RDS"
        port          = 0
        protocol      = "-1"
        sg_cidr_block = ["0.0.0.0/0"]
        }
    }

    efs_ingress = {
      "EFS" = {
        description = "Allow connection to EFS from EC2 only"
        port        = 2049
        protocol    = "tcp"
        }
    }

    efs_egress = {
      "EFS" = {
        description   = "Allow connection from EFS to anywhere"
        port          = 0
        protocol      = "-1"
        sg_cidr_block = ["0.0.0.0/0"]
        }
    }

    # Instance details
    ami_type = "c5.xlarge"

    # Database details
    rds_engine         = "mysql"
    rds_engine_version = "8.0"
    rds_instance_class = "db.t3.micro"
    allocated_storage  = "20"
    rds_username       = "<username>"
    rds_password       = "<password>"
   ``````
---

## **4.2 Future Enhancements**

Although the core architecture is functional, the following enhancements will improve **observability**, **resilience**, and **operational management**.

### **4.2.1 Enable CloudWatch Monitoring**

* Configure **Amazon CloudWatch** to collect and visualize key metrics from:

  * **EC2 instances:** CPU utilization, network I/O, and disk performance.
  * **RDS database:** connections, read/write latency, storage, and free memory.
  * **ALB:** request counts, target response times, and HTTP error rates.
* Set up **CloudWatch Alarms** for threshold breaches (e.g., high CPU, low storage).
* Optionally integrate **CloudWatch Logs** for application and system logs from EC2.

### **4.2.2 Enable AWS Backup for RDS and EFS**

* Implement **AWS Backup** policies to automate and centralize backups for:

  * **RDS databases** (daily incremental and weekly full backups).
  * **EFS file systems** for application data.
* Use **lifecycle policies** to transition older backups to cold storage (Glacier) for cost efficiency.
* Validate recovery by performing periodic restore tests in non-production environments.

### **4.2.3 Use AWS Systems Manager (SSM)**

* Leverage **AWS Systems Manager** for:

  * **Patching** EC2 instances automatically via Patch Manager.
  * **Secure parameter storage** (e.g., DB passwords, API keys) using **Parameter Store**.
  * **Session Manager** to access private EC2 instances securely without SSH keys.
* Integrate Terraform with SSM parameters for dynamic configuration management.

---

### **4.2.4 Additional Considerations**

* **Implement HTTPS:** Add an ACM-issued certificate and configure HTTPS listener on the ALB.
* **Add CI/CD Integration:** Use GitHub Actions or AWS CodePipeline to automate Terraform deployment and testing.
* **Introduce Caching:** Integrate **Amazon ElastiCache (Redis)** to enhance performance for read-heavy workloads.
* **Add Bastion Host or VPN Access:** For secure administrative access to private instances if SSM is not used.

---

### **4.3 Summary**

After following these steps, you will have:

* A fully deployed **three-tier AWS infrastructure** provisioned via Terraform.
* An environment capable of supporting **high availability**, **scalability**, and **security best practices**.
* A roadmap of **enhancements** to improve operational efficiency and monitoring.
