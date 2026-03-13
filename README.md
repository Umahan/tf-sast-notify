


# Terraform SAST Notify CI/CD Pipeline
![CI/CD](https://img.shields.io/badge/GitLab-CI-Pipeline-blue)
![Terraform](https://img.shields.io/badge/Terraform-1.8+-844FBA?logo=terraform)
![Checov](https://img.shields.io/badge/Checkov-Security%2520Scanning-brightgreen)
![TFlint](https://img.shields.io/badge/TFLint-Linter-orange)
![Yandex](https://img.shields.io/badge/Yandex.Cloud-Provider-blue?logo=yandex)

A production‑ready GitLab CI/CD pipeline for Terraform projects with integrated security linting, validation, planning, and deployment, complete with Telegram notifications and Yandex Cloud provider mirror support.

# Overview
This repository contains a GitLab CI/CD pipeline designed for Terraform‑based infrastructure management. It ensures code quality and security through automated linting, runs a plan only on merge requests targeting the default branch, and allows manual apply/destroy actions. Notifications about pipeline and job status are sent directly to a Telegram chat.

The pipeline is optimized for Yandex Cloud (using their Terraform provider mirror) but can be adapted to any provider.

# Features

✅ Multi‑stage pipeline – lint → validate → plan → apply / destroy
🔒 Security scanning with Checkov
🔍 Best‑practice linting with TFLint
📦 Terraform plan only on merge requests to main
📊 JSON plan output – easily integrate with merge request widgets
🛑 Manual approval for apply and destroy stages
🤖 Telegram notifications – instant feedback on job status
☁️ Yandex Cloud provider mirror – bypass HashiCorp registry restrictions

# Pipeline Stages & Jobs
```lint:checkov```
Image: bridgecrew/checkov

Purpose: Scans Terraform code for security misconfigurations and compliance violations.

Command: checkov -d .

Fails on: Any high‑severity issue found.

```lint:tflint```
- Image: ghcr.io/terraform-linters/tflint
- Purpose: Checks for provider‑specific best practices and potential errors.
- Command: tflint
- Fails on: Any linting error or warning.

```validate```
- Image: hashicorp/terraform:1.8
- Purpose: Runs terraform validate to ensure configuration syntax is correct.
- Dependencies: None (runs after lint stages automatically due to stage ordering).

```plan```
- Triggers: Only for merge request events where the target branch is main.
- Purpose: Creates a Terraform execution plan and exports it as a JSON artifact.
- Artifact: planfile (binary) and plan.json (report) for GitLab MR integration.

Note: The convert_report alias transforms the plan into a summary of create/update/delete counts.

```apply```
- When: Manual (when: manual)
- Purpose: Applies the previously generated plan. Requires a successful plan job.
- Dependencies: plan (uses its planfile artifact).

```destroy```
- When: Manual (when: manual)
- Purpose: Destroys all resources managed by Terraform (use with caution!).
- Command: terraform destroy --auto-approve


# Telegram Notifications
After every job, the `after_script` sends a message to the configured Telegram chat containing:
- Pipeline name and ID (linked to the pipeline URL)
- Job name and status (linked to the job URL)
- Status is formatted in bold (success, failed, etc.)

**Example message:**

> Pipeline: main/123, Job: plan => success

The notification script automatically installs `curl` if not present (using either `apk` or `apt` depending on the base image).
