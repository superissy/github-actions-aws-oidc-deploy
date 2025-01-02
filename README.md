[![Deploy AWS S3 Bucket with OIDC authentication](https://github.com/superissy/github-actions-aws-oidc-deploy/actions/workflows/s3_bucket_oidc.yaml/badge.svg)](https://github.com/superissy/github-actions-aws-oidc-deploy/actions/workflows/s3_bucket_oidc.yaml)

# Deploy AWS S3 Bucket with OIDC Authentication

This repository contains a GitHub Actions workflow to automate the creation and configuration of an AWS S3 bucket using OpenID Connect (OIDC) authentication. The workflow ensures secure and efficient bucket deployment, including best practices for versioning and encryption.

---

## Workflow Overview

The GitHub Actions workflow is defined in `.github/workflows/deploy-s3-bucket.yaml`. It performs the following actions:

1. **Authentication with AWS**:
   - Uses GitHub's OIDC provider to authenticate with AWS securely.
   - Assumes an IAM role to execute bucket operations.

2. **Bucket Management**:
   - Checks if the specified S3 bucket already exists.
   - Creates a new bucket if necessary, with the bucket name dynamically generated using the GitHub username.
   - Configures the bucket with:
     - **Versioning** for better object management and recovery.
     - **Server-side encryption** (AES256) for data security.
     - **Public access block** to ensure no unauthorized access.

3. **Environment Variables**:
   - `AWS_REGION`: AWS region for the bucket (default: `us-east-1`).
   - `AWS_ROLE_TO_ASSUME`: The IAM role ARN to assume via OIDC.
   - `BUCKET_NAME`: Unique bucket name combining the GitHub username and a suffix.

---

## Prerequisites

Before using this workflow, ensure the following:

1. **AWS IAM Role**:
   - Create an IAM role with permissions to manage S3 buckets.
   - Allow the IAM role to be accessed via GitHub's OIDC provider.

2. **GitHub Repository Permissions**:
   - Grant the workflow `id-token: write` and `contents: read` permissions in the `permissions` section of the YAML file.

3. **AWS CLI**:
   - The workflow relies on AWS CLI commands for bucket operations.
4. **Basic understanding of GitHub Actions**

---

## Workflow YAML File

Below is the workflow file used to deploy and configure the S3 bucket:

```yaml
name: Deploy AWS S3 Bucket with OIDC authentication

on:
  push

env:
  AWS_REGION: "us-east-1"
  AWS_ROLE_TO_ASSUME: "arn:aws:iam::129224467924:role/github-actions-aws-oidc-deploy-role"
  BUCKET_NAME: "${{ github.actor }}-oidc-test-bucket"

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ env.AWS_ROLE_TO_ASSUME }}
          role-session-name: s3-deploy-action-session
          aws-region: ${{ env.AWS_REGION }}

      - name: Check if bucket exists
        id: check_bucket
        run: |
          if aws s3api head-bucket --bucket ${{ env.BUCKET_NAME }} 2>/dev/null; then
            echo "Bucket ${{ env.BUCKET_NAME }} already exists"
            echo "bucket_exists=true" >> $GITHUB_OUTPUT
          else
            echo "Bucket ${{ env.BUCKET_NAME }} does not exist"
            echo "bucket_exists=false" >> $GITHUB_OUTPUT
          fi

      - name: Create S3 bucket
        if: steps.check_bucket.outputs.bucket_exists == 'false'
        run: |
          aws s3api create-bucket \
            --bucket ${{ env.BUCKET_NAME }} \
            --region ${{ env.AWS_REGION }} \
            $(if [ "${{ env.AWS_REGION }}" != "us-east-1" ]; then echo "--create-bucket-configuration LocationConstraint=${{ env.AWS_REGION }}"; fi)

          aws s3api put-bucket-versioning \
            --bucket ${{ env.BUCKET_NAME }} \
            --versioning-configuration Status=Enabled

          aws s3api put-bucket-encryption \
            --bucket ${{ env.BUCKET_NAME }} \
            --server-side-encryption-configuration '{
              "Rules": [
                {
                  "ApplyServerSideEncryptionByDefault": {
                    "SSEAlgorithm": "AES256"
                  }
                }
              ]
            }'

      - name: Configure bucket public access
        if: steps.check_bucket.outputs.bucket_exists == 'false'
        run: |
          aws s3api put-public-access-block \
            --bucket ${{ env.BUCKET_NAME }} \
            --public-access-block-configuration \
              '{"BlockPublicAcls":true,"IgnorePublicAcls":true,"BlockPublicPolicy":true,"RestrictPublicBuckets":true}'
