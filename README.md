[![Deploy AWS S3 Bucket with OIDC authentication](https://github.com/superissy/github-actions-aws-oidc-deploy/actions/workflows/s3_bucket_oidc.yaml/badge.svg)](https://github.com/superissy/github-actions-aws-oidc-deploy/actions/workflows/s3_bucket_oidc.yaml)

# OIDC GitHub Action for Creating a Test S3 Bucket 
This README provides detailed instructions on using the OIDC (OpenID Connect) GitHub Action to create and manage a test Amazon Elastic S3 bucket.

## Prerequisites
Before using this GitHub Action, ensure you have the following:

- An AWS account with appropriate permissions to create and manage S3 bucket
- A GitHub repository where the workflow will be configured.
- Basic understanding of GitHub Actions



# Steps and how the OIDC works 
- Register Github as an Identity Provider (OIDC)
- Action requests to generate signed JWT 
- Issues a signed JWT
- Action sends JWT requested role ARN to AWS 
- Validate JWT, Verify that the token is allowed to assume the requested role, and Send a short-lived access token in exchange

# The Github Action pipeline does the following 
- Create an s3 bucket for you 
