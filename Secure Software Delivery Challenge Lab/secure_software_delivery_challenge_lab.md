# Secure Software Delivery: Challenge Lab Solution

This project demonstrates my skills and expertise in securely deploying applications using Google Cloud services. It is based on the **"Secure Software Delivery: Challenge Lab"**, which is part of the **Secure Software Delivery course** by [Cloud Skills Boost](https://www.cloudskillsboost.google) (offered by Google). The project focuses on creating a secure CI/CD pipeline for containerized applications, emphasizing automated builds, vulnerability scanning, signing, and deployment.

---

## Table of Contents
- [Overview](#overview)
- [Objectives](#objectives)
- [Key Steps](#key-steps)
  - [1. Enable APIs and Set Up the Environment](#1-enable-apis-and-set-up-the-environment)
  - [2. Configure CI/CD Pipeline](#2-configure-cicd-pipeline)
  - [3. Set Up Binary Authorization](#3-set-up-binary-authorization)
  - [4. Fix Vulnerabilities and Redeploy](#4-fix-vulnerabilities-and-redeploy)
- [Outputs and Screenshots](#outputs-and-screenshots)
- [Conclusion](#conclusion)

---

## Overview
This project involved securely deploying a web application for Cymbal Bank. Since the application handles sensitive customer data, robust security measures were implemented. The pipeline incorporates:
- **Artifact Registry**: For storing Docker images.
- **Binary Authorization**: To enforce signed image deployment.
- **Cloud Build**: To automate the build, scan, and deployment process.

The solution adheres to the standards outlined in the **Secure Software Delivery course**, designed to build practical cloud security skills.

---

## Objectives
The main goals of the project were:
1. Enable necessary APIs and set up the Google Cloud environment.
2. Build and push containerized applications to Artifact Registry.
3. Integrate vulnerability scanning into the CI/CD pipeline.
4. Configure Binary Authorization for secure deployments.
5. Identify and fix vulnerabilities, then redeploy.

---

## Key Steps

### 1. Enable APIs and Set Up the Environment
```bash
gcloud services enable \
cloudkms.googleapis.com \
run.googleapis.com \
cloudbuild.googleapis.com \
container.googleapis.com \
containerregistry.googleapis.com \
artifactregistry.googleapis.com \
containerscanning.googleapis.com \
ondemandscanning.googleapis.com \
binaryauthorization.googleapis.com
