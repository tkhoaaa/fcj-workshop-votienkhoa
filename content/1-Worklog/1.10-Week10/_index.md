---
title: "Week 10 Worklog"
date: 2026-06-22
weight: 10
chapter: false
pre: "<b>1.10. </b>"
---
### Week 10 Objectives:

* Improve efficiency by documenting repeated commands and automating routine tasks.
* Review resource usage and apply basic cost awareness during lab work.
* Organize technical evidence for the final internship report.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Collect and categorize AWS CLI commands used throughout the workshop. | 22/06/2026 | 22/06/2026 | Personal CLI notes |
| **2** | Create a simple helper script or command checklist to support environment verification or cleanup. | 23/06/2026 | 23/06/2026 | AWS CLI Docs |
| **3** | Review AWS Billing, Budgets, and Cost Explorer to understand resource usage from the lab environment. | 24/06/2026 | 24/06/2026 | [AWS Billing Docs](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/) |
| **4** | Reorganize screenshots, diagrams, and step-by-step notes to prepare a cleaner workshop narrative. | 25/06/2026 | 25/06/2026 | Report preparation notes |
| **5** | Draft a structured outline for the report sections: objective, architecture, implementation, result, and lessons learned. | 26/06/2026 | 26/06/2026 | Personal outline |
| **6** | Review all documentation for missing details and update evidence where screenshots or outputs were incomplete. | 27/06/2026 | 27/06/2026 | Self-review |

### Detailed Implementation:

#### 1. Convert repeated manual steps into reusable operational assets

By this point in the internship, many AWS actions had been repeated several times. Instead of relying only on memory, I organized the most important **AWS CLI commands** into reusable notes and helper scripts.

These included commands related to:

* S3 bucket checks
* EC2 verification
* IAM identity confirmation
* endpoint testing
* cleanup preparation

This work connected technical execution with operational consistency.

#### 2. Use CLI-based verification to support workshop repeatability

I created a small verification flow so that after changing resources, I could quickly check:

* who I am authenticated as
* whether the expected EC2 or endpoint exists
* whether S3 access behaves correctly

This connected:

* **AWS CLI**
* **IAM identity**
* **EC2**
* **S3**
* **endpoint-based architecture**

and reduced the time needed for re-testing.

#### 3. Review cloud cost implications of lab resources

I revisited **AWS Budgets**, **Billing**, and **Cost Explorer** to identify which lab resources might continue generating costs if left running. This included special attention to:

* NAT Gateway
* Interface Endpoints
* EC2 instances
* storage resources

This showed that cloud architecture design should include not only technical correctness, but also cost discipline.

#### 4. Organize evidence for report-quality documentation

I grouped diagrams, screenshots, policy snippets, and CLI outputs by topic so they could support a proper technical report. This turned lab output into report-ready material and made it easier to explain service relationships clearly.

### AWS Service Integration This Week:

* **AWS CLI + IAM:** Identity verification supported safe command execution.
* **AWS CLI + S3/EC2/VPC Endpoints:** Service state and access behavior were validated programmatically.
* **Billing + Deployed Resources:** Cost analysis was tied back to actual infrastructure decisions.
* **Documentation + Workshop Architecture:** Technical evidence was organized around real service integrations rather than random screenshots.

### Week 10 Achievements:

* Built a reusable set of CLI references and verification steps for the workshop environment.
* Improved repeatability and reduced testing time through documentation and simple automation.
* Increased cost awareness around network and endpoint-based lab resources.
* Organized technical materials into a more professional reporting structure.
* Prepared the project for a smoother transition into final report drafting and sharing.

### Challenges Faced:

* Documentation required discipline because every screenshot and command needed enough context to remain meaningful later.
* Cost awareness required understanding which services generate charges even when they seem “idle.”

### Lessons Learned and Next Steps:

* A well-executed cloud project should be reproducible, well documented, and cost-conscious.
* In the following week, I would focus on **knowledge sharing** and **report drafting**, using the collected evidence to explain the internship work clearly.
