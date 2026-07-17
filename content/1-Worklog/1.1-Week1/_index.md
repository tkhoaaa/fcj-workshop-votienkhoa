---
title: "Week 1 Worklog"
date: 2026-04-20
weight: 1
chapter: false
pre: "<b>1.1. </b>"
---

### Week 1 Objectives:

* Get acquainted with the internship environment and the First Cloud AI Journey (FCAJ) team.
* Understand the core concepts of Cloud Computing and AWS Global Infrastructure.
* Set up a secure AWS environment (Free Tier) for practical labs.
* Establish the basic workflow that will be used throughout the 12-week internship.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Join the team, understand internship regulations and workflow, and research Cloud Computing models such as IaaS, PaaS, and SaaS. | 20/04/2026 | 20/04/2026 | [FCAJ Guide](https://workshop-sample.fcjuni.com/) |
| **2** | Create an AWS Free Tier account and apply security best practices such as enabling MFA for the Root account and creating an IAM administrator user. | 21/04/2026 | 21/04/2026 | [AWS IAM Docs](https://docs.aws.amazon.com/iam/) |
| **3** | Learn about AWS Regions and Availability Zones, then configure AWS Budget Alerts to control costs during labs. | 22/04/2026 | 22/04/2026 | [AWS Billing](https://aws.amazon.com/billing/) |
| **4** | Install and configure AWS CLI on the local machine, create access keys, and define the default profile for `ap-southeast-1`. | 23/04/2026 | 23/04/2026 | [AWS CLI Guide](https://docs.aws.amazon.com/cli/) |
| **5** | Study Amazon EC2 fundamentals including instance types, AMIs, key pairs, and the Shared Responsibility Model. | 24/04/2026 | 25/04/2026 | [Amazon EC2](https://aws.amazon.com/ec2/) |
| **6** | Launch the first EC2 instance using Amazon Linux 2023 and test access through both SSH and AWS Instance Connect. | 26/04/2026 | 26/04/2026 | [FCAJ Workshop](https://awsstudygroup.com/) |

### Detailed Implementation:

#### 1. Secure the AWS account before creating resources

The first practical step was not creating EC2, but securing the AWS account properly. I logged in with the Root account, enabled **MFA**, and then created a separate **IAM administrator user** for daily work. This follows AWS best practice because the Root account should only be used for sensitive account-level actions.

I also reviewed the difference between:

* **Root account** for account ownership
* **IAM user** for day-to-day administration
* **IAM policy** for permission control

This security setup became the foundation for all later labs because every AWS service in later weeks would be created under this controlled access model.

#### 2. Configure AWS CLI and connect local machine to AWS account

After the IAM user was ready, I created an access key and configured AWS CLI locally by using:

```bash
aws configure
aws sts get-caller-identity
```

This step connected the **local workstation** to the **AWS account** so I could manage services from both the web console and the command line. That dual workflow became very useful later when testing networking, S3 access, and endpoint behavior.

#### 3. Understand how EC2 fits into the AWS environment

Before launching the instance, I reviewed how EC2 depends on several connected AWS components:

* **AMI** provides the machine image
* **Key Pair** is used for secure SSH access
* **Security Group** controls inbound and outbound traffic
* **Region/AZ** defines where the instance is placed
* **EBS** acts as the attached block storage volume

This helped me understand that EC2 is not an isolated service. It works together with **IAM**, **network security**, and **storage** from the beginning.

#### 4. Launch the first EC2 instance

I launched an Amazon Linux 2023 instance from the AWS Console, selected a suitable instance type for Free Tier practice, attached the default EBS volume, and created or selected a Key Pair. I then opened the minimum required inbound rules in the Security Group for SSH access.

After the instance was created, I tested connectivity using:

* **AWS Instance Connect** directly from the browser
* **SSH** from the local machine using the downloaded private key

This lab demonstrated the first real integration between:

* **IAM user permissions**
* **EC2**
* **Security Group**
* **Key Pair**
* **EBS volume**

### AWS Service Integration This Week:

* **IAM + AWS Account Security:** IAM users were used instead of the Root account for daily administration.
* **AWS CLI + IAM Credentials:** CLI access was authenticated through the IAM user’s access keys.
* **EC2 + Security Group:** Network access to the instance was controlled through Security Group rules.
* **EC2 + EBS:** The instance used attached block storage as its system disk.
* **Region + Availability Zone + EC2:** Compute resources were launched in a specific AWS location for later lab consistency.

### Week 1 Achievements:

* Mastered the core benefits of Cloud Computing and the purpose of AWS Global Infrastructure.
* Successfully secured the AWS account using **MFA** and the **least privilege** approach.
* Installed and configured **AWS CLI** and verified account identity through `aws sts get-caller-identity`.
* Launched and accessed the first **EC2 instance** through both browser-based and terminal-based methods.
* Built the initial hands-on foundation that would support all service integrations in later weeks.

### Challenges Faced:

* At the beginning, there were many new concepts introduced at once such as IAM, CLI, Regions, and EC2 dependencies.
* Understanding why account security must come before infrastructure setup required a shift from a purely technical mindset to a governance mindset.

### Lessons Learned and Next Steps:

* Every AWS lab depends on a secure and correctly configured account foundation.
* In the following week, I would move from basic compute setup to **VPC networking**, where EC2 would be placed inside a custom network environment instead of relying only on default settings.
