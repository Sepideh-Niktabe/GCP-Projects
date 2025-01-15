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
At the beginning of the lab, determine the **region,project Id and Project Number** provided in the lab instructions and enter it when prompted. Run the following commands to enable APIs and create the required repositories:

```bash
export PROJECT_ID=$(gcloud config get-value project)
export PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID \
--format='value(projectNumber)')
```
![My Image](/Secure%20Software%20Delivery%20Challenge%20Lab/images/initialization_1.jpg)


```bash
echo "Please set the below values correctly"
read -p "Enter the REGION (as provided in the lab instructions): " REGION
```
```bash
export REGION
```
![My Image](/Secure%20Software%20Delivery%20Challenge%20Lab/images/initialization_2.jpg)

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
```
![My Image](/Secure%20Software%20Delivery%20Challenge%20Lab/images/Task1_1.jpg)

<!-- ![alt text](![initialization](https://github.com/user-attachments/assets/490696fe-693b-4e0b-943e-305c1fb4403e)) -->

```bash
mkdir sample-app && cd sample-app
gcloud storage cp gs://spls/gsp521/* .
```
![My Image](/Secure%20Software%20Delivery%20Challenge%20Lab/images/Task1_2.jpg)


```bash
gcloud artifacts repositories create artifact-scanning-repo \
  --repository-format=docker \
  --location=$REGION \
  --description="Scanning repository"

gcloud artifacts repositories create artifact-prod-repo \
  --repository-format=docker \
  --location=$REGION \
  --description="Production repository"
```

![My Image](/Secure%20Software%20Delivery%20Challenge%20Lab/images/Task1_3.jpg)



### Task 2: Create a Cloud Build Pipeline

#### Lab Task
Build a Docker image for the application, push it to the Artifact Registry, and verify vulnerabilities.


#### Solution

```bash
gcloud auth configure-docker $REGION-docker.pkg.dev

gcloud projects add-iam-policy-binding ${PROJECT_ID} --member="serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com" --role="roles/iam.serviceAccountUser"
 
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member="serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com" --role="roles/ondemandscanning.admin"

```
![My Image](/Secure%20Software%20Delivery%20Challenge%20Lab/images/rule1.jpg)


![My Image](/Secure%20Software%20Delivery%20Challenge%20Lab/images/rule2.jpg)

```bash

cat > cloudbuild.yaml <<EOF
steps:

# Build Step. Replace the <image-name> placeholder with the correct value.
- id: "build"
  name: 'gcr.io/cloud-builders/docker'
  args: ['build', '-t', '${REGION}-docker.pkg.dev/${PROJECT_ID}/artifact-scanning-repo/sample-image', '.']
  waitFor: ['-']

# Push to Artifact Registry. Replace the <image-name> placeholder with the correct value.
- id: "push"
  name: 'gcr.io/cloud-builders/docker'
  args: ['push',  '${REGION}-docker.pkg.dev/${PROJECT_ID}/artifact-scanning-repo/sample-image']


## More steps will be added here in a later section

# Replace <image-name> placeholder with the value from the build step.
images:
  - ${REGION}-docker.pkg.dev/${PROJECT_ID}/artifact-scanning-repo/sample-image
EOF
```

Submit the build using:
```bash
gcloud builds submit
```

![My Image](/Secure%20Software%20Delivery%20Challenge%20Lab/images/Task1_4.jpg)


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
```

![My Image](/Secure%20Software%20Delivery%20Challenge%20Lab/images/Task1_5.jpg)


```bash

NOTE_ID=vulnerability_note
curl -vvv -X POST \
-H "Content-Type: application/json"  \
-H "Authorization: Bearer $(gcloud auth print-access-token)"  \
--data-binary @./vulnerability_note.json  \
"https://containeranalysis.googleapis.com/v1/projects/${PROJECT_ID}/notes/?noteId=${NOTE_ID}"

```

![My Image](/Secure%20Software%20Delivery%20Challenge%20Lab/images/Task1_6.jpg)

```bash
curl -vvv -H "Authorization: Bearer $(gcloud auth print-access-token)" \
"https://containeranalysis.googleapis.com/v1/projects/${PROJECT_ID}/notes/${NOTE_ID}"

```
![My Image](/Secure%20Software%20Delivery%20Challenge%20Lab/images/Task1_7.jpg)



2. Set up Binary Authorization:


```bash
ATTESTOR_ID=vulnerability-attestor
gcloud container binauthz attestors create vulnerability-attestor \
    --attestation-authority-note=$NOTE_ID \
    --attestation-authority-note-project=${PROJECT_ID}
```

```bash
PROJECT_NUMBER=$(gcloud projects describe "${PROJECT_ID}"  --format="value(projectNumber)")
 
BINAUTHZ_SA_EMAIL="service-${PROJECT_NUMBER}@gcp-sa-binaryauthorization.iam.gserviceaccount.com"
```

![My Image](/Secure%20Software%20Delivery%20Challenge%20Lab/images/Task1_8.jpg)

```bash

cat > ./iam_request.json << EOM
{
'resource': 'projects/${PROJECT_ID}/notes/${NOTE_ID}',
'policy': {
'bindings': [
 {
   'role': 'roles/containeranalysis.notes.occurrences.viewer',
   'members': [
     'serviceAccount:${BINAUTHZ_SA_EMAIL}'
   ]
 }
]
}
}
EOM
```
![My Image](/Secure%20Software%20Delivery%20Challenge%20Lab/images/Task1_9.jpg)
```bash
curl -X POST  \
-H "Content-Type: application/json" \
-H "Authorization: Bearer $(gcloud auth print-access-token)" \
--data-binary @./iam_request.json \
"https://containeranalysis.googleapis.com/v1/projects/${PROJECT_ID}/notes/${NOTE_ID}:setIamPolicy"
```
![My Image](/Secure%20Software%20Delivery%20Challenge%20Lab/images/Task1_10.jpg)

```bash
KEY_LOCATION=global
KEYRING=binauthz-keys
KEY_NAME=lab-key
KEY_VERSION=1
```
![My Image](/Secure%20Software%20Delivery%20Challenge%20Lab/images/Task1_11.jpg)

```bash
gcloud kms keyrings create "${KEYRING}" --location="${KEY_LOCATION}"
```
```bash
gcloud kms keys create "${KEY_NAME}" \
--keyring="${KEYRING}" --location="${KEY_LOCATION}" \
--purpose asymmetric-signing   \
--default-algorithm="ec-sign-p256-sha256"
```
```bash
gcloud beta container binauthz attestors public-keys add  \
--attestor="${ATTESTOR_ID}"  \
--keyversion-project="${PROJECT_ID}"  \
--keyversion-location="${KEY_LOCATION}" \
--keyversion-keyring="${KEYRING}" \
--keyversion-key="${KEY_NAME}" \
--keyversion="${KEY_VERSION}"
```
```bash
gcloud container binauthz policy export > my_policy.yaml


cat > my_policy.yaml << EOM
defaultAdmissionRule:
  enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
  evaluationMode: REQUIRE_ATTESTATION
  requireAttestationsBy:
    - projects/${PROJECT_ID}/attestors/vulnerability-attestor
globalPolicyEvaluationMode: ENABLE
name: projects/${PROJECT_ID}/policy
EOM
```
![My Image](/Secure%20Software%20Delivery%20Challenge%20Lab/images/Task1_12.jpg)
```bash
gcloud container binauthz policy import my_policy.yaml
```
![My Image](/Secure%20Software%20Delivery%20Challenge%20Lab/images/Task1_13.jpg)



### Task 4: Integrate Vulnerability Scanning

#### Lab Task
Enhance the pipeline with vulnerability scanning and enforce policy rules.

#### Solution
Update `cloudbuild.yaml` to include scanning and attestation:


```bash
gcloud projects add-iam-policy-binding ${PROJECT_ID} \
--member serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com \
--role roles/binaryauthorization.attestorsViewer

gcloud projects add-iam-policy-binding ${PROJECT_ID} \
--member serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com \
--role roles/cloudkms.signerVerifier

gcloud projects add-iam-policy-binding ${PROJECT_ID} --member serviceAccount:${PROJECT_NUMBER}-compute@developer.gserviceaccount.com --role roles/cloudkms.signerVerifier

gcloud projects add-iam-policy-binding ${PROJECT_ID} \
--member serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com \
--role roles/containeranalysis.notes.attacher


gcloud projects add-iam-policy-binding ${PROJECT_ID} --member="serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com" --role="roles/iam.serviceAccountUser"
 
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member="serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com" --role="roles/ondemandscanning.admin"
```
![My Image](/Secure%20Software%20Delivery%20Challenge%20Lab/images/Task1_14.jpg)


```bash
git clone https://github.com/GoogleCloudPlatform/cloud-builders-community.git
cd cloud-builders-community/binauthz-attestation
gcloud builds submit . --config cloudbuild.yaml
cd ../..
rm -rf cloud-builders-community
```

![My Image](/Secure%20Software%20Delivery%20Challenge%20Lab/images/Task1_15.jpg)


---

### Task 5: Fix Vulnerabilities and Redeploy

#### Lab Task
Identify and resolve vulnerabilities, then redeploy the pipeline.

#### Solution

1. Fill in the missing parts of the pipeline

```bash
cat <<EOF > cloudbuild.yaml
steps:

- id: "build"
  name: 'gcr.io/cloud-builders/docker'
  args: ['build', '-t', '${REGION}-docker.pkg.dev/${PROJECT_ID}/artifact-scanning-repo/sample-image:latest', '.']
  waitFor: ['-']

- id: "push"
  name: 'gcr.io/cloud-builders/docker'
  args: ['push',  '${REGION}-docker.pkg.dev/${PROJECT_ID}/artifact-scanning-repo/sample-image:latest']

- id: scan
  name: 'gcr.io/cloud-builders/gcloud'
  entrypoint: 'bash'
  args:
  - '-c'
  - |
      (gcloud artifacts docker images scan \\
      <command url> \\
      --location us \\
      --format="value(response.scan)") > /workspace/scan_id.txt

- id: severity check
  name: 'gcr.io/cloud-builders/gcloud'
  entrypoint: 'bash'
  args:
  - '-c'
  - |
      gcloud artifacts docker images list-vulnerabilities \$(cat /workspace/scan_id.txt) \\
      --format="value(vulnerability.effectiveSeverity)" | if grep -Fxq CRITICAL; \\
      then echo "Failed vulnerability check for CRITICAL level" && exit 1; else echo \\
      "No CRITICAL vulnerability found, congrats !" && exit 0; fi

- id: 'create-attestation'
  name: 'gcr.io/${PROJECT_ID}/binauthz-attestation:latest'
  args:
    - '--artifact-url'
    - '${REGION}-docker.pkg.dev/${PROJECT_ID}/artifact-scanning-repo/sample-image:latest'
    - '--attestor'
    - 'projects/${PROJECT_ID}/attestors/vulnerability-attestor'
    - '--keyversion'
    - 'projects/${PROJECT_ID}/locations/global/keyRings/binauthz-keys/cryptoKeys/lab-key/cryptoKeyVersions/1'

- id: "push-to-prod"
  name: 'gcr.io/cloud-builders/docker'
  args: 
    - 'tag' 
    - '${REGION}-docker.pkg.dev/${PROJECT_ID}/artifact-scanning-repo/sample-image:latest'
    - '${REGION}-docker.pkg.dev/${PROJECT_ID}/artifact-prod-repo/sample-image:latest'
- id: "push-to-prod-final"
  name: 'gcr.io/cloud-builders/docker'
  args: ['push', '${REGION}-docker.pkg.dev/${PROJECT_ID}/artifact-prod-repo/sample-image:latest']

- id: 'deploy-to-cloud-run'
  name: 'gcr.io/cloud-builders/gcloud'
  entrypoint: 'bash'
  args:
  - '-c'
  - |
    gcloud run deploy auth-service --image=${REGION}-docker.pkg.dev/${PROJECT_ID}/artifact-scanning-repo/sample-image:latest \
    --binary-authorization=default --region=$REGION --allow-unauthenticated

images:
  - ${REGION}-docker.pkg.dev/${PROJECT_ID}/artifact-scanning-repo/sample-image:latest
EOF
```
```bash
sed -i "s|<command url>|${REGION}-docker.pkg.dev/${PROJECT_ID}/artifact-scanning-repo/sample-image:latest|g" cloudbuild.yaml
```
```bash
gcloud builds submit
```
2. Update the `Dockerfile` to resolve vulnerabilities:

```bash
cat > ./Dockerfile << EOF
FROM python:3.8-alpine
 
# App
WORKDIR /app
COPY . ./
 
RUN pip3 install Flask==3.0.3
RUN pip3 install gunicorn==23.0.0
RUN pip3 install Werkzeug==3.0.4
 
CMD exec gunicorn --bind :\$PORT --workers 1 --threads 8 main:app
 
EOF
```

![My Image](/Secure%20Software%20Delivery%20Challenge%20Lab/images/Task1_17.jpg)

2. Redeploy the pipeline:

    ```bash
    gcloud builds submit
    ```
    ```bash
    gcloud beta run services add-iam-policy-binding --region=$REGION --member=allUsers --role=roles/run.invoker auth-service
    ```

---






## Conclusion

This project demonstrates the ability to:
- Implement secure CI/CD pipelines using Google Cloud.
- Detect and remediate vulnerabilities.
- Enforce deployment of trusted images with Binary Authorization.

The lab is part of the **Secure Software Delivery course** on [Cloud Skills Boost by Google](https://www.cloudskillsboost.google).

---

*Project By:* **Sepideh Niktabe**
