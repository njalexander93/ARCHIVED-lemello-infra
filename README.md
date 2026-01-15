# ChefAI Infrastructure

This repository contains the **Infrastructure as Code (IaC)** for ChefAI, managed using
**Terraform** and deployed on **AWS**.

It is responsible for provisioning and managing all shared cloud resources, including:

- Networking (VPC, subnets, security groups)
- PostgreSQL (AWS RDS)
- Redis
- IAM roles and policies
- Supporting AWS services required by ChefAI

This repository is environment-agnostic and is designed to support multiple environments
(e.g. dev, staging, production) through variable configuration.

---

## Requirements

- **Terraform 1.14.3**
- AWS CLI (configured with appropriate credentials)
- An AWS account
- git

---

## Terraform Version Policy

ChefAI Infrastructure is standardized on **Terraform 1.14.x**.

All contributors and automation must use Terraform `1.14.x` to ensure consistent provider
behavior, state handling, and plan output.

Verify your version:

```bash
terraform --version
````

Expected output:

```text
Terraform v1.14.3
```

---

## Tech Stack

* Terraform 1.14.x
* AWS (VPC, RDS, Redis, IAM, Secrets Manager)
* Remote Terraform state (recommended: S3 + DynamoDB)

---

## Repository Structure

```text
TBD
```

---

## Terraform State Management

Terraform state **must not** be stored locally or committed to Git.

This project is intended to use a **remote backend**, such as:

* Amazon S3 for state storage
* DynamoDB for state locking

Backend configuration should be defined in Terraform configuration and may vary by
environment.

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

Fill in the values appropriate for your environment **before** running Terraform.

⚠️ The `terraform.tfvars` file may contain sensitive values and **must never be committed**.

---

## Initial Setup

### Initialize Terraform

```bash
terraform init
```

This will:

* Download required providers
* Initialize the configured backend
* Generate or update `terraform.lock.hcl`

The `terraform.lock.hcl` file **must be committed** to ensure consistent provider versions
across all environments.

---

### Plan Changes

```bash
terraform plan
```

Review the execution plan carefully before applying any changes.

---

### Apply Changes

```bash
terraform apply
```

Only apply changes from a trusted environment with valid AWS credentials.

---

## Environments

Environment separation (e.g. dev, staging, production) is handled via:

* Separate `terraform.tfvars` files
* Separate remote state backends
* Optional directory-based layouts under `environments/`

The exact environment strategy can evolve as the project grows.

---

## Security Notes

* Terraform state files may contain sensitive data and must be stored securely
* No credentials, secrets, or `.tfvars` files are committed to Git
* AWS credentials should be provided via:

  * Environment variables
  * AWS named profiles
  * CI/CD role assumption
* Application secrets should be stored in AWS Secrets Manager

---

## Version Control Rules

* `terraform.lock.hcl` **must be committed**
* `.tfstate` files are **never committed**
* `.tfvars` files are **never committed**
* `.terraform/` directories are ignored

---

## License

Do NOT modify or remove this copyright and confidentiality notice.

**Copyright © Nikolai Alexander. All rights reserved.**

The code contained herein is CONFIDENTIAL to Nikolai Alexander. Portions
may also be trade secret. Any use, duplication, derivation, distribution or
disclosure of this code, for any reason, not expressly authorized in writing
by Nikolai Alexander is prohibited. All rights are expressly reserved by Nikolai Alexander.

---
