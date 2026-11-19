# Security Notes

This repository is a learning reference. Perform a security review before production use.

## Network controls

- Keep EC2 and RDS in private subnets.
- Do not assign public IP addresses to application instances.
- Restrict ALB-to-EC2 and EC2-to-RDS access with security-group references.
- Database subnet route tables have no internet default route.
- Custom NACLs are stateless: include return ephemeral ports where required.

## Application controls

- Use PDO prepared statements for database queries.
- Escape output with `htmlspecialchars`.
- Validate all input server-side.
- Do not expose stack traces or database errors to clients.
- Store secrets in AWS Secrets Manager or Parameter Store SecureString in production.
- Rotate database credentials and use least-privilege DB users.

## AWS controls

- Use IAM roles, not static access keys on EC2.
- Use Systems Manager Session Manager instead of public SSH.
- Enable HTTPS with ACM.
- Keep S3 Block Public Access enabled and use CloudFront OAC.
- Review WAF managed-rule metrics and tune false positives before enabling blocking in production.
- Encrypt RDS storage and backups.
- Protect Terraform state and CI credentials.

Report vulnerabilities privately to the repository maintainer. Do not open public issues containing credentials or exploitable details.
