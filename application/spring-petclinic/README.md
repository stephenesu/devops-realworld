# DevOps Real World CI/CD Platform

A production-grade DevOps implementation built around the Spring PetClinic application demonstrating a complete software delivery lifecycle.

This project showcases how modern DevOps teams build, analyze, secure, package, and deploy applications using Jenkins, SonarQube, Docker, Kubernetes, Terraform and AWS.

---

# Project Goals

The objective of this repository is to simulate how a real software company delivers applications from source code to production.

The pipeline includes:

- Source Control
- Continuous Integration
- Static Code Analysis
- Quality Gates
- Artifact Packaging
- Docker Image Build
- Container Registry
- Kubernetes Deployment
- Infrastructure as Code
- Monitoring
- Security Scanning

---

# Technology Stack

| Category | Technology |
|-----------|------------|
| Language | Java 17 |
| Framework | Spring Boot |
| Build Tool | Maven |
| SCM | GitHub |
| CI/CD | Jenkins |
| Code Analysis | SonarQube Community |
| Containerization | Docker |
| Container Registry | Amazon ECR |
| IaC | Terraform |
| Cloud | AWS |
| Orchestration | Kubernetes |
| Monitoring | Prometheus + Grafana |
| Reverse Proxy | NGINX |
| Security | Trivy |

---

# Repository Structure

```
application/
    spring-petclinic/

infrastructure/
    docker/
    kubernetes/
    terraform/

docs/

scripts/
```

---

# High Level Architecture

```
Developer
     │
     ▼
GitHub Repository
     │
     ▼
Jenkins Pipeline
     │
     ├───────────────► Maven Build
     │
     ├───────────────► Unit Tests
     │
     ├───────────────► SonarQube Analysis
     │
     ├───────────────► Quality Gate
     │
     ├───────────────► Docker Build
     │
     ├───────────────► Trivy Scan
     │
     ├───────────────► Push to Amazon ECR
     │
     ▼
Kubernetes Cluster
     │
     ▼
Application
```

---

# CI/CD Pipeline

1. Developer pushes code to GitHub

2. Jenkins detects webhook

3. Jenkins checks out source code

4. Maven compiles project

5. Unit tests execute

6. SonarQube performs static analysis

7. Jenkins waits for Quality Gate

8. Docker image is built

9. Image is scanned using Trivy

10. Image is pushed to Amazon ECR

11. Kubernetes deployment is updated

12. Application becomes available

---

# Prerequisites

- Docker
- Docker Compose
- Java 17
- Maven
- Jenkins
- SonarQube
- AWS CLI
- kubectl
- Terraform

---

# Running Locally

Clone the repository

```bash
git clone https://github.com/stephenesu/devops-realworld.git
```

Start the application

```bash
cd application/spring-petclinic

./mvnw spring-boot:run
```

Access

```
http://localhost:8080
```

---

# Running DevOps Stack

```bash
cd infrastructure/docker

docker compose up -d
```

Services

| Service | URL |
|----------|-----|
| Jenkins | http://localhost:8080 |
| SonarQube | http://localhost:9000 |

---

# Infrastructure

Terraform provisions:

- VPC
- Subnets
- Internet Gateway
- NAT Gateway
- Security Groups
- EKS Cluster
- Amazon ECR
- IAM Roles

---

# Kubernetes

Application deployment includes

- Deployment
- Service
- Ingress
- ConfigMaps
- Secrets

---

# Monitoring

Metrics are collected using

- Prometheus
- Grafana

---

# Security

Security scanning includes

- SonarQube
- Trivy

Future additions

- Dependency Check
- Snyk
- OWASP ZAP

---

# Author

Stephen Joe Esu

Cloud Engineer | DevOps Engineer

GitHub

https://github.com/stephenesu
