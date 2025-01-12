## Change Log

### [1.0.1] - 2025-01-
#### Added
- Added bages to readme file

### [1.0.0] - 2025-01-02
#### Added
- CloudFormation template to define:
  - OIDC provider.
  - IAM role with S3 full access and `sts:AssumeRoleWithWebIdentity` permissions.
  - Environment-specific tags for resources.
- GitHub Actions workflow with:
  - Environment (`Development`) for write operations.
  - Separation of read (`READ_ROLE`) and write (`DEV_DEPLOY_ROLE`) roles.

#### Changed
- Structured the GitHub Actions workflow for multiple environments (`Development`, `Production`).
- Differentiated ID token subject claims:
  - `repo:ORG_OR_USER_NAME/REPOSITORY:pull_request` for general access.
  - `repo:ORG_OR_USER_NAME/REPOSITORY:environment:Development` for specific environments.

#### Fixed
- Clarified role and environment usage in the GitHub Actions workflow.