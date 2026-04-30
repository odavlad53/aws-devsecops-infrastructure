# AWS DevSecOps Infrastructure Platform — Tasks 1-3

Part of a 5-task progressive DevSecOps platform built on AWS from scratch.
Tasks 4-5 (ECS Fargate + EKS): https://github.com/odavlad53/terraform-cicd-project

## What This Builds

Secure AWS infrastructure provisioned entirely through Terraform, with a
4-stage GitHub Actions CI/CD pipeline that validates, security-scans, and
deploys infrastructure automatically — stopping immediately if any security
issue is found.

## Task 1 — Terraform IaC Foundations

- S3 bucket: AES256 SSE encryption, versioning, public access block
- EC2 instance: t2.micro with detailed CloudWatch monitoring
- State security: .gitignore excludes .tfstate, .tfvars, .terraform/
  — secrets and infrastructure layout never enter Git

## Task 2 — GitHub Actions CI/CD Pipeline

4-stage pipeline with security gates:

Stage 1: terraform fmt + validate — syntax and formatting check
Stage 2: tfsec security scan — HARD FAIL on any critical finding
Stage 3: terraform plan — posts diff as PR comment for human review
Stage 4: terraform apply — main branch only, requires manual approval

AWS credentials stored exclusively in GitHub Secrets. Never in code or logs.

## Task 3 — Security Hardening and GitOps

IAM least privilege:
- EC2 role: S3 read-only + SSM Session Manager (no SSH, no port 22)
- CI/CD deploy role: scoped to deployment actions only, cannot create
  IAM users or escalate privileges

Why SSM instead of SSH:
No open ports. No keys to manage. Every session automatically logged
in CloudTrail. Zero inbound attack surface.

Encryption at rest:
- S3: AES256 server-side encryption
- EBS: root volume encryption enabled

Automated security scanning (daily 02:00 UTC):
- TruffleHog: scans entire Git history for secrets and credentials
- Trivy: IaC misconfiguration scanning, SARIF output to GitHub Security tab
- Checkov: CIS Benchmark compliance validation, hard fail on violations

Pre-commit hooks:
- Blocks AWS credentials before they enter Git
- Blocks secrets and tokens before they enter Git
- Enforces terraform fmt and validate on every commit

Branch protection:
- Direct pushes to main blocked for all users
- PRs require all CI checks to pass + 1 reviewer approval

## Repository Structure

terraform-infrastructure/
├── .github/workflows/
│   ├── terraform.yml           # 4-stage CI/CD pipeline
│   └── security-schedule.yml   # Daily TruffleHog/Trivy/Checkov
├── terraform-infrastructure/
│   ├── main.tf                 # S3 and EC2 resources
│   ├── iam.tf                  # Least-privilege IAM roles
│   ├── backend.tf              # Remote state configuration
│   ├── variables.tf            # Input variables
│   ├── outputs.tf              # Output values
│   └── provider.tf             # AWS provider pinned version
├── .gitignore                  # Excludes tfstate, tfvars, .terraform/
└── GITOPS_WORKFLOW.md          # Branch strategy documentation

## Security Principles Demonstrated

Shift-left security  — tfsec at Stage 2, before any infrastructure exists
Least privilege      — separate scoped IAM roles per component
No secrets in code   — GitHub Secrets + pre-commit hooks + TruffleHog
Defence in depth     — pre-commit hooks + CI scanning, two independent layers
Immutable audit trail — SSM sessions logged in CloudTrail automatically
GitOps               — all changes via PR, no direct pushes to main

## Tech Stack

IaC: Terraform
CI/CD: GitHub Actions
Security Scanning: tfsec, Checkov, Trivy, TruffleHog
AWS: S3, EC2, IAM, CloudTrail, CloudWatch, SSM

## Related

Tasks 4-5 (ECS Fargate + EKS + Helm):
https://github.com/odavlad53/terraform-cicd-project

Olga Davladova — Cloud Security Engineer | DevSecOps | AWS
https://linkedin.com/in/olga-davladova
