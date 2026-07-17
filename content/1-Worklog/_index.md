---
title: "Worklog"
date: 2026-05-12
weight: 1
chapter: true
pre: "<b>1. </b>"
---

{{% notice info %}}
**Introduction:** This page summarizes my entire 12-week internship process at the **Workforce Bootcamp - First Cloud AI Journey (FCAJ)** program. The roadmap is built starting from AWS fundamentals, networking, compute, storage, security, monitoring to an in-depth workshop on **private/hybrid access to Amazon S3**, then completed with documentation, knowledge sharing, and an internship report.
{{% /notice %}}

### 12-Week Roadmap Overview

Throughout the internship, I not only learned each AWS service individually but also step-by-step connected them into complete architectures. This roadmap helped me better understand how components like **IAM**, **VPC**, **EC2**, **EBS**, **S3**, **EFS**, **CloudWatch**, **CloudTrail**, **Route 53**, and **VPC Endpoints** coordinate with each other in practice.

| Week | Main Topic | Task Summary |
| :--- | :--- | :--- |
| **Week 1** | [Kick-off, account security, and environment setup](1.1-week1/) | Getting started with AWS, securing account with MFA and IAM, installing AWS CLI, launching the first EC2 instance. |
| **Week 2** | [Networking with VPC, Subnet, and Route Table](1.2-week2/) | Designing custom VPC, dividing public/private subnets, configuring IGW, NAT Gateway, Security Groups, and NACL. |
| **Week 3** | [Compute with EC2, Linux, and EBS](1.3-week3/) | Configuring EC2 instance, installing Nginx, attaching and expanding EBS volume, creating AMI for reuse. |
| **Week 4** | [Storage with S3 and EFS](1.4-week4/) | Creating S3 bucket, enabling versioning and lifecycle rules, hosting static website, and mounting EFS to EC2. |
| **Week 5** | [IAM, Roles, and Security Governance](1.5-week5/) | Designing least-privilege access, attaching IAM Role to EC2, testing S3 access without static credentials. |
| **Week 6** | [CloudWatch, CloudTrail, and System Observability](1.6-week6/) | Setting up metrics, alarms, logs, and audit trails to monitor the status of the deployed infrastructure. |
| **Week 7** | [Gateway VPC Endpoint for Amazon S3](1.7-week7/) | Practicing the workshop for private S3 access from a workload in VPC via Gateway Endpoint and route tables. |
| **Week 8** | [Interface Endpoint, Route 53, and Hybrid Connectivity](1.8-week8/) | Simulating S3 access from an on-premises environment, configuring Interface Endpoint and DNS resolution with Route 53. |
| **Week 9** | [VPC Endpoint Policies and Multi-layer Security](1.9-week9/) | Restricting S3 access using endpoint policy, bucket policy, and IAM policy, along with building a troubleshooting checklist. |
| **Week 10** | [Automation, documentation, and cost awareness](1.10-week10/) | Summarizing AWS CLI commands, creating helper scripts, reviewing costs, and standardizing workshop documentation. |
| **Week 11** | [Knowledge sharing and report drafting](1.11-week11/) | Converting technical parts into understandable content, drafting worklog/report, and preparing sharing materials. |
| **Week 12** | [Final review, self-evaluation, and internship wrap-up](1.12-week12/) | Reviewing the entire 12 weeks, finalizing the report, cleaning up lab resources, and determining next learning directions. |

---

### How I implemented the worklog during the internship

I pursued the **Learning by Doing** method and maintained the worklog structure to combine learning theory, hands-on practice, and recording practical technical lessons:

1. **Theoretical Research**
   I read official documentation from [AWS Documentation](https://docs.aws.amazon.com/) and workshop materials to correctly understand concepts before operating.

2. **Hands-on Deployment**
   I created resources directly on AWS using both **AWS Management Console** and **AWS CLI** to understand both operational methods simultaneously.

3. **Observing connections between services**
   Each week, I didn't just record "what was created," but also paid attention to:
   * which service connects to which service
   * how the network path goes
   * how IAM permissions affect
   * which DNS, route table, or policy determines the final outcome

4. **Logging and Reflection**
   I updated the worklog weekly to record:
   * objectives
   * completed tasks
   * detailed deployment methods
   * achieved results
   * encountered difficulties
   * learned lessons and next directions

---

### Prominent AWS service groups in this roadmap

Over 12 weeks, the service groups I used and connected the most include:

* **Identity and Security:** IAM Users, Roles, Policies, MFA
* **Networking:** VPC, Subnets, Route Tables, Internet Gateway, NAT Gateway, Security Groups, NACLs
* **Compute:** Amazon EC2
* **Storage:** Amazon EBS, Amazon S3, Amazon EFS
* **Observability:** Amazon CloudWatch, AWS CloudTrail
* **Hybrid/Private Access:** Gateway VPC Endpoint, Interface VPC Endpoint, Route 53
* **Governance and Cost Awareness:** AWS Budgets, Billing, Cost Explorer

These services are not learned in isolation, but are gradually connected into a complete chain serving the practical problem of **private access to Amazon S3** in cloud and hybrid environments.

---

### Ultimate goal of the Worklog section

This worklog section is not only used to list the tasks I have completed during the internship, but also aims to demonstrate:

* My process of learning and applying AWS step by step
* How I connect AWS services into real architectures
* My troubleshooting mindset, security, and cost awareness during lab exercises
* My development in technical skills, self-learning ability, and presentation skills

Through the 12-week worklog, I wish to clearly demonstrate the journey from an AWS beginner to being able to understand, deploy, explain, and summarize a cloud architecture with many interconnected components.
