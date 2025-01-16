# Networking Fundamentals: Lab and Solution

This document outlines the **"Networking Fundamentals on Google Cloud"** lab, including the tasks and corresponding solutions for configuring VPC networks, web servers, and load balancers.

---

## Table of Contents
- [Overview](#overview)
- [Lab Scenario](#lab-scenario)
- [Tasks and Solutions](#tasks-and-solutions)
  - [Task 1: Set Up the Environment and Create Web Servers](#task-1-set-up-the-environment-and-create-web-servers)
  - [Task 2: Configure Firewall Rules and Load Balancers](#task-2-configure-firewall-rules-and-load-balancers)
  - [Task 3: Create an HTTP Load Balancer and Test Traffic](#task-3-create-an-http-load-balancer-and-test-traffic)
- [Conclusion](#conclusion)

---

## Overview

This lab focuses on setting up a Virtual Private Cloud (VPC) network, configuring web servers, and deploying load balancers to distribute traffic across multiple instances.

---

## Lab Scenario

In this lab, you will:
1. Set up the required Google Cloud environment.
2. Create virtual machine (VM) instances to act as web servers.
3. Configure load balancers to distribute traffic.
4. Set up firewall rules to allow HTTP traffic.

---

## Tasks and Solutions

### Task 1: Set Up the Environment and Create Web Servers

#### Step 1: Set Project Variables

```bash
# Configure gcloud settings
gcloud config set compute/region $REGION
gcloud config set compute/zone $ZONE

```
![My Image](/Networking%20Fundamentals%20on%20Google%20Cloud%20Challenge%20Lab/images/Task1_1.jpg)
#### Step 2: Create Web Servers

```bash
gcloud compute instances create web1 \
    --zone=$ZONE \
    --machine-type=e2-small \
    --tags=network-lb-tag \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
apt-get update
apt-get install apache2 -y
service apache2 restart
echo "<h3>Web Server: web1</h3>" | tee /var/www/html/index.html'

gcloud compute instances create web2 \
    --zone=$ZONE \
    --machine-type=e2-small \
    --tags=network-lb-tag \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
apt-get update
apt-get install apache2 -y
service apache2 restart
echo "<h3>Web Server: web2</h3>" | tee /var/www/html/index.html'

gcloud compute instances create web3 \
    --zone=$ZONE \
    --machine-type=e2-small \
    --tags=network-lb-tag \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
apt-get update
apt-get install apache2 -y
service apache2 restart
echo "<h3>Web Server: web3</h3>" | tee /var/www/html/index.html'
```
![My Image](/Networking%20Fundamentals%20on%20Google%20Cloud%20Challenge%20Lab/images/Task1_2.jpg)

![My Image](/Networking%20Fundamentals%20on%20Google%20Cloud%20Challenge%20Lab/images/Task1_2_2.jpg)
---

### Task 2: Configure Firewall Rules and Load Balancers

#### Step 1: Create Firewall Rules

```bash
gcloud compute firewall-rules create www-firewall-network-lb \
    --allow tcp:80 \
    --target-tags network-lb-tag
```

#### Step 2: Set Up Network Load Balancer

```bash
gcloud compute addresses create network-lb-ip-1 --region=$REGION

gcloud compute http-health-checks create basic-check

gcloud compute target-pools create www-pool \
    --region=$REGION \
    --http-health-check basic-check

gcloud compute target-pools add-instances www-pool \
    --instances web1,web2,web3 --zone=$ZONE

gcloud compute forwarding-rules create www-rule \
    --region=$REGION \
    --ports 80 \
    --address network-lb-ip-1 \
    --target-pool www-pool

# Retrieve the forwarding rule IP
IPADDRESS=$(gcloud compute forwarding-rules describe www-rule --region=$REGION --format="json" | jq -r .IPAddress)
echo "Access your load balancer at: $IPADDRESS"
```

---
![My Image](/Networking%20Fundamentals%20on%20Google%20Cloud%20Challenge%20Lab/images/Task2_1.jpg)

### Task 3: Create an HTTP Load Balancer and Test Traffic

#### Step 1: Create an Instance Template and Managed Instance Group

```bash
gcloud compute instance-templates create lb-backend-template \
    --region=$REGION \
    --network=default \
    --subnet=default \
    --tags=allow-health-check \
    --machine-type=e2-medium \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
apt-get update
apt-get install apache2 -y
echo "Page served from: $(hostname)" | tee /var/www/html/index.html
systemctl restart apache2'


gcloud compute instance-groups managed create lb-backend-group \
    --template=lb-backend-template --size=2 --zone=$ZONE
```

![My Image](/Networking%20Fundamentals%20on%20Google%20Cloud%20Challenge%20Lab/images/Task3-1-.jpg)
#### Step 2: Set Up Firewall Rules and Global Load Balancer

```bash
gcloud compute firewall-rules create fw-allow-health-check \
    --network=default \
    --action=allow \
    --direction=ingress \
    --source-ranges=130.211.0.0/22,35.191.0.0/16 \
    --target-tags=allow-health-check \
    --rules=tcp:80

# Reserve a global static IP address
gcloud compute addresses create lb-ipv4-1 --ip-version=IPV4 --global

# Create health check
gcloud compute health-checks create http http-basic-check --port 80

# Create backend service
gcloud compute backend-services create web-backend-service \
    --protocol=HTTP \
    --port-name=http \
    --health-checks=http-basic-check \
    --global

gcloud compute backend-services add-backend web-backend-service \
    --instance-group=lb-backend-group \
    --instance-group-zone=$ZONE \
    --global

# Create URL map and HTTP proxy
gcloud compute url-maps create web-map-http --default-service web-backend-service
gcloud compute target-http-proxies create http-lb-proxy --url-map web-map-http

gcloud compute forwarding-rules create http-content-rule \
    --address=lb-ipv4-1 \
    --global \
    --target-http-proxy=http-lb-proxy \
    --ports=80
```

---
![My Image](/Networking%20Fundamentals%20on%20Google%20Cloud%20Challenge%20Lab/images/Task3_2.jpg)
![My Image](/Networking%20Fundamentals%20on%20Google%20Cloud%20Challenge%20Lab/images/Task3_3.jpg)
![My Image](/Networking%20Fundamentals%20on%20Google%20Cloud%20Challenge%20Lab/images/Task3_4.jpg)
![My Image](/Networking%20Fundamentals%20on%20Google%20Cloud%20Challenge%20Lab/images/Task3_5.jpg)
![My Image](/Networking%20Fundamentals%20on%20Google%20Cloud%20Challenge%20Lab/images/Task3_6.jpg)
![My Image](/Networking%20Fundamentals%20on%20Google%20Cloud%20Challenge%20Lab/images/Task3-7.jpg)
## Conclusion

This lab demonstrated how to:
- Set up a VPC network on Google Cloud.
- Configure VM instances as web servers.
- Deploy load balancers to distribute traffic.
- Implement firewall rules for secure access.

By completing this lab, you have gained foundational knowledge of networking on Google Cloud.

---

*Prepared by:* **Sepideh Niktabe**
