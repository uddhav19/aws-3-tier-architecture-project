# aws-3-tier-architecture-project
AWS 3-Tier Architecture using EC2, Nginx, Tomcat, and RDS

## Project Overview

This project demonstrates the implementation of a **classic AWS 3-Tier Architecture** using core AWS services. The architecture is designed to be **secure, scalable, and production-aligned**, making it suitable for learning, interviews, and portfolio presentation.

The application follows a clear separation of concerns:

* **Presentation Tier**: Nginx (Public Subnet)
* **Application Tier**: Apache Tomcat (Private Subnet)
* **Database Tier**: Amazon RDS MySQL (Private Subnet)

---

## Architecture Diagram (Logical Flow)
![AWS 3 Tier Architecture](images/3_tier_architecture_diagram.png)

User → Nginx (Public) → Tomcat (Private) → RDS MySQL (Private)

---

## AWS Services Used

* Amazon VPC
* EC2 (Amazon Linux 2)
* Nginx
* Apache Tomcat 9
* Amazon RDS (MySQL)
* Internet Gateway
* NAT Gateway
* Route Tables
* Security Groups
* Elastic IP

---

## Network Design

### VPC Configuration

* **VPC Name**: VPC-3-Tier
* **CIDR Block**: 192.168.0.0/16

### Subnets

| Subnet Name           | Type    | CIDR           |
| --------------------- | ------- | -------------- |
| Public-Subnet-Nginx   | Public  | 192.168.1.0/24 |
| Private-Subnet-Tomcat | Private | 192.168.2.0/24 |
| Private-Subnet-RDS    | Private | 192.168.3.0/24 |

---

## Routing Configuration

### Public Route Table

* 0.0.0.0/0 → Internet Gateway

### Private Route Table

* 0.0.0.0/0 → NAT Gateway

---

## Security Groups

### SG-Nginx-Public

* Inbound:

  * HTTP (80) from 0.0.0.0/0
  * SSH (22) from My IP
* Outbound: All traffic

### SG-Tomcat-Private

* Inbound:

  * 8080 from SG-Nginx-Public
  * 22 from SG-Nginx-Public or Bastion
* Outbound: All traffic

### SG-RDS-Private

* Inbound:

  * 3306 from SG-Tomcat-Private
* Outbound: All traffic

---

## EC2 Setup

### Nginx Server (Public Tier)

**AMI**: Amazon Linux 2
**Subnet**: Public-Subnet-Nginx
**Public IP**: Enabled

Installation:

```bash
sudo yum update -y
sudo amazon-linux-extras install nginx1 -y
sudo systemctl start nginx
sudo systemctl enable nginx
```

---

### Tomcat Server (Application Tier)

**AMI**: Amazon Linux 2
**Subnet**: Private-Subnet-Tomcat
**Public IP**: Disabled

Installation:

```bash
sudo yum install java-11 -y
cd /opt
sudo wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.90/bin/apache-tomcat-9.0.90.tar.gz
sudo tar -xvf apache-tomcat-9.0.90.tar.gz
sudo chmod +x apache-tomcat-9.0.90/bin/*.sh
sudo apache-tomcat-9.0.90/bin/startup.sh
```

Tomcat runs on port **8080**.

---

## RDS Setup (Database Tier)

* **Engine**: MySQL
* **Deployment**: Single-AZ
* **Public Access**: Disabled
* **Subnet Group**: Private-Subnet-RDS
* **Security Group**: SG-RDS-Private

Note down the **RDS Endpoint** for application configuration.

---

## Database Configuration in Tomcat

Edit `context.xml`:

```bash
nano apache-tomcat-9.0.90/conf/context.xml
```

Add:

```xml
<Resource name="jdbc/TestDB"
          auth="Container"
          type="javax.sql.DataSource"
          username="dbuser"
          password="dbpassword"
          driverClassName="com.mysql.cj.jdbc.Driver"
          url="jdbc:mysql://RDS-ENDPOINT:3306/dbname"/>
```

Place MySQL JDBC driver in:

```bash
apache-tomcat-9.0.90/lib/
```

Restart Tomcat after configuration.

---

## Nginx Reverse Proxy Configuration

Edit Nginx configuration:

```bash
sudo nano /etc/nginx/nginx.conf
```

Add:

```nginx
location / {
    proxy_pass http://TOMCAT-PRIVATE-IP:8080;
}
```

Restart Nginx:

```bash
sudo systemctl restart nginx
```

---

## Application Testing

Access the application using:

```
http://<Nginx-Public-IP>
```

Request flow:

```
Client → Nginx → Tomcat → RDS
```

---

## Cost Optimization and Cleanup

After practice:

* Stop or terminate EC2 instances
* Delete NAT Gateway
* Delete RDS instance
* Release Elastic IP
* Delete VPC and related resources

---

## Resume Description

**AWS 3-Tier Architecture Project**

* Designed and implemented a secure 3-tier architecture using VPC, EC2, Nginx, Tomcat, and RDS.
* Configured public and private subnets with NAT Gateway and strict security groups.
* Integrated Java application with MySQL RDS using JDBC and reverse proxy.

---

## Future Enhancements

* Application Load Balancer (ALB)
* Auto Scaling Group for Tomcat
* HTTPS with ACM
* IAM Roles for secure credential management
* CloudWatch monitoring and alarms

---

## Author

Uddhav Hon
