# Security Policy

This document outlines the security policy for the Lemello Infrastructure repository.

---

## Reporting Security Vulnerabilities

If you discover a security vulnerability in this repository, please report it privately rather than opening a public issue.

Security issues include, but are not limited to:

- Exposed credentials or API tokens in Terraform code
- Misconfigured cloud resources that could lead to unauthorized access
- Infrastructure configurations that violate security best practices
- Terraform state management vulnerabilities
- DigitalOcean service misconfigurations

---

## Reporting Process

To report a security vulnerability:

1. Do not open a public issue or pull request
2. Contact the repository maintainers directly through private channels
3. Provide a detailed description of the vulnerability, including:
   - The affected resource or configuration file
   - Steps to reproduce the issue
   - Potential impact of the vulnerability
   - Any suggested remediation steps

---

## Response Timeline

Upon receiving a security report:

- Initial acknowledgment within 48 hours
- Assessment of the issue within 7 days
- Resolution and disclosure timeline determined based on severity

---

## Security Best Practices

When contributing to this repository:

- Never commit credentials, API tokens, or other sensitive values
- Use Terraform variables for all sensitive configuration
- Ensure `.tfvars` files are excluded from version control
- Review Terraform plan output before applying changes
- Follow the principle of least privilege when configuring resource permissions
- Enable appropriate network security controls for all resources
- Regularly rotate credentials and API tokens

---

## Scope

This security policy applies to:

- All Terraform configuration files
- App Platform specifications
- Database configurations
- Container registry configurations
- Any scripts or automation related to infrastructure provisioning
- Documentation that references security-sensitive information

---

## Out of Scope

Application-level security issues should be reported to the appropriate application repository:

- Backend security issues: lemello-app/backend
- Webapp security issues: lemello-app/webapp
