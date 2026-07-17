---
title: "Week 6 Worklog"
date: 2026-05-25
weight: 6
chapter: false
pre: "<b>1.6. </b>"
---
### Week 6 Objectives:

* Improve system observability using Amazon CloudWatch and AWS CloudTrail.
* Learn how to monitor EC2 instances, create alarms, and review AWS API activity.
* Develop an operational mindset for detecting and troubleshooting issues earlier.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Study the main components of Amazon CloudWatch: Metrics, Logs, Dashboards, and Alarms. | 25/05/2026 | 25/05/2026 | [Amazon CloudWatch Docs](https://docs.aws.amazon.com/cloudwatch/) |
| **2** | Create CloudWatch Alarms for EC2 metrics such as CPU utilization and instance status checks. | 26/05/2026 | 26/05/2026 | [Using Amazon CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html) |
| **3** | Install or review CloudWatch Agent configuration to forward additional logs and metrics from EC2. | 27/05/2026 | 27/05/2026 | [CloudWatch Agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html) |
| **4** | Explore AWS CloudTrail and review event history for actions performed during previous lab exercises. | 28/05/2026 | 28/05/2026 | [AWS CloudTrail Docs](https://docs.aws.amazon.com/cloudtrail/) |
| **5** | Create a small monitoring checklist for EC2-based workloads, including health, access, and performance signals. | 29/05/2026 | 29/05/2026 | Self-study notes |
| **6** | Write a short operations note describing how logs and alarms help reduce troubleshooting time. | 30/05/2026 | 30/05/2026 | Self-study notes |

### Detailed Implementation:

#### 1. Add monitoring to the EC2 workload from previous weeks

I reused the EC2-based environment built in earlier weeks and connected it to **CloudWatch**. Instead of treating the server as a black box, I monitored:

* CPU utilization
* status checks
* basic health signals

This was the first step in turning a simple instance into a manageable cloud workload.

#### 2. Connect EC2 logs and metrics to CloudWatch

By reviewing or installing **CloudWatch Agent**, I learned how operating system-level data can be pushed from **EC2** into **CloudWatch Logs** and **CloudWatch Metrics**. This created a useful operations flow:

* **EC2 operating system**
* **CloudWatch Agent**
* **CloudWatch Logs / Metrics**
* **Alarms and dashboards**

This integration is important because later networking labs become much easier when logs and health data are visible centrally.

#### 3. Build alarms for operational awareness

I created **CloudWatch Alarms** around EC2 metrics so the system could notify or at least signal when resource usage becomes abnormal. Even in a lab setting, this introduced the idea that cloud infrastructure should be observable rather than manually checked.

#### 4. Use CloudTrail to trace administrative actions

In parallel, I reviewed **CloudTrail** event history to see which AWS API calls had been made during setup and testing. This showed the difference between:

* **CloudWatch** for performance and operational monitoring
* **CloudTrail** for audit and history of actions

Together, these two services created a strong visibility layer around the infrastructure already deployed.

### AWS Service Integration This Week:

* **EC2 + CloudWatch Metrics:** Instance performance became measurable.
* **EC2 + CloudWatch Agent + CloudWatch Logs:** OS-level data was centralized.
* **CloudWatch + Alarms:** Monitoring data was used for proactive detection.
* **AWS Account Activity + CloudTrail:** Administrative and API actions became traceable.
* **Operations + Existing Infrastructure:** Observability was layered on top of previously deployed compute and network resources.

### Week 6 Achievements:

* Built a practical understanding of **CloudWatch** and **CloudTrail** in an operational context.
* Improved visibility into instance health, performance, and activity history.
* Learned how central logs and alarms support faster troubleshooting.
* Connected infrastructure deployment with basic observability practices.
* Established monitoring habits that would be valuable in later hybrid connectivity labs.

### Challenges Faced:

* It took time to decide which metrics were actually meaningful rather than simply available by default.
* Distinguishing the roles of CloudWatch and CloudTrail required repeated comparison through hands-on use.

### Lessons Learned and Next Steps:

* Good cloud operations rely on being able to observe, trace, and explain system behavior.
* In the following week, I would apply this visibility mindset to the **Gateway VPC Endpoint** workshop for Amazon S3.
