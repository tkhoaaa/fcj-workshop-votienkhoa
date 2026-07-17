---
title: "Week 8 Worklog"
date: 2026-06-08
weight: 8
chapter: false
pre: "<b>1.8. </b>"
---
### Week 8 Objectives:

* Extend the workshop to support hybrid access patterns using an **Interface VPC Endpoint**.
* Understand how DNS resolution affects private connectivity to Amazon S3.
* Practice troubleshooting private access from an on-premises simulation environment.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Compare **Gateway Endpoints** and **Interface Endpoints** in terms of architecture, use case, and cost considerations. | 08/06/2026 | 08/06/2026 | [AWS PrivateLink Docs](https://docs.aws.amazon.com/vpc/latest/privatelink/) |
| **2** | Create the **S3 Interface Endpoint** and review subnet placement and security group rules. | 09/06/2026 | 09/06/2026 | [Create Interface Endpoint](https://workshop-sample.fcjuni.com/5-workshop/5.4-cloudfront-amazon-quick/5.4.2-cloudfront-foundation-benefits/) |
| **3** | Test private connectivity from the simulated on-premises EC2 instance to Amazon S3. | 10/06/2026 | 10/06/2026 | [Test Interface Endpoint](https://workshop-sample.fcjuni.com/5-workshop/5.4-cloudfront-amazon-quick/5.4.3-amazon-quick-ai-assistant/) |
| **4** | Configure and verify DNS simulation using Route 53 private hosted zones and custom records. | 11/06/2026 | 11/06/2026 | [DNS Simulation Lab](https://workshop-sample.fcjuni.com/5-workshop/5.4-cloudfront-amazon-quick/5.4.4-practical-reflection/) |
| **5** | Investigate common failure points related to DNS, security groups, and endpoint configuration. | 12/06/2026 | 12/06/2026 | Troubleshooting notes |
| **6** | Document key observations about hybrid connectivity and prepare screenshots for the workshop report. | 13/06/2026 | 13/06/2026 | Self-study notes |

### Detailed Implementation:

#### 1. Move from VPC-only access to hybrid access design

In Week 7, the S3 access path was designed for workloads already inside a VPC. This week expanded the architecture to support an **on-premises simulation** by using an **Interface Endpoint** instead.

This changed the integration model from:

* **EC2 inside VPC** -> **Gateway Endpoint** -> **S3**

to a more hybrid-oriented model:

* **Simulated on-premises workload**
* **VPC network path**
* **Interface Endpoint**
* **private DNS resolution**
* **Amazon S3**

#### 2. Create the Interface Endpoint with correct network placement

I created the **S3 Interface Endpoint**, selected the relevant subnets, and attached a suitable **Security Group**. Unlike Gateway Endpoints, Interface Endpoints rely on elastic network interfaces and subnet placement, so the network-level setup became more detailed.

This created a direct relationship between:

* **Interface Endpoint**
* **Subnets**
* **Security Groups**
* **Private IP-based connectivity**

#### 3. Use DNS to direct traffic to the private endpoint

The key learning this week was that private connectivity depends not only on network resources, but also on **name resolution**. I used the workshop DNS simulation to understand how **Route 53** and private DNS records can direct S3 requests toward the endpoint instead of a public address path.

This demonstrated a powerful cloud integration pattern:

* **Application request**
* **DNS resolution**
* **Private endpoint**
* **AWS service**

#### 4. Troubleshoot by checking each layer separately

When validating the setup, I checked:

* endpoint status
* security group rules
* subnet and route alignment
* DNS response behavior
* IAM access

This layered troubleshooting process made the hybrid lab much more understandable and also mirrored real-world cloud debugging practice.

### AWS Service Integration This Week:

* **Interface Endpoint + Subnets:** Private connectivity was placed into selected VPC subnets.
* **Interface Endpoint + Security Group:** Traffic to the endpoint network interfaces was controlled.
* **Route 53 + DNS Simulation:** Name resolution redirected application requests to private paths.
* **EC2 On-Prem Simulation + S3:** A simulated external workload accessed S3 through private AWS networking.
* **IAM + Endpoint Path + DNS:** Successful access required permissions, connectivity, and name resolution to align.

### Week 8 Achievements:

* Understood the practical difference between **Gateway** and **Interface** endpoints.
* Learned how **DNS** can be just as important as routing in private service connectivity.
* Successfully practiced a hybrid-style connectivity workflow in a workshop environment.
* Strengthened troubleshooting skills by validating one layer at a time.
* Collected strong technical material for explaining hybrid AWS architecture later in the report.

### Challenges Faced:

* DNS behavior was more complex than expected because a small configuration mistake could redirect traffic incorrectly.
* The hybrid scenario introduced more moving parts than the VPC-only scenario from the previous week.

### Lessons Learned and Next Steps:

* Private service access in AWS often depends on the combination of networking, DNS, and identity, not only one of those layers.
* In the following week, I would reinforce the security side of this architecture by using **VPC Endpoint Policies** and testing restricted access behavior.
