# Contributing to Lemello Infrastructure

This repository contains infrastructure-as-code for Lemello, managed via Terraform and deployed on DigitalOcean.

Contributions must follow the guidelines below to ensure consistency, security, and reliability.

---

## Pull Request Policy

All changes to this repository must be made through pull requests.

Direct commits to the main branch are not permitted.

---

## Before Contributing

Ensure you have the required tools installed and configured:

- Terraform 1.14.x
- DigitalOcean CLI (`doctl`) with valid credentials
- Docker (for local testing)
- git

Verify your Terraform version:

```bash
terraform --version
```

Expected output:

```text
Terraform v1.14.3
```

---

## Terraform Best Practices

All contributions must adhere to the following standards:

### Formatting

Run `terraform fmt` before committing:

```bash
terraform fmt -recursive
```

All Terraform files must be properly formatted according to Terraform's canonical style.

### Variables and Outputs

- Use descriptive variable names
- Provide clear descriptions for all variables
- Specify appropriate types for all variables
- Document all outputs with descriptions
- Never hardcode sensitive values

### Security

- No credentials, API tokens, or secrets in Terraform code
- Use variables for all sensitive configuration
- Ensure `.tfvars` files are excluded from version control
- Follow the principle of least privilege for all resources

### State Management

- Never commit `.tfstate` files
- Never commit `.tfvars` files
- Always commit `terraform.lock.hcl` to ensure provider consistency

---

## Pull Request Requirements

Before submitting a pull request:

1. Run `terraform fmt` to format all files
2. Run `terraform validate` to check configuration syntax
3. Run `terraform plan` to preview changes
4. Document the purpose and impact of your changes in the pull request description
5. Include relevant context about what resources are affected

---

## Change Review Process

All pull requests will be reviewed for:

- Correctness of Terraform syntax and logic
- Security implications of the proposed changes
- Cost impact of new or modified resources
- Alignment with project architecture and standards
- Proper documentation and variable usage

---

## Testing Infrastructure Changes

Before submitting a pull request:

- Review the Terraform plan output carefully
- Understand the impact of each resource change
- Consider the cost implications of your changes
- Verify that no secrets or credentials are included

Do not apply infrastructure changes without approval.

---

## Commit Message Guidelines

Use clear, descriptive commit messages that explain:

- What resources are being modified
- Why the change is necessary
- Any breaking changes or special considerations

Example:

```text
Add staging environment App Platform configuration

- Create separate app spec for staging deployment
- Configure 512MB instances for cost optimization
- Update variables to support environment-specific sizing
```

---

## Questions and Support

If you have questions about contributing to this repository:

- Review the README.md for architecture and setup details
- Consult the Terraform documentation for provider-specific questions
- Reach out to the repository maintainers for clarification

---

## License

All contributions are subject to the repository license. By submitting a pull request, you agree that your contributions will be licensed under the same terms.
