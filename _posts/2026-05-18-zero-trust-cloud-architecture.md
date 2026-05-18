
---
layout: post
title: "Building a Zero-Trust 3-Tier Network with Terraform & LocalStack"
date: 2026-05-18
categories: [Cloud-Security, Infrastructure-As-Code]
tags: [Terraform, Zero-Trust, AWS, LocalStack, Networking]
author: "Rao Tariq Jameel"
---

Perimeter security is no longer enough. If an attacker breaches the exterior firewall of a traditional network, they gain full lateral access to everything inside. 

To solve this, I built and audited a **0-Trust 3-Tier Cloud VPC Network Architecture** using **Terraform** and **LocalStack**. By implementing a *Never Trust, Always Verify* posture, this design enforces strict isolation between the public web interface, application logic, and backend data storage.

---

## 🏗️ Subnet Layout & Traffic Flow

The architecture divides a `/16` VPC into three distinct `/24` subnets, forcing traffic to move sequentially through isolated security zones.

```text
       [ Public Internet (0.0.0.0/0) ]
                      │
              ( Ingress: Port 443 )
                      ▼
┌──────────────────────────────────────────────────────────┐
│ TIER 1: PUBLIC WEB INGRESS SUBNET (10.0.1.0/24)         │
│ 🛡️ SG Policy: Inbound 443 | Outbound 0.0.0.0/0 (Global)  │
└─────────────────────┬────────────────────────────────────┘
                      │
             ( Ingress: Port 8080 )
             ( Identity: Web SG Only )
                      ▼
┌──────────────────────────────────────────────────────────┐
│ TIER 2: PRIVATE APP-LEVEL SUBNET (10.0.2.0/24)          │
│ 🛡️ SG Policy: Inbound 8080 | Outbound 10.0.0.0/16 (VPC)  │
└─────────────────────┬────────────────────────────────────┘
                      │
             ( Ingress: Port 5432 )
             ( Identity: App SG Only )
                      ▼
┌──────────────────────────────────────────────────────────┐
│ TIER 3: PRIVATE ISOLATED DATABASE SUBNET (10.0.3.0/24)   │
│ 🛡️ SG Policy: Inbound 5432 | Outbound: DEFAULT-DENY (None)│
└──────────────────────────────────────────────────────────┘

Rather than using volatile IP whitelisting, I implemented Identity-Based Security Group Chaining. Firewalls at each layer explicitly evaluate the cryptographic source security group token instead of the source IP address. If the proper identity badge isn't present, the packets are instantly dropped.
🛠️ Infrastructure-as-Code Blueprints

The environment is entirely declared in a monolithic Terraform configuration for clean, deterministic deployments.
1. Network Topology (network.tf)
Terraform

resource "aws_vpc" "main_vpc" {
  cidr_block           = "10.0.0.0/16" 
  enable_dns_hostnames = true          
}

resource "aws_subnet" "web_subnet" {
  vpc_id            = aws_vpc.main_vpc.id 
  cidr_block        = "10.0.1.0/24"       
  availability_zone = "us-east-1a"        
}

resource "aws_subnet" "app_subnet" {
  vpc_id            = aws_vpc.main_vpc.id 
  cidr_block        = "10.0.2.0/24"       
  availability_zone = "us-east-1a"        
}

resource "aws_subnet" "db_subnet" {
  vpc_id            = aws_vpc.main_vpc.id 
  cidr_block        = "10.0.3.0/24"       
  availability_zone = "us-east-1a"        
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main_vpc.id
}

resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.main_vpc.id
  route {
    cidr_block = "0.0.0.0/0"             
    gateway_id = aws_internet_gateway.igw.id 
  }
}

resource "aws_route_table_association" "web_assoc" {
  subnet_id      = aws_subnet.web_subnet.id
  route_table_id = aws_route_table.public_rt.id
}

2. Firewall Policies (security_groups.tf)
Terraform

resource "aws_security_group" "web_sg" {
  name        = "web-security-group"
  vpc_id      = aws_vpc.main_vpc.id
  
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"                  
    cidr_blocks = ["0.0.0.0/0"]          
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"                   
    cidr_blocks = ["0.0.0.0/0"]          
  }
}

resource "aws_security_group" "app_sg" {
  name        = "app-security-group"
  vpc_id      = aws_vpc.main_vpc.id
  
  ingress {
    from_port       = 8080               
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [aws_security_group.web_sg.id] # Chained Identity
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["10.0.0.0/16"]        
  }
}

resource "aws_security_group" "db_sg" {
  name        = "db-security-group"
  vpc_id      = aws_vpc.main_vpc.id
  
  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.app_sg.id] # Chained Identity
  }
  # Egress omitted entirely to enforce a true outbound Default-Deny posture
}

🔬 Empirical Verification & Security Auditing

To prove the security model under active conditions, I ran verification tests from inside the container network namespace.

    Test 1: Public Web Ingress

        Action: Sent a request to Tier 1 Web (10.0.1.4) on port 443.

        Result: curl: (7) Failed to connect to 10.0.1.4 port 443: Connection refused

        Proof: PASS. The firewall let the packet pass smoothly to the host; the connection was only refused because no application daemon (like Nginx) was listening yet.

    Test 2: Unmapped Port Posture

        Action: Attempted to scan MySQL port 3306 on the Tier 3 database (10.0.3.4).

        Result: curl: (28) Connection timed out after 5001 milliseconds

        Proof: PASS. Since port 3306 wasn't explicitly opened in the configuration, the security group silently dropped the traffic without returning an error code.

    Test 3: Lateral Movement Block

        Action: Attempted to bypass Tier 2 and hit the database port 5432 directly from an unauthorized zone.

        Result: curl: (28) Connection timed out

        Proof: PASS. Because the traffic lacked the cryptographic token of the Tier 2 Application Security Group, the database firewall dropped the connection instantly.

🛠️ Real-World Engineering Hurdles Resolved

    Local Host Network Blindspot

        Issue: Running commands from my native Ubuntu terminal to the 10.0.0.0/16 addresses resulted in complete packet loss.

        Cause: The subnets live within LocalStack's isolated container network namespace, which my host OS routing table cannot resolve.

        Fix: Tunneled directly into the runtime containers via Docker (sudo docker exec -it localstack_main /bin/bash) to execute the test suite natively from within the shared environment plane.

    Stateful Traffic Deadlock

        Issue: Ingress tests to the web server timed out continuously.

        Cause: The web security group's outbound rules were originally over-restricted to internal VPC ranges (10.0.0.0/16). While ingress was permitted, the server's return handshake packet was trapped and dropped at the egress perimeter.

        Fix: Opened the Web Tier egress rule block to 0.0.0.0/0 to allow legitimate global return paths.

    Git Upstream and Auth Refusal

        Issue: Pushing code returned a fatal Repository not found error.

        Cause: The local repository was bound to a template placeholder string, and GitHub rejects standard passwords over HTTPS for terminal interactions.

        Fix: Cleared the defunct tracking location (git remote remove origin), linked the actual target URL, and utilized a secure Personal Access Token (PAT) to complete the remote deployment.

🎯 Wrap Up

This project demonstrates hands-on mastery of declarative cloud networking, automated infrastructure provisioning, identity-aware firewall chaining, and granular defensive controls.
