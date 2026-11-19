# Contributing

1. Open an issue describing the change or problem.
2. Create a branch from `main`.
3. Keep infrastructure changes small and document new variables and costs.
4. Run `terraform fmt -recursive deployment/terraform` and `terraform validate`.
5. Add or update tests and documentation.
6. Open a pull request describing the change, validation performed, and any AWS resources affected.

Never commit credentials, Terraform state, private keys, or real customer data.
