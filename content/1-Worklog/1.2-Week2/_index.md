---
title: "Week 2 Worklog"
date: 2026-04-27
weight: 2
chapter: false
pre: "<b>1.2. </b>"
---

### Week 2 Objectives:

* Understand each VPC component in depth: subnets, route tables, Internet Gateway, NAT Gateway, and Network ACLs.
* Build a custom VPC by hand instead of reusing the default one, with public and private subnets across two Availability Zones.
* Compare Amazon RDS and Amazon DynamoDB and decide which fits the LingoRise data model.
* Learn AWS IAM in practice: users, groups, roles, and policy documents under least privilege.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Study VPC components in depth: CIDR planning, subnets, route tables, Internet Gateway, NAT Gateway, and Network ACLs. | 27/04/2026 | 27/04/2026 | [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) |
| **2** | Build a custom VPC `10.0.0.0/16` in `ap-southeast-1` with two public and two private subnets, then attach an Internet Gateway and a NAT Gateway and fix the route table associations. | 28/04/2026 | 28/04/2026 | [NAT Gateway Docs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) |
| **3** | Compare Security Groups and Network ACLs by testing allowed and denied traffic to confirm stateful versus stateless behavior. | 29/04/2026 | 29/04/2026 | [Security in Amazon VPC](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Security.html) |
| **4** | Research Amazon RDS for PostgreSQL: DB subnet groups, parameter groups, automated backups, then contrast it with Amazon DynamoDB partition and sort keys and on-demand capacity. | 30/04/2026 | 30/04/2026 | [Amazon RDS](https://aws.amazon.com/rds/) |
| **5** | Study AWS IAM users, groups, roles, and policies; read a policy JSON document line by line and apply least privilege to the lab account. | 01/05/2026 | 01/05/2026 | [AWS IAM Docs](https://docs.aws.amazon.com/iam/) |

### Detailed Implementation:

#### 1. Plan the CIDR range before creating a single subnet

I started from the address plan, not from the console. The **VPC** used `10.0.0.0/16`, then I carved out four `/24` subnets so the layout stayed readable:

* `10.0.1.0/24` public in `ap-southeast-1a`, `10.0.2.0/24` public in `ap-southeast-1b`
* `10.0.11.0/24` private in `ap-southeast-1a`, `10.0.12.0/24` private in `ap-southeast-1b`

Two Availability Zones were not optional. **Amazon RDS** requires a DB subnet group with subnets in at least two AZs, so the network had to be multi-AZ before the database existed. Reserving the `10.0.1x.0/24` block for private subnets also left room to grow without renumbering.

#### 2. Wire routing with Internet Gateway, NAT Gateway, and separate route tables

I attached an **Internet Gateway** to the VPC and gave the public route table a `0.0.0.0/0` route to it. A subnet only becomes public through that route, not through its name, which was the point I had been getting wrong.

For the private subnets I placed a **NAT Gateway** in a public subnet with an Elastic IP, then pointed the private route table's default route at the NAT Gateway. Outbound package installs work, inbound connections do not. The traffic path is:

* **Private subnet** -> private route table -> **NAT Gateway** -> **Internet Gateway** -> Internet

I then compared **Security Groups** against **Network ACLs** by blocking port 22 at each layer in turn. The Security Group is stateful, so allowing inbound SSH is enough and the reply leaves automatically. The NACL is stateless, so denying the ephemeral port range `1024-65535` on the outbound rules broke the same SSH session even though inbound was allowed. That test made the distinction stick better than any diagram.

#### 3. Choose between Amazon RDS and Amazon DynamoDB for LingoRise

I read both services with the LingoRise question bank in mind. **Amazon DynamoDB** gives single-digit millisecond reads, on-demand capacity with no instance to size, and a data model driven entirely by the partition key and sort key. **Amazon RDS** gives a managed relational engine with a DB subnet group, parameter groups, automated backups, and a maintenance window.

LingoRise queries do not fit a key-value access pattern. Exam sessions join to questions, questions join to passages and assets, and the admin review queue filters on several columns at once. Question metadata is semi-structured, which PostgreSQL handles with `JSONB` while still allowing SQL joins. Schema changes ship as idempotent migrations tracked in a `_migrations` table, and that only makes sense against a relational engine.

So the project settled on **Amazon RDS for PostgreSQL** on `db.t4g.micro` with 20 GB of storage, placed in the private subnets from task 2 and never given a public endpoint. DynamoDB stays a good fit for pure key lookups, which LingoRise does not currently need.

#### 4. Read IAM as a set of policy documents, not console checkboxes

I created an `fcj-developers` group, attached permissions there, and added my IAM user to the group so no permission was ever pinned to a single user. Then I created a **role** and read the difference in practice: a user has long-lived credentials, a role is assumed and returns temporary credentials, which is what a Lambda function will use later instead of an access key.

The part that mattered was reading a policy JSON instead of trusting the console summary:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["ssm:GetParameter", "ssm:GetParameters"],
    "Resource": "arn:aws:ssm:ap-southeast-1:*:parameter/lingorise/dev/*"
  }]
}
```

Every element does work: `Action` is the API call, `Resource` scopes it to one parameter path, and the wildcard sits at the end of the path rather than on the whole service. I also compared identity-based policies, attached to a user, group, or role, against resource-based policies such as an S3 bucket policy, which name a `Principal` and live on the resource itself. Then I confirmed the effective permissions with:

```bash
aws iam list-attached-group-policies --group-name fcj-developers
aws sts get-caller-identity
```

### AWS Service Integration This Week:

* **VPC + Subnets + Availability Zones:** The address space was split into public and private subnets in two AZs so later services could be placed by exposure level.
* **Route Tables + Internet Gateway:** A `0.0.0.0/0` route to the IGW is what actually makes a subnet public.
* **Private Subnets + NAT Gateway:** Private workloads got outbound-only Internet access for package installs without any inbound path.
* **Security Groups + Network ACLs:** Instance-level stateful filtering was layered under subnet-level stateless filtering.
* **Amazon RDS + VPC:** The DB subnet group ties the database to the private subnets, which is why the network had to be designed first.
* **IAM + AWS CLI + SSM Parameter Store:** A group policy scoped to `/lingorise/dev/*` let the CLI read only the parameters that stage needs.

### Week 2 Achievements:

* Built a custom **VPC** end to end from a written CIDR plan, with four subnets across two Availability Zones.
* Proved by testing that a subnet is public only because of its route table entry, not its name or tag.
* Demonstrated stateful versus stateless filtering by breaking an SSH session with a **Network ACL** outbound rule while the **Security Group** still allowed it.
* Justified **Amazon RDS for PostgreSQL** over **Amazon DynamoDB** for LingoRise using concrete reasons: relational joins, `JSONB` metadata, and SQL migrations.
* Applied **IAM** least privilege through a group policy scoped to a single Parameter Store path, and read the policy JSON rather than the console summary.

### Challenges Faced:

* The NAT Gateway has to sit in a public subnet while serving private subnets, which felt backwards until I traced the packet path through both route tables.
* Network ACL rules are evaluated in number order and need matching outbound entries for return traffic, so my first deny rule blocked more than intended.
* Choosing between RDS and DynamoDB was not a performance question but a data-model question, and I initially compared them on the wrong criteria.

### Lessons Learned and Next Steps:

* Networking and identity are prerequisites, not later steps. The DB subnet group and the IAM role both depend on decisions made before any workload exists.
* A policy document is readable. Once I could parse `Effect`, `Action`, and `Resource`, permission errors stopped being guesswork.
* In the next week, I would put this VPC to work with a real workload: an **Nginx** web server on **Amazon Linux** in the public subnet, **S3 Event Notifications** triggering a **Lambda** function, and an **AMI** captured from the configured instance.
