---
title: "Week 1 Worklog"
date: 2026-04-20
weight: 1
chapter: false
pre: "<b>1.1. </b>"
---

### Week 1 Objectives:

* Complete onboarding with the First Cloud AI Journey (FCAJ) team and set up a secure AWS Free Tier account for the whole internship.
* Learn the fundamentals of **Amazon EC2** and **Amazon VPC** and finish the basic hands-on lab exercises.
* Study **AWS Elastic Disaster Recovery (DRS)** at a concept level, including replication, RPO and RTO.
* Trial-run an **AWS SAM** deployment with **AWS Systems Manager Parameter Store** for secrets, and create an **AMI** for backup and repeatable deployment.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Join the FCAJ team, review internship regulations and workflow, create the AWS Free Tier account, enable MFA on the Root account and create an IAM administrator user. | 20/04/2026 | 20/04/2026 | [FCAJ Guide](https://workshop-sample.fcjuni.com/) |
| **2** | Install and configure AWS CLI for `ap-southeast-1`, set up AWS Budget alerts, and study Amazon EC2 fundamentals: instance types, AMIs, key pairs, Security Groups, EBS. | 21/04/2026 | 21/04/2026 | [AWS CLI Guide](https://docs.aws.amazon.com/cli/) |
| **3** | Study Amazon VPC fundamentals and complete the basic lab: launch an Amazon Linux 2023 instance in a custom VPC and verify access through SSH and EC2 Instance Connect. | 22/04/2026 | 22/04/2026 | [Amazon VPC](https://aws.amazon.com/vpc/) |
| **4** | Research AWS Elastic Disaster Recovery: replication agent, staging area subnet, point-in-time recovery, and how RPO and RTO drive the recovery design. | 23/04/2026 | 23/04/2026 | [AWS DRS Docs](https://docs.aws.amazon.com/drs/) |
| **5** | Trial an AWS SAM deploy of a small Lambda and API Gateway stack, store a secret in SSM Parameter Store as SecureString, and create an AMI from the EC2 instance for backup. | 24/04/2026 | 24/04/2026 | [AWS SAM Docs](https://docs.aws.amazon.com/serverless-application-model/) |

### Detailed Implementation:

#### 1. Onboarding and securing the AWS Free Tier account

The week started with FCAJ onboarding: internship regulations, the weekly reporting workflow, and how the capstone project would be reviewed. The first hands-on step was not launching a server but securing the account. I created the AWS Free Tier account, enabled **MFA** on the Root user, then created an **IAM** administrator user for daily work so the Root user is reserved for account-level actions only.

I then configured **AWS CLI** on my machine and pinned `ap-southeast-1` as the default region, so console work and command-line work always target the same place.

```bash
aws configure --profile fcaj
aws sts get-caller-identity --profile fcaj
aws budgets describe-budgets --account-id <account-id>
```

I also created an **AWS Budgets** alert with a small monthly threshold. Cost control matters here because later weeks add managed services that are only partly covered by Free Tier.

#### 2. Amazon EC2 and Amazon VPC fundamentals

Working through the basic labs, I launched an Amazon Linux 2023 instance and traced every component it depends on:

* **AMI** supplies the machine image
* **Key Pair** authenticates SSH access
* **Security Group** filters inbound and outbound traffic at the instance level
* **Amazon EBS** provides the attached root volume
* **Region and Availability Zone** decide where the instance physically runs

Then I moved the instance into a custom **Amazon VPC** instead of the default one. I created the VPC with a `/16` CIDR, one public subnet, an internet gateway, and a route table sending `0.0.0.0/0` to that gateway. Verifying access through both **EC2 Instance Connect** and SSH from my laptop made the dependency chain concrete: without a correct route table entry the instance is reachable in the console but not over the network.

The key takeaway is that EC2 is never standalone. IAM controls who may launch it, VPC decides whether it can be reached, and EBS holds its state.

#### 3. Studying AWS Elastic Disaster Recovery

I studied **AWS Elastic Disaster Recovery (DRS)** rather than deploying it end to end, because a full DRS drill needs source servers and continuous replication storage beyond what Free Tier allows. I documented the moving parts:

* A **replication agent** installed on the source server streams block-level changes to AWS.
* A **staging area subnet** holds low-cost replication servers and EBS volumes, not full-size instances.
* **Point-in-time recovery** snapshots let you launch a recovery instance from a chosen moment rather than only the latest state.
* **RPO** describes how much data loss is acceptable, **RTO** how long the recovery may take. Those two numbers drive the whole design.

What connected this back to the earlier labs is that a DRS recovery instance lands inside a VPC and subnet you prepare in advance, with Security Groups and IAM roles already in place. Disaster recovery is a networking and identity exercise as much as a storage one.

#### 4. SAM deploy trial, Parameter Store, and AMI backup

Last, I ran an exploratory **AWS SAM** deployment: a minimal template with one **AWS Lambda** function behind **Amazon API Gateway**. `sam deploy --guided` created the CloudFormation stack, the S3 artifact bucket, and the execution role, which showed me that SAM is a thin layer over CloudFormation rather than a separate deployment system.

To avoid putting a secret in the template, I stored it in **AWS Systems Manager Parameter Store** as a SecureString and referenced it from the template at deploy time.

```bash
aws ssm put-parameter --name "/lingorise/dev/demo/api-key" \
  --value "<secret>" --type SecureString --region ap-southeast-1
sam deploy --guided --region ap-southeast-1
```

The template resolved it with `{{resolve:ssm:/lingorise/${Stage}/demo/api-key}}`, so the secret value never appears in source control. This trial later becomes the deployment path for the **LingoRise** capstone project.

Finally I created an **AMI** from the configured EC2 instance. The image captures the OS, installed packages, and EBS snapshot, so I can relaunch an identical instance instead of repeating setup by hand. That gives me both a backup and a repeatable deployment baseline.

### AWS Service Integration This Week:

* **IAM + AWS CLI:** The CLI authenticated with the IAM administrator user's access keys instead of Root credentials.
* **EC2 + VPC + Security Group:** Instance reachability came from the subnet route table and the internet gateway, while the Security Group filtered traffic at the instance boundary.
* **EC2 + EBS + AMI:** The AMI packaged the instance configuration together with an EBS snapshot for repeatable launches.
* **AWS DRS + VPC + EBS:** DRS replicates into staging EBS volumes and launches recovery instances into a VPC and subnet prepared beforehand.
* **AWS SAM + CloudFormation + Lambda + API Gateway:** SAM expanded into a CloudFormation stack that provisioned the function, the REST API, and the execution role together.
* **SSM Parameter Store + CloudFormation:** The stack resolved a SecureString parameter at deploy time so no secret lived in the template.

### Week 1 Achievements:

* Completed FCAJ onboarding and secured the AWS Free Tier account with **MFA**, an IAM administrator user, and a Budgets alert.
* Configured **AWS CLI** for `ap-southeast-1` and verified identity with `aws sts get-caller-identity`.
* Finished the basic labs: launched an Amazon Linux 2023 instance inside a custom **VPC** and verified SSH plus EC2 Instance Connect access.
* Documented the **AWS Elastic Disaster Recovery** model, including replication agent, staging subnet, point-in-time recovery, and RPO/RTO tradeoffs.
* Ran a successful **AWS SAM** deploy trial with a secret held in **SSM Parameter Store** as SecureString.
* Created an **AMI** from the working instance for backup and repeatable deployment.

### Challenges Faced:

* Many concepts arrived at once. IAM, VPC routing, DRS terminology, and SAM packaging each have their own vocabulary, and it took effort to see how they compose.
* DRS could not be fully deployed within Free Tier limits, so I had to be explicit that this part was studied and documented rather than exercised in a real failover drill.
* The first `sam deploy` failed on IAM permissions until I understood that SAM needs rights to create CloudFormation stacks, roles, and the artifact bucket, not just the Lambda function.

### Lessons Learned and Next Steps:

* Account security and cost guardrails belong before infrastructure, not after. Every later lab inherits that foundation.
* Infrastructure as code plus Parameter Store keeps configuration reproducible and secrets out of the repository, which is exactly what the capstone project needs.
* Next week I move into **Amazon VPC** components in depth, covering subnets, route tables, internet gateway, NAT Gateway, and NACLs, then AWS database services with **Amazon RDS** and **Amazon DynamoDB**, and **AWS IAM** in more detail.
