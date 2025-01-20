# 3-Tier-Board-Project

## 1. Introduction
This bulletin board system is designed with a 3-Tier architecture to ensure efficiency, scalability, and maintainability.

The system allows users to register accounts, enabling them to create and manage posts. The server and database infrastructure are hosted on AWS, ensuring reliability and scalability. This implementation is based on a provided codebase, which has been customized to meet the specific needs of the project.

--- 
## 2. Architecture Design
<img src="https://github.com/user-attachments/assets/6fea22c8-a2d2-49ae-99e2-95dff54107cb" alt="architecture" width="1000px" />
The overall structure is as described above.


<img width="1000" alt="3tier_architecture" src="https://github.com/user-attachments/assets/15a723ee-cddb-42bb-878f-2044d5f190d3" />
It is structured with three tiers: a web server, a WAS (Web Application Server), and a database.


<img width="1000" alt="userflow" src="https://github.com/user-attachments/assets/5b6fe1f4-8a35-4a87-b0ab-083ac94367bb" />

When a user accesses the domain registered in **Route53**, the request is routed through the **Internet Gateway (IGW)** and the **External Application Load Balancer (EX-ALB)** to the **web server**. 

- Requests from the web server to the **WAS (Web Application Server)** are forwarded via the **Internal Application Load Balancer (IN-ALB)**.
- Execution requests are handled by the **WAS**, while data query requests are processed by the **database**.

<img width="1000" alt="admin flow" src="https://github.com/user-attachments/assets/f87d320b-07f7-460d-9fd6-c9a25d9ed3c5" />
The administrator can access and manage the resources in the private subnet through the Bastion server.

---

## 3. Tech Stack
### Infrastructure Components
| Component       | Version/Service                                                   | Description                                         |
|-----------------|-------------------------------------------------------------------|-----------------------------------------------------|
| **Operating System** | <img src="https://img.shields.io/badge/Amazon%20Linux%202-FF9A00?style=flat-square&logo=amazonlinux2&logoColor=white"/> | A secure and lightweight Linux-based OS for cloud environments. |
| **Web Server**      | <img src="https://img.shields.io/badge/Apache%202.4-FF0000?style=flat-square&logo=Apache&logoColor=white"/>            | Open-source web server software to handle HTTP requests. |
| **WAS**             | <img src="https://img.shields.io/badge/Apache%20Tomcat%209.0.78-F8DC75?style=flat-square&logo=apachetomcat&logoColor=black"/> | Application server to execute Java-based web applications. |
| **Database (DB)**   | <img src="https://img.shields.io/badge/MySQL%208.0.33-4479A1?style=flat-square&logo=MySQL&logoColor=white"/>           | Relational database management system for structured data storage. |
| **In-Memory DB**    | <img src="https://img.shields.io/badge/Redis%207.0.7-FF4438?style=flat-square&logo=Redis&logoColor=white"/>            | In-memory key-value store for caching and fast data access. |

---

### AWS Services
| AWS Service            | Logo                                                       | Description                                         |
|-------------------------|-----------------------------------------------------------|-----------------------------------------------------|
| **RDS**                | <img src="https://img.shields.io/badge/Amazon%20RDS-527FFF?style=flat-square&logo=amazonrds&logoColor=white"/>      | Managed relational database service for scalable applications. |
| **ElastiCache for Redis** | <img src="https://img.shields.io/badge/Amazon%20ElastiCache%20for%20Redis-FF4438?style=flat-square&logo=redis&logoColor=white"/> | Fully managed in-memory cache for real-time applications. |
| **CloudWatch**         | <img src="https://img.shields.io/badge/Amazon%20CloudWatch-FF4F8B?style=flat-square&logo=amazonaws&logoColor=white"/> | Monitoring service for AWS resources and custom applications. |
| **Route53**            | <img src="https://img.shields.io/badge/Amazon%20Route%2053-0052CC?style=flat-square&logo=amazons3&logoColor=white"/> | Scalable domain name system (DNS) and traffic management service. |
| **Auto Scaling**       | <img src="https://img.shields.io/badge/AWS%20Auto%20Scaling-232F3E?style=flat-square&logo=amazonaws&logoColor=white"/> | Automatically adjusts capacity to maintain performance and cost-efficiency. |


---


## 4. Implementation Details
### Instance Specifications
| Role            | AMI                   | Instance Type | Key Pair | Public IP | Allowed Ports   | Storage       |
|------------------|-----------------------|---------------|----------|-----------|-----------------|---------------|
| **Bastion**     | Amazon Linux 2 AMI    | t3.micro      | Yes      | Yes       | 22              | 8 GiB (gp3)   |
| **WEB**         | Amazon Linux 2 AMI    | t3.micro      | Yes      | None      | 22, 80          | 8 GiB (gp3)   |
| **WAS**         | Amazon Linux 2 AMI    | t3.medium     | Yes      | None      | 22, 8080        | 8 GiB (gp3)   |


### Naming Rule & Network

#### Naming Convention
- **Format**: `<Organization>-<Subnet>-<AZ>-<Function>`
- **Separator**: `-`
- **Order**: Organization - Subnet - AZ - Function

#### Network Configuration

| VPC             | Region           | AZ | Public Subnet         | Routing Table    | IPv4 CIDR      | Private Subnet       | Routing Table            | IPv4 CIDR      |
|------------------|------------------|----|-----------------------|------------------|----------------|-----------------------|--------------------------|----------------|
| **2MIR-VPC**    | ap-northeast-2   | A  | 2MIR-PRI-A            | PUB-RT           | 172.16.10.0/24 | 2MIR-PRI-A-WEB        | 2MIR-PRI-A-WEB-RT        | 172.16.11.0/24 |
|                  |                  |    |                       |                  |                | 2MIR-PRI-A-WAS        | 2MIR-PRI-A-WAS-RT        | 172.16.12.0/24 |
|                  |                  |    |                       |                  |                | 2MIR-PRI-A-DB         | 2MIR-PRI-A-DB-RT         | 172.16.13.0/24 |
| **2MIR-VPC**    | ap-northeast-2   | C  | 2MIR-PRI-C            | PUB-RT           | 172.16.20.0/24 | 2MIR-PRI-C-WEB        | 2MIR-PRI-C-WEB-RT        | 172.16.21.0/24 |
|                  |                  |    |                       |                  |                | 2MIR-PRI-C-WAS        | 2MIR-PRI-C-WAS-RT        | 172.16.22.0/24 |
|                  |                  |    |                       |                  |                | 2MIR-PRI-C-DB         | 2MIR-PRI-C-DB-RT         | 172.16.23.0/24 |


### ALB Configuration
<img width="1000" alt="ex_in_alb" src="https://github.com/user-attachments/assets/a9678945-28bf-4283-a74d-591628e9f3a1" />

- Traffic on **ports 80 and 443** is routed from the **External ALB (EX-ALB)** to the **web server**.
- Traffic on **port 8080** is routed from the **Internal ALB (IN-ALB)** to the **WAS (Web Application Server)**.
- Target groups were created for:
  - **EX-ALB** on **port 80** and **port 443** to route traffic to the web server.
  - **IN-ALB** on **port 8080** to route traffic to the WAS (Web Application Server).
- An **SSL/TLS certificate** was issued and attached to the **EX-ALB** using **AWS Certificate Manager (ACM)** to enable secure HTTPS connections on **port 443**.
- The load balancers distribute traffic using the **Round Robin (RR)** algorithm to ensure even distribution across target instances.

### ASG Configuration
<img width="1000" alt="asg_group" src="https://github.com/user-attachments/assets/9d3bb3d2-8e24-4998-9975-487b96d88cde"/>

Auto Scaling Groups (ASG) were configured for each layer to handle traffic loads effectively.

### Web Server to WAS Communication

- The communication between the web server and WAS is handled using **mod_proxy**.
- While there are multiple options, such as **mod_proxy** and **mod_jk**, **mod_proxy** was chosen for its versatility and simplicity in configuration.

| Feature                     | **mod_proxy**                                    | **mod_jk**                                 |
|-----------------------------|--------------------------------------------------|-------------------------------------------|
| **Purpose**                 | General-purpose reverse proxy module            | Specialized module for communication with Tomcat servers |
| **Protocol Support**        | Supports HTTP, HTTPS, and AJP protocols          | Primarily supports AJP protocol            |
| **Configuration Simplicity**| Easier to configure                              | More complex, requires additional settings |
| **Flexibility**             | Can be used for non-Tomcat servers as well       | Designed specifically for Tomcat servers   |
| **Performance**             | Slightly lower performance with AJP compared to mod_jk | Optimized for AJP communication            |
| **Load Balancing**          | Built-in support with mod_proxy_balancer         | Built-in support with robust features      |
| **Compatibility**           | Works with Apache HTTP Server 2.2+              | May require additional modules for newer versions |
| **Use Case**                | Recommended for general use and simpler setups   | Ideal for legacy Tomcat applications requiring AJP |


### Session Clustering with Redis
<img width="1000" alt="redis" src="https://github.com/user-attachments/assets/205cb871-7c19-4d7e-bfa4-0ffb426b0b7c"/>

- To maintain session information across multiple WAS instances, **Redis** was used for session clustering.
- The **session manager** in Tomcat was configured to integrate with Redis, enabling seamless session sharing and consistency across all WAS nodes.

---


## 5. Setup & Deployment
### Summary of Steps:
1. Configure EC2 instances.
2. Set up ALB (Application Load Balancer) and Target Groups (TG).
3. Set up **RDS (Relational Database Service)** for the database layer.
4. Set up **ElastiCache (Redis)** for session clustering and caching.
5. Install and configure the web server.
6. Install and configure the WAS (Web Application Server).
7. Create AMIs for the configured EC2 instances and set up Auto Scaling Groups (ASG).
8. Update the Target Groups with the newly created ASG instances.

### 5-1. Configure EC2 instances
<img width="1000" alt="ec2instance" src="https://github.com/user-attachments/assets/0221a836-a989-42f2-b77e-7ff8a4886882" />
The EC2 instances are configured according to the setup described above.

### 5-2. Set up ALB (Application Load Balancer) and Target Groups (TG).
<img width="1000" alt="userflow" src="https://github.com/user-attachments/assets/5b6fe1f4-8a35-4a87-b0ab-083ac94367bb" />

The ALB and Target Groups are configured according to the setup described above.

### 5-3. Set up **RDS (Relational Database Service)** for the database layer.
<img width="1000" alt="image" src="https://github.com/user-attachments/assets/d1e608d2-84c3-4d28-b74c-993853000a56" />

- **RDS MySQL** has been configured within the private subnet as a **Multi-AZ deployment**.

#### **Multi-AZ Deployment Features:**
1. **High Availability**: Ensures minimal downtime by maintaining a standby replica in a different Availability Zone (AZ).
2. **Automatic Failover**: In the event of an AZ failure, the standby instance automatically becomes the primary.
3. **Data Durability**: Synchronous replication between primary and standby instances ensures consistent data.
4. **Read Consistency**: Only the primary instance handles writes, while the standby remains in sync.
5. **Managed Backup**: AWS RDS automatically performs backups on the primary instance, replicated to the standby.

This setup provides enhanced reliability and resilience for database operations.

### 5-4. Set up **ElastiCache (Redis)** for session clustering and caching.


<div style="display: flex; flex-direction: row; gap: 10px;">
    <img width="500" alt="create_redis_cluster" src="https://github.com/user-attachments/assets/c31ccdb0-3300-4cea-890a-6b8ebb170e20" />
    <img width="500" alt="cluster_setting" src="https://github.com/user-attachments/assets/2a1b1e1b-85b5-45cf-9db0-aeab78283b94" />
</div>

- **Cluster Mode**: Disabled to reduce costs.  
  - Enabling cluster mode improves availability and scalability.

- **Node Type**: Configured with the smallest capacity, **cache.t3.micro**.

- **VPC Configuration**: Set to the same VPC as the currently running services.

### 5-5. Install and configure the web server.

#### 5-5-1. **Update the system packages**
   ```bash
   sudo yum update -y
   ```
   
#### 5-5-2. **Install Apache HTTP server**
   ```bash
   sudo yum install httpd -y
   ```

#### 5-5-3. **Enable, start, and check the status of the Apache HTTP server**
   ```bash
   sudo systemctl enable httpd
   sudo systemctl start httpd
   sudo systemctl status httpd
   ```
<img width=800 alt="apachstatus" src="https://github.com/user-attachments/assets/d2b3f51f-4fff-4835-9ddf-49e0da42163e" />

You can verify that the httpd (web server) is running.

#### 5-5-4. **Integrate Apache with Tomcat - Configure Proxy Settings**
   ```bash
   cd /etc/httpd/conf.d
   sudo vi virtualhost.conf
   ```
<img width=800 alt="mod_proxy" src="https://github.com/user-attachments/assets/0ba0af8c-9c71-4a16-a28c-44a2686f6f06"/>

- All incoming traffic on **port 80** is forwarded to the **IN-ALB** for further processing.
  
### 5-6. Install and configure the WAS (Web Application Server).

#### 5-6-1. **Download Tomcat binary and Extract**
   ```bash
   wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.78/bin/apache-tomcat-9.0.78.tar.gz
   tar xvf apache-tomcat-9.0.78.tar.gz 
   ```
#### 5-6-2. **Move tomcat file**
   ```bash
   mv apache-tomcat-9.0.78 ./tomcat9 
   ```

#### 5-6-3. Start and Test Tomcat9 Operation
   ```bash
   cd /usr/local/lib/tomcat9/bin
   ./start.sh
   ```
<div style="display: flex; flex-direction: row; gap: 10px; justify-content: center; align-items: center;">
    <img width="400" alt="tomcat_check" src="https://github.com/user-attachments/assets/c2962126-3cc2-4ead-8347-8f707c84a13b"/>
    <img width="400" alt="tomcat_page" src="https://github.com/user-attachments/assets/dc590776-6728-4158-a834-4b602673e4f7"/>
</div>

+ Verify Tomcat is Running Successfully

#### 5-6-4. Setting Connection with Session Clustering
   ```bash
   wget https://github.com/ran-jit/tomcat-cluster-redis-session-manager/releases/download/2.0.4/tomcat-cluster-redis-session-manager.zip
   sudo cp tomcat-cluster-redis-session-manager/lib/* /usr/local/lib/tomcat9/lib/
   sudo cp tomcat-cluster-redis-session-manager/conf/*
   ```
- Copy Redis Session Manager folder to Tomcat's lib,conf Directory

   ```bash
   sudo vi /usr/local/lib/tomcat9/conf/redis-data-cache.properties
   ```
<img width="1000" alt="properties" src="https://github.com/user-attachments/assets/6410ff4f-b3d9-454a-8283-4ff917f2753f" />

- Fill in the Redis endpoint to be used as the host address.

   ```bash
   sudo vi /usr/local/lib/tomcat9/conf/context.xml
   ```
<img width="1000" alt="context" src="https://github.com/user-attachments/assets/f4db2af5-2b20-4e85-8c43-6ef21ea87bdd" />

- Register Reids handler, manager to Tomcat Conf

#### 5-6-5. Modify and Deploy Source Code
   ```bash
   cd /usr/local/lib
   git clone https://github.com/bigbany/3-Tier-Board-Project.git
   ```
- git clone


<img width="1000" alt="db_setting" src="https://github.com/user-attachments/assets/eb95e160-e13f-4e96-97c7-57ccc39c0b88" />
- Update Database Configuration in `pom.xml`

   ```bash
   cd /usr/local/lib/3-Tier-Board-Project
   ./mvnw package
   ```
<img width=1000 alt="source_build" src="https://github.com/user-attachments/assets/accaaf2e-d954-4258-9c18-1e494fdc33e4">

- Build the Source Code

   ```bash
   sudo cp ./target/petclinic.war /usr/local/lib/tomcat9/webapps
   ```
- Copy the Built WAR File to Tomcat's Default Deployment Directory

<img width=1000 alt="source_output" src="https://github.com/user-attachments/assets/7595e01b-5035-4d80-82d1-c71df64e4086" />
- Verify the Website is running Successfully 

### 5-7. Create AMIs for the configured EC2 instances and set up Auto Scaling Groups (ASG).
- Create AMIs and launch templates for the **Web Server** and **WAS**, then configure Auto Scaling Groups (ASG) with scale-in protection and scaling policies.
- metric type is **Average CPU Utilization** and target value was set to **30**.

### 5-8. Update the Target Groups with the newly created ASG instances.
- Configure ALB Target Groups with ASG

---


## 6. Testing & Results
### Auto Scaling Test with CPU Load

- **Scenario**: Simulated CPU load on the **Web Server** using the `stress` command:
  ```bash
  stress --cpu 2 --timeout 900
---
- Initially, the ASG had 2 instances.
- During the CPU load, the ASG scaled up to 4 instances.
- After stopping the load, the ASG scaled back down to 2 instances.

## 7. Challenges & Improvements
- Integrated **Tomcat** with **Redis** to enable session persistence, ensuring seamless user experiences across multiple server instances.
- Gained a thorough understanding of the **3-Tier Architecture** concept and the roles of **Web Server** and **WAS**.
- Learned how to configure **EC2**, **RDS**, and **ElastiCache**, and developed skills in integrating **Web** and **WAS**, including session management.
- Enhanced capabilities as a cloud engineer through hands-on experience in troubleshooting and resolving issues during these configurations.

---


## 8. References
 [petclinic source code](https://github.com/SteveKimbespin/petclinic_btc/)
 [sample application](fr.slideshare.net/AntoineRey/spring-framework-petclinic-sample-application) 
 [tomcat-redis-session-manager](https://github.com/ran-jit/tomcat-cluster-redis-session-manager)
 

---



