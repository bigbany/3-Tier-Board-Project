# 3-Tier-Board-Project

## 1. Introduction
이 게시판 시스템은 효율성, 확장성, 유지보수성을 보장하기 위해 3-Tier 아키텍처로 설계되었습니다.

사용자는 계정을 등록하여 게시글을 생성하고 관리할 수 있으며, 서버와 데이터베이스 인프라는 AWS에 호스팅되어 있어 신뢰성과 확장성을 확보합니다.
애플리케이션은 제공된 코드베이스를 기반으로 하며, 프로젝트의 특정 요구 사항에 맞게 커스터마이징되었습니다.

--- 
## 2. Architecture Design
<img src="https://github.com/user-attachments/assets/6fea22c8-a2d2-49ae-99e2-95dff54107cb" alt="architecture" width="1000px" />
전반적인 인프라 구성은 위와 같습니다.


<img width="1000" alt="3tier_architecture" src="https://github.com/user-attachments/assets/15a723ee-cddb-42bb-878f-2044d5f190d3" />
전체 구조는 웹 서버, WAS(Web Application Server), 데이터베이스로 구성된 3계층 구조입니다.


<img width="1000" alt="userflow" src="https://github.com/user-attachments/assets/5b6fe1f4-8a35-4a87-b0ab-083ac94367bb" />

Route53에 등록된 도메인에 사용자가 접근하면, 요청은 Internet Gateway(IGW)와 외부 ALB(EX-ALB)를 거쳐 웹 서버로 전달됩니다.

- 웹 서버에서 WAS로의 요청은 내부 ALB(IN-ALB)를 통해 전달되며, 실행 요청은 WAS에서 처리되고 데이터 쿼리는 데이터베이스가 처리합니다.

<img width="1000" alt="admin flow" src="https://github.com/user-attachments/assets/f87d320b-07f7-460d-9fd6-c9a25d9ed3c5" />
관리자는 Bastion 서버를 통해 프라이빗 서브넷의 리소스에 접근하고 관리할 수 있습니다.

---

## 3. Tech Stack
### Infrastructure Components
| Component       | Version/Service                                                   | Description                                         |
|-----------------|-------------------------------------------------------------------|-----------------------------------------------------|
| **Operating System** | <img src="https://img.shields.io/badge/Amazon%20Linux%202-FF9A00?style=flat-square&logo=amazonlinux2&logoColor=white"/> | 클라우드 환경에 적합한 보안적이고 경량화된 Linux 기반 OS|
| **Web Server**      | <img src="https://img.shields.io/badge/Apache%202.4-FF0000?style=flat-square&logo=Apache&logoColor=white"/>            | HTTP 요청 처리를 위한 오픈소스 웹 서버 소프트웨어 |
| **WAS**             | <img src="https://img.shields.io/badge/Apache%20Tomcat%209.0.78-F8DC75?style=flat-square&logo=apachetomcat&logoColor=black"/> | Java 기반 웹 애플리케이션을 실행하는 애플리케이션 서버 |
| **Database (DB)**   | <img src="https://img.shields.io/badge/MySQL%208.0.33-4479A1?style=flat-square&logo=MySQL&logoColor=white"/>           | 구조화된 데이터 저장을 위한 관계형 데이터베이스 시스템 |
| **In-Memory DB**    | <img src="https://img.shields.io/badge/Redis%207.0.7-FF4438?style=flat-square&logo=Redis&logoColor=white"/>            | 캐싱 및 빠른 데이터 접근을 위한 인메모리 키-값 저장소 |

---

### AWS Services
| AWS Service            | Logo                                                       | Description                                         |
|-------------------------|-----------------------------------------------------------|-----------------------------------------------------|
| **RDS**                | <img src="https://img.shields.io/badge/Amazon%20RDS-527FFF?style=flat-square&logo=amazonrds&logoColor=white"/>      | 확장 가능한 애플리케이션을 위한 관리형 관계형 DB 서비스 |
| **ElastiCache for Redis** | <img src="https://img.shields.io/badge/Amazon%20ElastiCache%20for%20Redis-FF4438?style=flat-square&logo=redis&logoColor=white"/> | 실시간 애플리케이션을 위한 완전 관리형 인메모리 캐시 |
| **CloudWatch**         | <img src="https://img.shields.io/badge/Amazon%20CloudWatch-FF4F8B?style=flat-square&logo=amazonaws&logoColor=white"/> | AWS 리소스 및 사용자 애플리케이션에 대한 모니터링 서비스 |
| **Route53**            | <img src="https://img.shields.io/badge/Amazon%20Route%2053-0052CC?style=flat-square&logo=amazons3&logoColor=white"/> | 확장 가능한 DNS 및 트래픽 관리 서비스 |
| **Auto Scaling**       | <img src="https://img.shields.io/badge/AWS%20Auto%20Scaling-232F3E?style=flat-square&logo=amazonaws&logoColor=white"/> | 성능 유지와 비용 효율성을 위한 자동 용량 조정 기능 |


---


## 4. Implementation Details
### 인스턴스 사양
| Role            | AMI                   | Instance Type | Key Pair | Public IP | Allowed Ports   | Storage       |
|------------------|-----------------------|---------------|----------|-----------|-----------------|---------------|
| **Bastion**     | Amazon Linux 2 AMI    | t3.micro      | Yes      | Yes       | 22              | 8 GiB (gp3)   |
| **WEB**         | Amazon Linux 2 AMI    | t3.micro      | Yes      | None      | 22, 80          | 8 GiB (gp3)   |
| **WAS**         | Amazon Linux 2 AMI    | t3.medium     | Yes      | None      | 22, 8080        | 8 GiB (gp3)   |


### 네이밍 규칙 및 네트워크 구성

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


### ALB 설
<img width="1000" alt="ex_in_alb" src="https://github.com/user-attachments/assets/a9678945-28bf-4283-a74d-591628e9f3a1" />

- 포트 80 및 443의 트래픽은 외부 ALB에서 웹 서버로 라우팅
- 포트 8080은 내부 ALB에서 WAS로 라우팅.
- EX-ALB는 80번 포트와 443번 포트를 통해 웹 서버로 트래픽을 라우팅합니다.
- IN-ALB는 8080번 포트를 통해 WAS(Web Application Server)로 트래픽을 전달합니다.
- **AWS Certificate Manager (ACM)**를 사용해 SSL/TLS 인증서를 발급받고, EX-ALB에 연결하여 443번 포트에서 HTTPS 보안 연결을 지원하도록 구성하였습니다.
- 로드 밸런서는 라운드 로빈(Round Robin, RR) 알고리즘을 사용해 트래픽을 각 대상 인스턴스에 고르게 분산시킵니다.

### ASG 설
<img width="1000" alt="asg_group" src="https://github.com/user-attachments/assets/9d3bb3d2-8e24-4998-9975-487b96d88cde"/>

각 계층별로 ASG를 설정하여 트래픽 변화에 유연하게 대응하도록 구성하였습니다.

### 웹 서버 ↔ WAS 통신

- mod_proxy를 사용해 웹 서버에서 WAS로의 트래픽을 전달합니다.
- mod_proxy는 단순하고 범용성이 뛰어나며, Tomcat뿐 아니라 다양한 서버에 사용할 수 있는 장점이 있어 선택되었습니다.

| 항목                          | **mod_proxy**                                                  | **mod_jk**                                             |
|-------------------------------|----------------------------------------------------------------|--------------------------------------------------------|
| **용도**                      | 범용 리버스 프록시 모듈                                        | Tomcat 서버와의 통신을 위한 특화 모듈                  |
| **지원 프로토콜**            | HTTP, HTTPS, AJP 프로토콜 지원                                 | 주로 AJP 프로토콜 지원                                 |
| **설정 용이성**              | 설정이 더 쉬움                                                | 설정이 복잡하며 추가 설정 필요                         |
| **유연성**                    | Tomcat이 아닌 서버에서도 사용 가능                             | Tomcat 전용으로 설계됨                                 |
| **성능**                      | AJP 사용 시 mod_jk보다 성능이 약간 낮음                        | AJP 통신에 최적화되어 있음                             |
| **로드 밸런싱**              | mod_proxy_balancer를 통한 내장 지원                            | 강력한 기능의 내장 로드 밸런싱 지원                    |
| **호환성**                    | Apache HTTP Server 2.2 이상에서 동작                           | 최신 버전에서는 추가 모듈이 필요할 수 있음             |
| **사용 사례**                | 일반적인 사용 및 간단한 구성에 권장                            | AJP가 필요한 레거시 Tomcat 애플리케이션에 적합         |



### Redis 기반 세션 클러스터링
<img width="1000" alt="redis" src="https://github.com/user-attachments/assets/205cb871-7c19-4d7e-bfa4-0ffb426b0b7c"/>

- 여러 WAS 인스턴스 간에 세션 정보를 공유하기 위해 Redis를 활용했습니다.
- Tomcat에 Redis 기반 세션 매니저를 설정하여 일관된 사용자 경험을 보장합니다.

---


## 5. Setup & Deployment
### 단계 요약:
1. EC2 인스턴스 구성
2. ALB (Application Load Balancer) 및 Target Group 설정
3. 데이터베이스 계층을 위한 **RDS (Relational Database Service)** 구성
4. 세션 클러스터링 및 캐싱을 위한 **ElastiCache (Redis)** 구성
5. 웹 서버 설치 및 구성
6. WAS (Web Application Server) 설치 및 구성
7. 구성된 EC2 인스턴스로 AMI 생성 및 Auto Scaling Group(ASG) 설정
8. 새로 생성된 ASG 인스턴스를 Target Group에 반영


### 5-1. EC2 인스턴스 구성
<img width="1000" alt="ec2instance" src="https://github.com/user-attachments/assets/0221a836-a989-42f2-b77e-7ff8a4886882" />
EC2 인스턴스는 위에서 설명한 설정에 따라 구성되었습니다.

### 5-2. ALB 및 Target Group 설정
<img width="1000" alt="userflow" src="https://github.com/user-attachments/assets/5b6fe1f4-8a35-4a87-b0ab-083ac94367bb" />

ALB와 Target Group은 위의 설계에 따라 구성되었습니다.

### 5-3. 데이터베이스 계층을 위한 **RDS** 설정
<img width="1000" alt="image" src="https://github.com/user-attachments/assets/d1e608d2-84c3-4d28-b74c-993853000a56" />

- **RDS MySQL**은 프라이빗 서브넷 내에 **Multi-AZ 배포**로 구성되었습니다.

#### **Multi-AZ 배포 특징:**
1. **고가용성**: 다른 AZ(가용 영역)에 스탠바이 복제본을 유지해 다운타임 최소화
2. **자동 장애 조치**: 장애 발생 시 스탠바이가 자동으로 Primary로 전환됨
3. **데이터 내구성**: Primary와 스탠바이 간 동기식 복제로 데이터 일관성 유지
4. **읽기 일관성**: 쓰기는 Primary에서만 처리되며, 스탠바이는 동기화 상태 유지
5. **백업 관리**: AWS RDS가 Primary에서 백업을 수행하고 이를 스탠바이에 복제

이 구성은 데이터베이스 운영의 신뢰성과 복원력을 강화합니다.

### 5-4. 세션 클러스터링 및 캐싱을 위한 **ElastiCache (Redis)** 설정


<div style="display: flex; flex-direction: row; gap: 10px;">
    <img width="500" alt="create_redis_cluster" src="https://github.com/user-attachments/assets/c31ccdb0-3300-4cea-890a-6b8ebb170e20" />
    <img width="500" alt="cluster_setting" src="https://github.com/user-attachments/assets/2a1b1e1b-85b5-45cf-9db0-aeab78283b94" />
</div>

- **클러스터 모드**: 비활성화 (비용 절감 목적)
  - 활성화 시 가용성과 확장성 향상 가능
- **노드 타입**: **cache.t3.micro** (최소 용량 구성)
- **VPC 구성**: 현재 서비스가 운영 중인 VPC와 동일하게 설정


### 5-5. 웹 서버 설치 및 구성

#### 5-5-1. **시스템 패키지 업데이트**
   ```bash
   sudo yum update -y
   ```
   
#### 5-5-2. **Apache HTTP 서버 설치**
   ```bash
   sudo yum install httpd -y
   ```

#### 5-5-3. **Apache 서버 활성화 및 상태 확인**
   ```bash
   sudo systemctl enable httpd
   sudo systemctl start httpd
   sudo systemctl status httpd
   ```
<img width=800 alt="apachstatus" src="https://github.com/user-attachments/assets/d2b3f51f-4fff-4835-9ddf-49e0da42163e" />

httpd 서버가 정상적으로 실행 중인지 확인할 수 있습니다.

#### 5-5-4. ** Apache와 Tomcat 연동 - Proxy 설정 구성**
   ```bash
   cd /etc/httpd/conf.d
   sudo vi virtualhost.conf
   ```
<img width=800 alt="mod_proxy" src="https://github.com/user-attachments/assets/0ba0af8c-9c71-4a16-a28c-44a2686f6f06"/>

- 80번 포트로 들어오는 모든 트래픽은 IN-ALB로 전달됩니다.
  
### 5-6. WAS (Web Application Server) 설치 및 구성

#### 5-6-1. **Tomcat 바이너리 다운로드 및 압축 해제**
   ```bash
   wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.78/bin/apache-tomcat-9.0.78.tar.gz
   tar xvf apache-tomcat-9.0.78.tar.gz 
   ```
#### 5-6-2. **Tomcat 디렉토리 이동**
   ```bash
   mv apache-tomcat-9.0.78 ./tomcat9 
   ```

#### 5-6-3. Tomcat 실행 및 테스트
   ```bash
   cd /usr/local/lib/tomcat9/bin
   ./start.sh
   ```
<div style="display: flex; flex-direction: row; gap: 10px; justify-content: center; align-items: center;">
    <img width="400" alt="tomcat_check" src="https://github.com/user-attachments/assets/c2962126-3cc2-4ead-8347-8f707c84a13b"/>
    <img width="400" alt="tomcat_page" src="https://github.com/user-attachments/assets/dc590776-6728-4158-a834-4b602673e4f7"/>
</div>

+ Tomcat이 성공적으로 실행되었는지 확인합니다.

#### 5-6-4. 세션 클러스터링 연동 설정
   ```bash
   wget https://github.com/ran-jit/tomcat-cluster-redis-session-manager/releases/download/2.0.4/tomcat-cluster-redis-session-manager.zip
   sudo cp tomcat-cluster-redis-session-manager/lib/* /usr/local/lib/tomcat9/lib/
   sudo cp tomcat-cluster-redis-session-manager/conf/*
   ```
- Redis 세션 매니저 폴더를 Tomcat의 lib 및 conf 디렉토리에 복사

   ```bash
   sudo vi /usr/local/lib/tomcat9/conf/redis-data-cache.properties
   ```
<img width="1000" alt="properties" src="https://github.com/user-attachments/assets/6410ff4f-b3d9-454a-8283-4ff917f2753f" />

- 사용할 Redis 엔드포인트를 설정

   ```bash
   sudo vi /usr/local/lib/tomcat9/conf/context.xml
   ```
<img width="1000" alt="context" src="https://github.com/user-attachments/assets/f4db2af5-2b20-4e85-8c43-6ef21ea87bdd" />

- Redis 핸들러 및 매니저를 Tomcat에 등록

#### 5-6-5. Modify and Deploy Source Code
   ```bash
   cd /usr/local/lib
   git clone https://github.com/bigbany/3-Tier-Board-Project.git
   ```
- git clone


<img width="1000" alt="db_setting" src="https://github.com/user-attachments/assets/eb95e160-e13f-4e96-97c7-57ccc39c0b88" />
- `pom.xml` 파일에서 DB 설정 수정

   ```bash
   cd /usr/local/lib/3-Tier-Board-Project
   ./mvnw package
   ```
<img width=1000 alt="source_build" src="https://github.com/user-attachments/assets/accaaf2e-d954-4258-9c18-1e494fdc33e4">

- 소스 코드 빌드

   ```bash
   sudo cp ./target/petclinic.war /usr/local/lib/tomcat9/webapps
   ```
- WAR 파일을 Tomcat 배포 디렉토리에 복사

<img width=1000 alt="source_output" src="https://github.com/user-attachments/assets/7595e01b-5035-4d80-82d1-c71df64e4086" />
- 웹사이트 정상 작동 확인

### 5-7. AMI 생성 및 ASG 구성
- Web Server와 WAS 인스턴스를 기반으로 AMI 및 Launch Template 생성
- Scale-In 보호 및 Auto Scaling 정책 적용
- **CPU 평균 사용률 30%**를 기준으로 설정

### 5-8. Target Group에 ASG 인스턴스 등록
- ALB Target Group을 ASG와 연동하여 트래픽을 받을 수 있도록 구성

---


## 6. 테스트 및 결과
### Auto Scaling 테스트 (CPU 부하)
- **시나리오**: stress 명령어를 사용하여 Web Server에 CPU 부하 시뮬레이션
  ```bash
  stress --cpu 2 --timeout 900
---
- 초기 ASG 인스턴스 수: 2
- 부하 중 ASG가 4개 인스턴스로 자동 확장
- 부하 종료 후 다시 2개로 축소됨

## 7. Challenges & Improvements
- **Tomcat**과 **Redis**를 연동하여 **세션 지속성**을 구현하고, 여러 서버 인스턴스 간 **끊김 없는 사용자 경험**을 보장하였습니다.
- **3-Tier 아키텍처** 개념과 **웹 서버(Web Server)** 및 **WAS**의 역할에 대해 깊이 있게 이해하게 되었습니다.
- **EC2**, **RDS**, **ElastiCache** 설정 방법을 익혔고, **웹(Web)**과 **WAS**의 연동 및 **세션 관리**에 대한 실무 역량을 개발하였습니다.
- 이러한 구성 과정에서 발생한 문제를 해결하며 **클라우드 엔지니어로서의 실전 역량**을 강화하였습니다.


---


## 8. References
 [petclinic source code](https://github.com/SteveKimbespin/petclinic_btc/)
 [sample application](fr.slideshare.net/AntoineRey/spring-framework-petclinic-sample-application) 
 [tomcat-redis-session-manager](https://github.com/ran-jit/tomcat-cluster-redis-session-manager)
 

---



