# SkillPulse 🎯

**A production-grade full-stack skill tracking app — built with Go, MySQL & Nginx, deployed on AWS EC2 using Terraform & GitHub Actions.**

![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Go](https://img.shields.io/badge/Go-00ADD8?style=for-the-badge&logo=go&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)

---

## 🌟 What is SkillPulse?

SkillPulse is a full-stack web application for tracking and managing skills. Beyond the app itself, this project is a **complete DevOps pipeline** — from writing code to deploying live on the internet automatically with a single `git push`.

**One push to `main` → live on AWS in under 2 minutes ⚡**

---

## 🏗️ Architecture

```
   git push main
        │
        ▼
  ┌─────────────────────────────────────┐
  │           GitHub Actions            │
  │  ci.yml ──────────► Docker Hub      │
  │  (build & push)     (image stored)  │
  │       │                  │          │
  │  cd.yml ◄────────────────┘          │
  │  (SSH deploy to EC2)                │
  └─────────────────┬───────────────────┘
                    │
                    ▼
     ┌──────────────────────────────┐
     │   AWS EC2 (t3.medium)        │
     │   Ubuntu 24.04 LTS           │
     │                              │
     │  ┌────────┐  ┌───────────┐   │
     │  │ Nginx  │─►│ Go/Gin    │   │
     │  │  :80   │  │ Backend   │   │
     │  └────────┘  └─────┬─────┘   │
     │                    │         │
     │             ┌──────▼──────┐  │
     │             │  MySQL 8.4  │  │
     │             │  (volume)   │  │
     │             └─────────────┘  │
     └──────────────────────────────┘
              ▲
         Port 80 (HTTP)
           End User
```

---

## 🛠️ Tech Stack

| Layer | Technology | Version |
|---|---|---|
| Backend API | Go + Gin | 1.26 |
| Database | MySQL | 8.4 |
| Frontend | HTML / CSS / JavaScript | — |
| Reverse Proxy | Nginx | Alpine |
| Containers | Docker + Docker Compose | — |
| Cloud Provider | AWS EC2 | t3.medium |
| Infrastructure | Terraform | >= 1.6 |
| CI/CD | GitHub Actions | — |
| Image Registry | Docker Hub | — |
| OS (EC2) | Ubuntu LTS | 24.04 (Noble) |

---

## 📁 Project Structure

```
SkillPulse/
│
├── 📄 docker-compose.yml        # Orchestrates all 3 services
├── 📄 .env.example              # Copy to .env and fill values
├── 📄 .gitignore
│
├── 📂 backend/                  # Go + Gin REST API source code
├── 📂 frontend/                 # HTML/CSS/JS (served by Nginx)
├── 📂 nginx/                    # Nginx reverse-proxy config
├── 📂 mysql/                    # init.sql — DB schema bootstrap
│
├── 📂 terraform/                # AWS infrastructure (run once, locally)
│   ├── main.tf                  # EC2, security group, key pair
│   ├── variables.tf             # region, instance type, repo URL
│   ├── outputs.tf               # public IP, SSH key, app URL
│   ├── user_data.sh.tpl         # EC2 bootstrap script
│   └── terraform.tfvars.example # Copy & fill before apply
│
└── 📂 .github/workflows/
    ├── ci.yml                   # Build & push backend image
    └── cd.yml                   # SSH deploy on EC2
```

---

## 🚀 Getting Started

### Run Locally

```bash
# 1. Clone the repo
git clone https://github.com/<your-username>/skillpulse-devops.git
cd skillpulse-devops

# 2. Set up environment variables
cp .env.example .env
# Edit .env — fill in DB credentials and your Docker Hub username

# 3. Start all services
docker compose up -d

# 4. Visit the app
open http://localhost
```

> **Backend dev tip:** While editing Go code locally, change the `backend` service in `docker-compose.yml` from `image: ...` to `build: ./backend` so it builds from source.

---

## ⚙️ CI/CD Pipeline

### How it works

| Workflow | Trigger | What it does |
|---|---|---|
| `ci.yml` | Push to `main` | Builds Go backend image → pushes `:latest` + `:<git-sha>` to Docker Hub |
| `cd.yml` | After `ci.yml` succeeds | SSHes into EC2 → `git pull` + `docker compose pull` + `docker compose up -d` |

### What reaches EC2 and how

| Change type | Delivered via |
|---|---|
| Backend Go code changes | Docker Hub image (built by CI) |
| Frontend HTML/CSS/JS changes | `git pull` in CD step |
| Nginx config / SQL / Compose file | `git pull` in CD step |

---

## ☁️ AWS Deployment (Terraform)

Terraform is run **once locally** to provision the EC2 instance. All future deploys are automatic via GitHub Actions.

### Steps

```bash
cd terraform

# 1. Fill in your variables
cp terraform.tfvars.example terraform.tfvars

# 2. Initialize and apply
terraform init
terraform apply -auto-approve

# 3. Save the SSH private key
terraform output -raw ssh_private_key > skillpulse-key.pem
chmod 600 skillpulse-key.pem

# 4. Verify EC2 is ready (~90 seconds after apply)
ssh -i skillpulse-key.pem ubuntu@$(terraform output -raw ec2_public_ip) \
  'cloud-init status --wait && docker --version'
```

### Resources Created by Terraform

| Resource | Details |
|---|---|
| EC2 Instance | t3.medium, Ubuntu 24.04 LTS, us-west-2 |
| Security Group | Ports 22 (SSH) + 80 (HTTP) open |
| Key Pair | 4096-bit RSA, auto-generated |
| EBS Volume | 20 GiB gp3, deleted on destroy |

### Tear Down

```bash
terraform destroy -auto-approve
# Zero AWS resources left behind ✅
```

---

## 🔐 Environment Variables

```env
# MySQL
MYSQL_ROOT_PASSWORD=your-root-password
DB_HOST=db
DB_PORT=3306
DB_USER=skillpulse
DB_PASSWORD=your-db-password
DB_NAME=skillpulse

# Docker Hub
DOCKERHUB_USERNAME=your-dockerhub-username
```

> ⚠️ Never commit `.env` to Git — it's already in `.gitignore`

---

## 🔑 GitHub Secrets Required

Set these before the pipeline can run:

```bash
REPO=<your-username>/skillpulse-devops

# Set once — never change
gh secret set EC2_USER           --body "ubuntu"             --repo $REPO
gh secret set DOCKERHUB_USERNAME --body "<your-dh-username>" --repo $REPO
gh secret set DOCKERHUB_TOKEN    --repo $REPO   # paste your Docker Hub PAT

# Refresh after every terraform apply
gh secret set EC2_HOST    --body "$(terraform output -raw ec2_public_ip)" --repo $REPO
gh secret set EC2_SSH_KEY --repo $REPO < skillpulse-key.pem
```

| Secret | Needs Refresh? |
|---|---|
| `EC2_HOST` | ✅ After every `terraform apply` |
| `EC2_SSH_KEY` | ✅ After every `terraform apply` |
| `EC2_USER` | ❌ Always `ubuntu` |
| `DOCKERHUB_USERNAME` | ❌ Never |
| `DOCKERHUB_TOKEN` | ❌ Only if leaked |

---

## 💰 AWS Cost Estimate

| Scenario | Cost |
|---|---|
| Running 24/7 (full month) | ~$33/mo |
| Stopped between sessions | ~$2/mo |
| Per session (apply → use → destroy) | A few cents |

---

## ✅ Validation

Fully tested end-to-end on **2026-04-27**:

- ✅ `terraform apply` — 4 resources in ~20s
- ✅ EC2 cloud-init bootstrap — ~90s
- ✅ CI build + Docker Hub push — ~55s
- ✅ CD SSH deploy — ~38s
- ✅ Backend API (`POST /api/skills`)
- ✅ Frontend change live after push
- ✅ MySQL data persistence (named volume)
- ✅ `terraform destroy` — 0 leftover AWS resources

---



---

<p align="center">
  <b>SkillPulse</b> — Built with Go · Docker · Terraform · GitHub Actions · AWS
</p>
