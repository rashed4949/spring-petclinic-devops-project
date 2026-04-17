# 🐾 Spring PetClinic — CI/CD Pipeline with Jenkins, Ansible & JFrog Artifactory


<img width="2018" height="764" alt="app-pic" src="https://github.com/user-attachments/assets/51ff4746-dea8-4cf3-bad8-d99690d69ef7" />


A fully automated **traditional CI/CD pipeline** built on DigitalOcean cloud infrastructure. Every `git push` to the `main` branch automatically triggers a build, runs tests, stores the artifact in JFrog Artifactory, and deploys the application to a production server using Ansible — with zero manual intervention.

---

## 📋 Table of Contents

- [Architecture Overview](#architecture-overview)
- [Infrastructure](#infrastructure)
- [Pipeline Flow](#pipeline-flow)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Screenshots](#screenshots)
- [What I Learned](#what-i-learned)

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        Developer Machine                        │
│                     git push → main branch                      │
└─────────────────────────┬───────────────────────────────────────┘
                          │  GitHub Webhook (HTTP POST)
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│              Droplet #1 — Jenkins Server                        │
│                                                                 │
│  1. Checkout code from GitHub                                   │
│  2. mvn clean verify (compile + unit tests)                     │
│  3. Upload JAR → JFrog Artifactory                              │
│  4. Trigger Ansible playbook                                    │
└──────────────┬───────────────────────┬──────────────────────────┘
               │ push artifact         │ ansible-playbook
               ▼                       ▼
┌──────────────────────┐   ┌───────────────────────────────────────┐
│  Droplet #2          │   │  Droplet #3 — Production Server       │
│  JFrog Artifactory   │   │                                       │
│                      │◄──│  - Pull JAR from Artifactory          │
│  Maven repo          │   │  - Deploy as systemd service          │
│  petclinic-libs-     │   │  - Health check on port 8080          │
│  release             │   │  - PetClinic live at :8080            │
└──────────────────────┘   └───────────────────────────────────────┘
```

---

## Infrastructure

| Droplet | Size | Role | Port |
|---|---|---|---|
| `jenkins-server` | 4GB RAM / 2 vCPU | Jenkins CI + Ansible controller | 8080 |
| `jfrog-server` | 4GB RAM / 2 vCPU | JFrog Artifactory OSS (artifact store) | 8082 |
| `prod-server` | 2GB RAM / 1 vCPU | Runs Spring PetClinic JAR | 8080 |

All droplets run **Ubuntu 22.04 LTS** on DigitalOcean, provisioned via the `doctl` CLI.

---

## Pipeline Flow

```
git push to main
      │
      ▼
GitHub Webhook fires (POST to Jenkins)
      │
      ▼
Jenkins — Stage 1: Checkout
  └─ Pulls latest code from GitHub
      │
      ▼
Jenkins — Stage 2: Build & Test
  └─ mvn clean verify
  └─ Runs unit + integration tests
  └─ Publishes JUnit test results
      │
      ▼
Jenkins — Stage 3: Upload to Artifactory
  └─ Extracts version from pom.xml dynamically
  └─ Pushes JAR to petclinic-libs-release repo
  └─ Publishes build info to Artifactory
      │
      ▼
Jenkins — Stage 4: Deploy via Ansible
  └─ Reads app_version from pom.xml
  └─ Runs deploy-petclinic.yml playbook
      │
      ▼
Ansible on prod-server
  └─ Stops existing service
  └─ Downloads new JAR from Artifactory
  └─ Writes systemd service file
  └─ Starts petclinic.service
  └─ Waits for port 8080
  └─ HTTP health check on /actuator/health
      │
      ▼
✅ Application live at http://PROD_IP:8080
```

---

## Tech Stack

| Tool | Version | Purpose |
|---|---|---|
| Jenkins | 2.541.3 (LTS) | CI server — build orchestration |
| JFrog Artifactory OSS | Latest | Artifact repository (Maven) |
| Ansible | 2.x | Configuration management & deployment |
| Maven | 3.9+ | Build tool for Java |
| Java | OpenJDK 17 | Runtime & compile target |
| Spring Boot | 3.3.0 | Application framework |
| GitHub Webhooks | — | Triggers Jenkins on push |
| DigitalOcean | — | Cloud provider (3 droplets) |
| Ubuntu | 22.04 LTS | OS on all droplets |
| systemd | — | Process manager for the app |

---

## Project Structure

```
spring-petclinic-devops-project/
├── src/                          # Spring PetClinic source code
├── ansible/
│   ├── inventory.ini             # Ansible inventory (prod server IP)
│   └── deploy-petclinic.yml      # Deployment playbook
├── Jenkinsfile                   # Declarative pipeline definition
├── pom.xml                       # Maven build config
└── README.md
```

---

## Prerequisites

- DigitalOcean account with API token
- `doctl` CLI installed locally
- SSH key uploaded to DigitalOcean
- GitHub account with a fork of this repo
- Basic knowledge of Linux, Java, and shell commands

---

### Screenshots
<img width="1796" height="940" alt="webhooks" src="https://github.com/user-attachments/assets/483c4270-cf13-4821-89a1-e359aa80c050" />
<img width="2559" height="1245" alt="jfrog-artifactory" src="https://github.com/user-attachments/assets/c17c31c5-f1b0-405a-b9bb-92a07d83228d" />
<img width="2544" height="984" alt="jenkins" src="https://github.com/user-attachments/assets/93a41621-bdf2-40f5-9a2d-65e92a5b4ace" />

---

## What I Learned

- How a traditional push-based CI/CD pipeline works end-to-end
- How Jenkins integrates with GitHub via webhooks and API tokens
- How JFrog Artifactory stores and versions build artifacts (Maven layout)
- How Ansible automates deployment over SSH with idempotent tasks
- Debugging systemd services with `journalctl`
- Real-world Jenkins CSRF authentication and how to handle it securely
- Why dynamic version extraction from `pom.xml` is better than hardcoding
- DigitalOcean infrastructure provisioning with `doctl` and firewall management
