# Set Up a Google Cloud Network: Challenge Lab Solution and Explanation

This project is based on the **"Set Up a Google Cloud Network: Challenge Lab"**, part of the **Google Cloud Networking course** by [Cloud Skills Boost](https://www.cloudskillsboost.google). Below are the tasks and solutions.

---

## Table of Contents
- [Overview](#overview)
- [Challenge Scenario](#challenge-scenario)
- [Tasks and Solutions](#tasks-and-solutions)
  - [Task 1: Create Networks](#task-1-create-networks)
  - [Task 2: Add Firewall Rules](#task-2-add-firewall-rules)
  - [Task 3: Add VMs to Your Network](#task-3-add-vms-to-your-network)
  - [Task 4: Test Connectivity Between VMs](#task-4-test-connectivity-between-vms)
- [Conclusion](#conclusion)

---

## Overview

This lab involves creating a Virtual Private Cloud (VPC) network in Google Cloud Platform (GCP) and ensuring proper connectivity between virtual machines (VMs) in different subnets. It demonstrates foundational networking concepts including subnetting, firewall rules, and inter-region communication.

---

## Challenge Scenario

You are tasked with setting up a Virtual Private Cloud (VPC) network in Google Cloud Platform (GCP) and ensuring proper connectivity between virtual machines (VMs) in different subnets. This involves:

1. Creating a VPC network with two subnets.
2. Configuring firewall rules for secure communication.
3. Launching two VMs in each subnet.
4. Testing connectivity between the VMs.

---

## Tasks and Solutions

### Task 1: Create Networks

#### Lab Task
Create a VPC network named `$VPC_NAME` with two subnets `$SUBNET_A` and `$SUBNET_B` using regional dynamic routing.

#### Solution


### Create VPC Network
![My Image](/Set%20Up%20a%20Google%20Cloud%20Network%20Challenge%20Lab/images/Task1_1.jpg)

### Create Subnets

![My Image](/Set%20Up%20a%20Google%20Cloud%20Network%20Challenge%20Lab/images/Task1_2.jpg)

![My Image](/Set%20Up%20a%20Google%20Cloud%20Network%20Challenge%20Lab/images/Task1_3.jpg)

### Task 2: Add Firewall Rules

#### Lab Task
Add firewall rules to allow ingress traffic for SSH, RDP, and ICMP protocols.

#### Solution
![My Image](/Set%20Up%20a%20Google%20Cloud%20Network%20Challenge%20Lab/images/Task2_1.jpg)

![My Image](/Set%20Up%20a%20Google%20Cloud%20Network%20Challenge%20Lab/images/Task2_2.jpg)

![My Image](/Set%20Up%20a%20Google%20Cloud%20Network%20Challenge%20Lab/images/Task2_3.jpg)

---

### Task 3: Add VMs to Your Network

#### Lab Task
Launch two VMs `$VM_1` and `$VM_2` in `$ZONE_1` and `$ZONE_2`, respectively.

#### Solution

### Create VM 1
![My Image](/Set%20Up%20a%20Google%20Cloud%20Network%20Challenge%20Lab/images/Task3_1.jpg)


![My Image](/Set%20Up%20a%20Google%20Cloud%20Network%20Challenge%20Lab/images/Task3_2.jpg)

### Create VM 2

![My Image](/Set%20Up%20a%20Google%20Cloud%20Network%20Challenge%20Lab/images/Task3_3.jpg)


![My Image](/Set%20Up%20a%20Google%20Cloud%20Network%20Challenge%20Lab/images/Task3_4.jpg)

![My Image](/Set%20Up%20a%20Google%20Cloud%20Network%20Challenge%20Lab/images/Task3_5.jpg)
---

### Test Connectivity Between VMs

#### Lab Task
Verify that the VMs can successfully communicate with each other and measure the latency between instances across all regions.

#### Solution

![My Image](/Set%20Up%20a%20Google%20Cloud%20Network%20Challenge%20Lab/images/Task3_6.jpg)


![My Image](/Set%20Up%20a%20Google%20Cloud%20Network%20Challenge%20Lab/images/Task3_7.jpg)

---

## Conclusion

This lab demonstrates the creation of a robust and secure network setup in Google Cloud. By following these steps, you can set up subnetting, firewall rules, and ensure seamless communication between resources across regions.

---

*Project By:* **Sepideh Niktabe**
