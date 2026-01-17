# ChefAI Infrastructure

This repository contains the **Infrastructure as Code (IaC)** for ChefAI, managed using
**Terraform** and deployed on **DigitalOcean**.

It is responsible for provisioning and managing all shared cloud resources, including:

- DigitalOcean App Platform applications (Staging and Production)
- Managed PostgreSQL 16 with pgvector
- DigitalOcean Container Registry (DOCR)
- DigitalOcean Spaces (Object Storage)
- DNS and SSL configuration
- Supporting services required by ChefAI

This repository is environment-agnostic and is designed to support multiple environments
(e.g., dev, staging, production) through variable configuration.

---

## Requirements

- **Terraform 1.14.x**
- DigitalOcean CLI (`doctl`) configured with appropriate credentials
- A DigitalOcean account with API token
- Docker (for local image builds)
- git

---

## Technology Stack

| Component          | Version/Service           | Rationale                                                     |
| ------------------ | ------------------------- | ------------------------------------------------------------- |
| **IaC**            | Terraform 1.14.x          | Declarative infrastructure management                         |
| **Cloud Provider** | DigitalOcean              | Cost-efficient PaaS for MVP ($55/mo target)                   |
| **Compute**        | App Platform              | Serverless containers with auto-scaling, SSL, DDoS protection |
| **Database**       | Managed PostgreSQL 16     | pgvector support for vector similarity search                 |
| **Registry**       | Container Registry (DOCR) | Immutable image storage for "Build Once, Deploy Twice"        |
| **Storage**        | Spaces                    | CDN-backed object storage for static assets                   |

---

## Architecture Overview

ChefAI uses a **PaaS-first architecture** on DigitalOcean to minimize DevOps overhead while
maximizing developer velocity.

### Why DigitalOcean over AWS?

For an MVP operating under constrained resources, DigitalOcean provides:

| Feature       | DigitalOcean       | AWS Equivalent | Cost Comparison      |
| ------------- | ------------------ | -------------- | -------------------- |
| Compute       | App Platform       | EKS + Fargate  | ~$30/mo vs ~$100+/mo |
| Database      | Managed PostgreSQL | RDS            | $15/mo vs $15-30/mo  |
| Control Plane | Included           | $72/mo (EKS)   | $0 vs $72/mo         |
| Load Balancer | Included           | ALB            | $0 vs ~$20/mo        |
| NAT Gateway   | Not needed         | Required       | $0 vs ~$30/mo        |

**Estimated Monthly Cost:** $50-60 (well within $40-90 budget)

### Environment Strategy

```
┌────────────────────────────────────────────────────────────┐
│                    DigitalOcean Project                    │
├────────────────────────────────────────────────────────────┤
│                                                            │
│         ┌──────────────────┐   ┌─────────────────┐         │
│         │   STAGING APP    │   │  PRODUCTION APP │         │
│         │  (Basic Tier)    │   │   (Pro Tier)    │         │
│         │                  │   │                 │         │
│         │ • Backend (512MB)│   │ • Backend (1GB) │         │
│         │ • Frontend(512MB)│   │ • Frontend(1GB) │         │
│         └────────┬─────────┘   └────────┬────────┘         │
│                  │                      │                  │
│                  └──────────┬───────────┘                  │
│                             │                              │
│                  ┌──────────▼──────────┐                   │
│                  │  SHARED DATABASE    │                   │
│                  │  PostgreSQL 16      │                   │
│                  │  (pgvector enabled) │                   │
│                  │                     │                   │
│                  │ • staging_db        │                   │
│                  │ • production_db     │                   │
│                  └─────────────────────┘                   │
│                                                            │
│                  ┌─────────────────────┐                   │
│                  │ CONTAINER REGISTRY  │                   │
│                  │      (DOCR)         │                   │
│                  │                     │                   │
│                  │ • chefai-backend    │                   │
│                  │ • chefai-frontend   │                   │
│                  └─────────────────────┘                   │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Terraform Version Policy

ChefAI Infrastructure is standardized on **Terraform 1.14.x**.

All contributors and automation must use Terraform `1.14.x` to ensure consistent provider
behavior, state handling, and plan output.

Verify your version:

```bash
terraform --version
```

Expected output:

```text
Terraform v1.14.3
```

---

## DigitalOcean CLI Setup

Install and configure `doctl`:

```bash
# Install doctl (macOS)
brew install doctl

# Install doctl (Linux)
snap install doctl

# Authenticate
doctl auth init
```

You will be prompted to enter your DigitalOcean API token.

---

## Repository Structure

```text
TBD
```

---

## Terraform State Management

Terraform state **must not** be stored locally or committed to Git.

This project uses a **remote backend** with DigitalOcean Spaces:

```hcl
terraform {
  backend "s3" {
    endpoint                    = "nyc3.digitaloceanspaces.com"
    bucket                      = "chefai-terraform-state"
    key                         = "terraform.tfstate"
    region                      = "us-east-1"  # Required but ignored by DO
    skip_credentials_validation = true
    skip_metadata_api_check     = true
    skip_region_validation      = true
  }
}
```

### State Locking

For state locking, consider using:

- A separate PostgreSQL table for locks
- Terraform Cloud (free tier available)
- Manual coordination for small teams

---

## Configuration

All required Terraform input variables are documented in the template file:

```text
terraform.tfvars.template
```

To get started:

```bash
cp terraform.tfvars.template terraform.tfvars
```

### Required Variables

```hcl
# DigitalOcean API Token
do_token = "dop_v1_..."

# Project Configuration
project_name = "chefai"
region       = "nyc3"

# Database Configuration
db_size = "db-s-1vcpu-1gb"  # Smallest managed DB

# App Platform Configuration
staging_instance_size    = "basic-xxs"   # 512MB RAM
production_instance_size = "professional-xs"  # 1GB RAM

# Container Registry
registry_name = "chefai-registry"
```

⚠️ The `terraform.tfvars` file may contain sensitive values and **must never be committed**.

---

## Initial Setup

### 1. Create DigitalOcean Project

```bash
doctl projects create --name "ChefAI" --purpose "AI Cooking Assistant"
```

### 2. Initialize Terraform

```bash
terraform init
```

This will:

- Download required providers (digitalocean/digitalocean)
- Initialize the configured backend
- Generate or update `terraform.lock.hcl`

The `terraform.lock.hcl` file **must be committed** to ensure consistent provider versions
across all environments.

---

### 3. Plan Changes

```bash
terraform plan
```

Review the execution plan carefully before applying any changes.

---

### 4. Apply Changes

```bash
terraform apply
```

Only apply changes from a trusted environment with valid DigitalOcean credentials.

---

## Resource Provisioning Details

### Managed PostgreSQL 16

```hcl
resource "digitalocean_database_cluster" "postgres" {
  name       = "chefai-db"
  engine     = "pg"
  version    = "16"
  size       = "db-s-1vcpu-1gb"
  region     = "nyc3"
  node_count = 1

  # Enable pgvector extension after creation
}
```

After provisioning, enable pgvector:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

### Container Registry

```hcl
resource "digitalocean_container_registry" "chefai" {
  name                   = "chefai-registry"
  subscription_tier_slug = "basic"  # 5GB, 5 repos
}
```

### App Platform (Example App Spec)

App Platform applications are defined via YAML App Specs:

```yaml
# chefai-production.yaml
name: chefai-production
region: nyc
services:
  - name: backend
    image:
      registry_type: DOCR
      repository: chefai-backend
      tag: ${IMAGE_TAG}
    instance_size_slug: professional-xs
    instance_count: 1
    http_port: 8000
    routes:
      - path: /api
    envs:
      - key: DATABASE_URL
        value: ${DATABASE_URL}
        type: SECRET

  - name: frontend
    image:
      registry_type: DOCR
      repository: chefai-frontend
      tag: ${IMAGE_TAG}
    instance_size_slug: professional-xs
    instance_count: 1
    http_port: 3000
    routes:
      - path: /
```

---

## Environments

Environment separation is handled via:

- Separate `terraform.tfvars` files per environment
- Separate remote state backends (different keys in Spaces)
- Logical database separation (same cluster, different databases)
- App Spec files per environment (`chefai-staging.yaml`, `chefai-production.yaml`)

### Logical Database Separation

A single managed PostgreSQL cluster hosts both environments:

```sql
-- Create logical databases
CREATE DATABASE chefai_staging;
CREATE DATABASE chefai_production;

-- Create environment-specific users
CREATE USER staging_user WITH PASSWORD '...';
CREATE USER production_user WITH PASSWORD '...';

-- Grant permissions
GRANT ALL PRIVILEGES ON DATABASE chefai_staging TO staging_user;
GRANT ALL PRIVILEGES ON DATABASE chefai_production TO production_user;
```

---

## CI/CD Integration

### GitHub Actions Secrets

Configure the following secrets in your GitHub repository:

| Secret                      | Description                                 |
| --------------------------- | ------------------------------------------- |
| `DIGITALOCEAN_ACCESS_TOKEN` | API token for doctl and Terraform           |
| `DATABASE_URL_STAGING`      | PostgreSQL connection string for staging    |
| `DATABASE_URL_PRODUCTION`   | PostgreSQL connection string for production |

### Deployment Workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build and Push to DOCR
        uses: digitalocean/action-doctl@v2
        with:
          token: ${{ secrets.DIGITALOCEAN_ACCESS_TOKEN }}

      - run: |
          doctl registry login
          docker build -t registry.digitalocean.com/chefai/backend:${{ github.sha }} .
          docker push registry.digitalocean.com/chefai/backend:${{ github.sha }}
```

---

## Cost Management

### Monthly Bill of Materials (Target: $55/mo)

| Resource              | Service Detail                                  | Cost       |
| --------------------- | ----------------------------------------------- | ---------- |
| **Database**          | Managed PostgreSQL (1GB RAM, 1 vCPU, 10GB Disk) | $15.00     |
| **Registry**          | Container Registry (Basic - 5GB)                | $5.00      |
| **Compute (Prod)**    | App Platform Pro - 2 Services (1GB each)        | $20.00     |
| **Compute (Staging)** | App Platform Basic - 2 Services (512MB each)    | $10.00     |
| **Storage**           | Spaces (Optional - 250GB)                       | $5.00      |
| **Total**             |                                                 | **$55.00** |

### Scaling Triggers

| Bottleneck       | Trigger                              | Upgrade Path                  | New Cost |
| ---------------- | ------------------------------------ | ----------------------------- | -------- |
| Database RAM     | pgvector index exceeds available RAM | Upgrade to 2GB RAM            | +$15/mo  |
| Frontend CPU     | SSR load causes CPU limits           | Add 1 replica                 | +$10/mo  |
| Registry Storage | Too many historical images           | Run garbage collection script | $0       |

---

## Security Notes

- Terraform state files may contain sensitive data and must be stored securely
- No credentials, secrets, or `.tfvars` files are committed to Git
- DigitalOcean credentials should be provided via:
  - Environment variables (`DIGITALOCEAN_TOKEN`)
  - `doctl` authentication
  - CI/CD secrets
- Application secrets are injected via App Platform environment variables

---

## Version Control Rules

- `terraform.lock.hcl` **must be committed**
- `.tfstate` files are **never committed**
- `.tfvars` files are **never committed**
- `.terraform/` directories are ignored
- App Spec YAML files **should be committed** (without secrets)

---

## Suggested Additions

Consider adding the following as the project matures:

- [ ] **Monitoring:** DigitalOcean Monitoring alerts and dashboards
- [ ] **Backups:** Automated database backup configuration
- [ ] **Disaster Recovery:** Runbooks for common failure scenarios
- [ ] **Cost Alerts:** Budget notifications via DigitalOcean
- [ ] **Network Security:** VPC configuration and firewall rules
- [ ] **Scaling Policies:** Autoscaling configuration for App Platform

---

## License

Do NOT modify or remove this copyright and confidentiality notice.

**Copyright © Nikolai Alexander. All rights reserved.**

The code contained herein is CONFIDENTIAL to Nikolai Alexander. Portions
may also be trade secret. Any use, duplication, derivation, distribution or
disclosure of this code, for any reason, not expressly authorized in writing
by Nikolai Alexander is prohibited. All rights are expressly reserved by Nikolai Alexander.

---
