---
title: "Week 5 Worklog"
date: 2026-05-18
weight: 5
chapter: false
pre: "<b>1.5. </b>"
---
### Week 5 Objectives:

* Strengthen my understanding of IAM, Roles, and AWS account security practices.
* Learn how to apply least-privilege access for users, services, and EC2 instances.
* Practice reviewing permissions from both the AWS Console and AWS CLI.
* Prepare access control patterns that can be reused in the S3 workshop.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Review IAM concepts including Users, Groups, Roles, and Policies. Compare managed policies and inline policies. | 18/05/2026 | 18/05/2026 | [AWS IAM Docs](https://docs.aws.amazon.com/iam/) |
| **2** | Enable additional account security checks such as MFA review, password policy validation, and access key rotation awareness. | 19/05/2026 | 19/05/2026 | [AWS Security Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) |
| **3** | Create an IAM Role for EC2 and attach a limited S3 access policy. Test the role from an EC2 instance using AWS CLI. | 20/05/2026 | 20/05/2026 | [IAM Roles for Amazon EC2](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html) |
| **4** | Write and validate a policy that only allows access to a specific S3 bucket and selected actions. | 21/05/2026 | 21/05/2026 | [IAM JSON Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html) |
| **5** | Study governance concepts such as AWS Organizations and Service Control Policies at a conceptual level. | 22/05/2026 | 22/05/2026 | [AWS Organizations](https://docs.aws.amazon.com/organizations/) |
| **6** | Summarize the week’s security lessons and document common mistakes that can lead to over-permissioned accounts. | 23/05/2026 | 23/05/2026 | Self-study notes |

### Detailed Implementation:

#### 1. Review access models before assigning permissions

I revisited how access is granted in AWS by comparing:

* **IAM Users** for human identities
* **IAM Groups** for permission grouping
* **IAM Roles** for temporary delegated access
* **Policies** as the permission language

This was important because later weeks required services to access other services, not just a user clicking through the Console.

#### 2. Connect EC2 to AWS services without hardcoded credentials

To practice secure service-to-service access, I created an **IAM Role** for EC2 and attached a policy that only allowed specific **S3 actions** on one bucket. Then I attached the role to an EC2 instance and tested access using AWS CLI from the instance itself.

This created a secure integration path:

* **EC2 instance** -> assumes **IAM Role**
* **IAM Role** -> has scoped **S3 permissions**
* **AWS CLI on EC2** -> accesses **S3** using temporary credentials

This was more secure than storing long-term access keys in the operating system.

#### 3. Write policies around actual resources

Instead of reading IAM policy syntax only theoretically, I wrote a policy with:

* specific **Action**
* specific **Resource ARN**
* limited access scope

Then I tested whether the instance could list, read, or upload objects depending on the permissions granted. This showed how IAM is directly connected to storage protection.

#### 4. Relate IAM to future workshop use cases

This week’s security work connected strongly to later workshop tasks because private S3 access is not useful unless permissions are also correct. I noted that even if:

* the network path works
* the endpoint is configured correctly

the request will still fail if the **IAM Role** or **S3 permissions** are incorrect.

### AWS Service Integration This Week:

* **IAM User + Console/CLI:** Administrative actions were performed through controlled user credentials.
* **EC2 + IAM Role:** The instance used a role instead of static access keys.
* **IAM Policy + S3:** Permissions were scoped to specific bucket resources and actions.
* **Governance Concepts + Account Security:** Policy thinking was expanded from one user or one service to an account-level perspective.

### Week 5 Achievements:

* Understood the practical difference between **Users**, **Groups**, **Roles**, and **Policies**.
* Successfully attached an **IAM Role** to EC2 and tested AWS service access without static credentials.
* Improved policy-writing confidence by connecting permissions to a real S3 use case.
* Developed a stronger least-privilege mindset for service access design.
* Prepared the permission model needed for later VPC Endpoint and S3 workshop tasks.

### Challenges Faced:

* IAM policy syntax was still detailed and unforgiving, especially when matching actions and resource ARNs.
* It took careful testing to distinguish between access failures caused by IAM and those caused by other factors.

### Lessons Learned and Next Steps:

* Secure service integration in AWS depends on both identity design and correct resource permissions.
* In the following week, I would extend this work by focusing on **CloudWatch** and **CloudTrail** so I could observe system behavior and detect access or performance issues more effectively.
