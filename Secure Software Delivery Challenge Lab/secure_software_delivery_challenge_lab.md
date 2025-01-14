# Secure Software Delivery: Challenge Lab Solution and Explanation

This project is based on the **"Secure Software Delivery: Challenge Lab"**, which is part of the **Secure Software Delivery course** by [Cloud Skills Boost](https://www.cloudskillsboost.google), offered by Google. Below are the tasks from the lab alongside their corresponding solutions and outputs.

---

## Table of Contents
- [Overview](#overview)
- [Challenge Scenario](#challenge-scenario)
- [Tasks and Solutions](#tasks-and-solutions)
  - [Task 1: Enable APIs and Set Up the Environment](#task-1-enable-apis-and-set-up-the-environment)
  - [Task 2: Create a Cloud Build Pipeline](#task-2-create-a-cloud-build-pipeline)
  - [Task 3: Set Up Binary Authorization](#task-3-set-up-binary-authorization)
  - [Task 4: Integrate Vulnerability Scanning](#task-4-integrate-vulnerability-scanning)
  - [Task 5: Fix Vulnerabilities and Redeploy](#task-5-fix-vulnerabilities-and-redeploy)
- [Conclusion](#conclusion)

---

## Overview

This lab focuses on securely deploying a web application for **Cymbal Bank**, which handles sensitive customer data. The goal is to implement a secure, automated CI/CD pipeline that builds, scans, signs, and deploys containerized applications while ensuring compliance with strict security standards. Key tools include:  

- **Artifact Registry**: For storing Docker images.  
- **Cloud Build**: For building and scanning container images.  
- **Binary Authorization**: To enforce strict image deployment policies.
---

## Challenge Scenario


As software engineers at **Cymbal** Bank, we are responsible for deploying a secure web application that handles sensitive customer data. Our objectives are to:
1. Build and push the application to Artifact Registry.
2. Scan the application for vulnerabilities.
3. Use Binary Authorization to enforce signed image deployment.
4. Fix vulnerabilities and redeploy a secure application to **Cloud Run**.

---

## Tasks and Solutions

### Task 1: Enable APIs and Set Up the Environment

#### Lab Task
Enable the required Google Cloud APIs and create Artifact Registry repositories for storing Docker images.

#### Solution
At the beginning of the lab, determine the **region** provided in the lab instructions and enter it when prompted. Run the following commands to enable APIs and create the required repositories:

```bash
echo "Please set the below values correctly"
read -p "Enter the REGION (as provided in the lab instructions): " REGION
```
```bash
export REGION
```
Run the following commands to enable APIs and create the required repositories:
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

![alt text]()
mkdir sample-app && cd sample-app
gcloud storage cp gs://spls/gsp521/* .

gcloud artifacts repositories create artifact-scanning-repo \
  --repository-format=docker \
  --location=$REGION \
  --description="Scanning repository"

gcloud artifacts repositories create artifact-prod-repo \
  --repository-format=docker \
  --location=$REGION \
  --description="Production repository"
```

**Output**:  
Attach a screenshot or paste the output from enabling APIs and creating repositories.  
![Enable APIs Output](path/to/enable-apis-output.png)

---

### Task 2: Create a Cloud Build Pipeline

#### Lab Task
Build a Docker image for the application, push it to the Artifact Registry, and verify vulnerabilities.

#### Solution
Here’s the `cloudbuild.yaml` configuration for building and pushing the image:
```yaml
steps:
  - id: "build"
    name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', '${REGION}-docker.pkg.dev/${PROJECT_ID}/artifact-scanning-repo/sample-image', '.']
  - id: "push"
    name: 'gcr.io/cloud-builders/docker'
    args: ['push', '${REGION}-docker.pkg.dev/${PROJECT_ID}/artifact-scanning-repo/sample-image']
images:
  - ${REGION}-docker.pkg.dev/${PROJECT_ID}/artifact-scanning-repo/sample-image
```
Submit the build using:
```bash
gcloud builds submit
```

**Output**:  
Attach the Cloud Build logs showing a successful build and vulnerabilities found during scanning.  
![Cloud Build Logs](path/to/cloud-build-logs.png)

---

### Task 3: Set Up Binary Authorization

#### Lab Task
Configure Binary Authorization to enforce signed image deployment.

#### Solution
1. Create an attestor and configure its note:
    ```bash
    cat > ./vulnerability_note.json << EOM
    {
      "attestation": {
        "hint": {
          "human_readable_name": "Container Vulnerabilities attestation authority"
        }
      }
    }
    EOM

    NOTE_ID=vulnerability_note
    curl -X POST -H "Content-Type: application/json" --data-binary @./vulnerability_note.json \
    "https://containeranalysis.googleapis.com/v1/projects/${PROJECT_ID}/notes/?noteId=${NOTE_ID}"
    ```

2. Set up Binary Authorization:
    ```bash
    gcloud container binauthz attestors create vulnerability-attestor \
      --attestation-authority-note=$NOTE_ID \
      --attestation-authority-note-project=${PROJECT_ID}
    ```

**Screenshot**:  
Include a screenshot of the Binary Authorization policy or configuration.  
![Binary Authorization Policy](path/to/binary-auth-policy.png)

---

### Task 4: Integrate Vulnerability Scanning

#### Lab Task
Enhance the pipeline with vulnerability scanning and enforce policy rules.

#### Solution
Update `cloudbuild.yaml` to include scanning and attestation:
```yaml
steps:
  - id: scan
    name: 'gcr.io/cloud-builders/gcloud'
    args:
      - 'artifacts docker images scan ${REGION}-docker.pkg.dev/${PROJECT_ID}/artifact-scanning-repo/sample-image --location=$REGION'
  - id: 'create-attestation'
    name: 'gcr.io/${PROJECT_ID}/binauthz-attestation:latest'
    args:
      - '--artifact-url'
      - '${REGION}-docker.pkg.dev/${PROJECT_ID}/artifact-scanning-repo/sample-image'
      - '--attestor'
      - 'projects/${PROJECT_ID}/attestors/vulnerability-attestor'
```

**Output**:  
Include logs from the scanning and attestation process.  
![Vulnerability Scanning Logs](path/to/vulnerability-scan.png)

---

### Task 5: Fix Vulnerabilities and Redeploy

#### Lab Task
Identify and resolve vulnerabilities, then redeploy the pipeline.

#### Solution
1. Update the `Dockerfile` to resolve vulnerabilities:
    ```Dockerfile
    FROM python:3.8-alpine

    WORKDIR /app
    COPY . ./
    RUN pip3 install Flask==3.0.3 gunicorn==23.0.0 Werkzeug==3.0.4

    CMD exec gunicorn --bind :$PORT --workers 1 --threads 8 main:app
    ```

2. Redeploy the pipeline:
    ```bash
    gcloud builds submit
    ```

**Screenshot**:  
Attach a screenshot of the successful build and deployment to Cloud Run.  
![Redeployment Success](path/to/redeployment-success.png)

---

## Conclusion

This project demonstrates the ability to:
- Implement secure CI/CD pipelines using Google Cloud.
- Detect and remediate vulnerabilities.
- Enforce deployment of trusted images with Binary Authorization.

The lab is part of the **Secure Software Delivery course** on [Cloud Skills Boost by Google](https://www.cloudskillsboost.google).

---

*Project By:* **Sepideh Niktabe**
