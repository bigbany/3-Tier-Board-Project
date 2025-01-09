# 3-Tier-Board-Project

## 1. Introduction
This bulletin board system is designed with a 3-Tier architecture to ensure efficiency, scalability, and maintainability.

The system allows users to register accounts, enabling them to create and manage posts. The server and database infrastructure are hosted on AWS, ensuring reliability and scalability. This implementation is based on a provided codebase, which has been customized to meet the specific needs of the project.

--- 
## 2. Architecture Design
<img src="https://private-user-images.githubusercontent.com/63151655/401084150-92bf8b48-5879-488c-b825-69085c380ec0.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MzYzMjgzNDIsIm5iZiI6MTczNjMyODA0MiwicGF0aCI6Ii82MzE1MTY1NS80MDEwODQxNTAtOTJiZjhiNDgtNTg3OS00ODhjLWI4MjUtNjkwODVjMzgwZWMwLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTAxMDglMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwMTA4VDA5MjA0MlomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTQ1MzBlMjRjYjJiYzhjNDhmNjQ4N2Q2NTdjOGUwNTFlMDNjNDFhOTk5Y2M0YjAxMjY0NTZkZmUyMzNlZmJmNWImWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.V4GYC0No_pyrhBNvIgzin7fRfsoZn4GviuhlpyCb5Uk" alt="architecture" width="1000px" />
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

---


## 5. Setup & Deployment

---


## 6. Testing & Results

---


## 7. Challenges & Improvements

---


## 8. Project Structure

---


## 9. Usage

---


## 10. References

---



