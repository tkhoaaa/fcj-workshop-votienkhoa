---
title: "Week 7 Worklog"
date: 2026-06-01
weight: 7
chapter: false
pre: "<b>1.7. </b>"
---
### Week 7 Objectives:

* Prepare the workshop environment for private access to Amazon S3.
* Understand the architecture and use case of **Gateway VPC Endpoints**.
* Practice configuring network routing for S3 access without using the public Internet.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Read the workshop overview and redraw the target architecture for private S3 connectivity. | 01/06/2026 | 01/06/2026 | [Workshop Overview](https://workshop-sample.fcjuni.com/5-workshop/5.1-workshop-overview/) |
| **2** | Review the lab environment including VPC, subnets, route tables, EC2 instances, and supporting resources. | 02/06/2026 | 02/06/2026 | Workshop notes |
| **3** | Create a **Gateway VPC Endpoint** for Amazon S3 in the workshop VPC. | 03/06/2026 | 03/06/2026 | [Gateway Endpoint Lab](https://workshop-sample.fcjuni.com/5-workshop/5.3-lotushacks-utmorpho/5.3.1-from-zero-to-idea/) |
| **4** | Verify route table updates and confirm that S3 traffic from the VPC uses the endpoint path. | 04/06/2026 | 04/06/2026 | [VPC Endpoints Docs](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html) |
| **5** | Test basic bucket access from an EC2 instance and document the result with screenshots and CLI output. | 05/06/2026 | 05/06/2026 | [Test Gateway Endpoint](https://workshop-sample.fcjuni.com/5-workshop/5.3-lotushacks-utmorpho/5.3.2-building-under-pressure/) |
| **6** | Summarize when Gateway Endpoints are appropriate and what limitations they have compared to Interface Endpoints. | 06/06/2026 | 06/06/2026 | Self-study notes |

### Detailed Implementation:

#### 1. Start from the workshop architecture instead of isolated resources

This week I moved from learning individual services to understanding a connected architecture. The goal was not simply “create an endpoint,” but to enable **private S3 access from a workload inside a VPC**.

I reviewed the full path involved:

* **EC2 instance inside VPC**
* **Route table associated with subnet**
* **Gateway VPC Endpoint**
* **Amazon S3**

This made it clear that endpoint configuration is really a networking integration task.

#### 2. Create the Gateway VPC Endpoint for S3

I opened the VPC console and created a **Gateway Endpoint** for Amazon S3. During this process, I selected:

* the correct **VPC**
* the correct **route table**
* the default endpoint policy for initial testing

This step updated the route logic so S3 requests from the selected subnets could travel through the endpoint rather than the public Internet path.

#### 3. Test S3 access from EC2 inside the VPC

From an EC2 instance in the VPC, I ran test commands using AWS CLI to access an S3 bucket. The purpose was to verify both:

* network path behavior
* IAM permission behavior

This showed the first strong connection between weeks:

* **Week 2:** VPC and route tables
* **Week 5:** IAM roles and permissions
* **Week 7:** S3 private access through a Gateway Endpoint

#### 4. Observe how networking and permissions must both be correct

The test would only succeed when multiple layers were aligned:

* EC2 must have valid IAM access
* the subnet must be associated with the correct route table
* the Gateway Endpoint must be attached properly
* S3 target access must be allowed

This was an important lesson because AWS integrations are often multi-layered rather than isolated to one service.

### AWS Service Integration This Week:

* **EC2 + IAM Role:** The instance authenticated to AWS services using temporary credentials.
* **EC2 + VPC/Subnet/Route Table:** Network routing defined how traffic left the instance.
* **Route Table + Gateway VPC Endpoint:** S3 traffic was redirected through a private AWS-managed path.
* **Gateway Endpoint + S3:** Object storage became reachable without public Internet routing.
* **Workshop Architecture + Earlier Weeks:** Networking, identity, and storage all came together in one lab.

### Week 7 Achievements:

* Understood the use case and benefits of **Gateway VPC Endpoints** for Amazon S3.
* Successfully configured private S3 access from workloads inside the VPC.
* Connected networking knowledge with storage and IAM behavior in a practical architecture.
* Built stronger confidence in reading workshop diagrams and translating them into AWS configurations.
* Prepared the groundwork for the more advanced hybrid access scenario in the next week.

### Challenges Faced:

* Selecting the correct route table for the lab required careful verification.
* It was not enough to create the endpoint; I had to confirm that the traffic path actually changed as expected.

### Lessons Learned and Next Steps:

* Private connectivity in AWS is usually the result of correct integration between network design and access control.
* In the next week, I would extend this model to **Interface Endpoints**, **DNS**, and **on-premises simulation** for hybrid architecture practice.
