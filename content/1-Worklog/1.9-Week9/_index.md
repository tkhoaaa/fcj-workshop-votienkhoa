---
title: "Week 9 Worklog"
date: 2026-06-15
weight: 9
chapter: false
pre: "<b>1.9. </b>"
---
### Week 9 Objectives:

* Learn how to restrict service access using **VPC Endpoint Policies**.
* Strengthen troubleshooting skills for private Amazon S3 connectivity.
* Consolidate the core technical lessons from the workshop environment.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Review how IAM policies, S3 bucket policies, and VPC Endpoint Policies work together. | 15/06/2026 | 15/06/2026 | [Endpoint Policy Docs](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-access.html) |
| **2** | Configure an S3 VPC Endpoint Policy that limits access to a specific bucket or action scope. | 16/06/2026 | 16/06/2026 | [Workshop Policy Section](https://workshop-sample.fcjuni.com/5-workshop/5.5-policy/) |
| **3** | Test both successful and denied access scenarios using AWS CLI from the lab EC2 instance. | 17/06/2026 | 17/06/2026 | AWS CLI |
| **4** | Create a troubleshooting checklist covering route tables, security groups, DNS, IAM, bucket policy, and endpoint policy. | 18/06/2026 | 18/06/2026 | Self-study notes |
| **5** | Draft a short technical note or blog outline summarizing the workshop and security lessons learned. | 19/06/2026 | 19/06/2026 | Personal draft |
| **6** | Review and organize screenshots, policy snippets, and command outputs for the report. | 20/06/2026 | 20/06/2026 | Report preparation notes |

### Detailed Implementation:

#### 1. Understand policy enforcement at multiple layers

This week focused on the idea that even with correct connectivity, service access can still be restricted intentionally. I studied how:

* **IAM policies**
* **S3 bucket policies**
* **VPC Endpoint Policies**

can all affect the same request differently.

Instead of treating access control as one simple rule, I approached it as a layered evaluation model.

#### 2. Restrict access through the endpoint itself

I configured a **VPC Endpoint Policy** to allow access only to a specific S3 bucket or limited actions. This meant that even if the instance had broader permissions somewhere else, the endpoint path itself could still act as a filter.

This created a more advanced integration chain:

* **EC2 with IAM Role**
* **Private network path through endpoint**
* **Endpoint policy check**
* **S3 bucket policy check**
* **Final object access decision**

#### 3. Validate allow and deny behavior using AWS CLI

I used AWS CLI from the EC2 instance to test:

* allowed commands
* denied commands
* expected failure cases

This was useful because it turned policy learning into observable behavior instead of abstract reading.

#### 4. Build a repeatable troubleshooting flow

By the end of the week, I had a checklist that started with:

* network path
* DNS behavior
* endpoint status
* IAM role permissions
* S3 bucket policy
* endpoint policy

This made troubleshooting more methodical and reduced confusion when access failed.

### AWS Service Integration This Week:

* **IAM Role + EC2:** The workload identity determined what the instance could request.
* **EC2 + VPC Endpoint:** All S3 traffic followed the controlled private path.
* **VPC Endpoint Policy + S3 Bucket Policy:** Access was filtered at both network-entry and storage-resource layers.
* **AWS CLI + Policy Testing:** Command-line tests were used to validate security behavior end to end.
* **Workshop Architecture + Governance:** Security controls were applied on top of the already working architecture.

### Week 9 Achievements:

* Understood how **VPC Endpoint Policies** add an extra control layer to private service access.
* Gained practical experience testing both successful and intentionally denied S3 operations.
* Improved my mental model of policy evaluation across multiple AWS services.
* Produced a troubleshooting method that can be reused in later labs or real projects.
* Strengthened the security depth of the workshop beyond simple connectivity.

### Challenges Faced:

* It was sometimes difficult to identify which policy layer caused a request to fail.
* Careful ARN and action matching was required to avoid unintended results.

### Lessons Learned and Next Steps:

* A secure AWS design is not only about enabling access, but also about limiting access at the right control points.
* In the next week, I would focus on **documentation**, **automation**, and **cost awareness** so the workshop becomes easier to maintain and present professionally.
