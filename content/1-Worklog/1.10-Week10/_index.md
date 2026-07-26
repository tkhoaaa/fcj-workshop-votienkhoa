---
title: "Week 10 Worklog"
date: 2026-06-22
weight: 10
chapter: false
pre: "<b>1.10. </b>"
---

### Week 10 Objectives:

* Package the whole LingoRise source tree on GitHub so another engineer can redeploy the stack without asking me anything.
* Clean up leftover AWS lab resources from earlier weeks and put spending limits in place for what remains.
* Confirm the real cost drivers of the running environment using **AWS Cost Explorer** and the bill by service.
* Complete the Final Internship Report in English, covering AWS service survey, architecture design, implementation, test results, and lessons learned.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Clean the repository for handover: README with deploy steps, `.env.example`, migration documentation, and a scan to confirm no secrets were committed. | 22/06/2026 | 22/06/2026 | Project repository |
| **2** | Write the redeploy runbook for the `lingorise-dev` SAM stack and tag a release on GitHub. | 23/06/2026 | 23/06/2026 | [AWS SAM Docs](https://docs.aws.amazon.com/serverless-application-model/) |
| **3** | Review Cost Explorer and the bill by service, then stop or delete unused lab resources from earlier weeks. | 24/06/2026 | 24/06/2026 | [AWS Cost Explorer](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html) |
| **4** | Configure AWS Budgets alerts for the account and confirm RDS and WAF as the dominant fixed cost. | 25/06/2026 | 25/06/2026 | [AWS Budgets Docs](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html) |
| **5** | Write the Final Internship Report in English and standardise the Hugo report site content to match it. | 26/06/2026 | 26/06/2026 | [FCJ Workshop Sample](https://workshop-sample.fcjuni.com/) |

### Detailed Implementation:

#### 1. Package the repository for handover

I treated the repository as the actual deliverable, not just a place where my code lived. The README now documents the layout of the Next.js frontend, the **AWS Lambda** handlers, and the **AWS SAM** template, plus the exact command sequence to build and deploy to a fresh account.

I added an `.env.example` listing every variable the app expects, with values replaced by placeholders. The real values stay in **AWS Systems Manager Parameter Store** as SecureString and are resolved at deploy time through `{{resolve:ssm:/lingorise/${Stage}/...}}`, so nothing sensitive needs to sit in the repository at all. I then grepped the history for key patterns and confirmed no access keys, database passwords, or webhook secrets had ever been committed.

The migration story also needed writing down. Migrations are idempotent SQL files tracked in the `_migrations` table, so I documented the ordering rule and the fact that re-running the set on an existing **Amazon RDS for PostgreSQL** database is safe.

#### 2. Write the redeploy runbook and tag a release

The runbook covers the path from an empty account to a working environment: create the Parameter Store entries, build the Lambda bundles with esbuild, deploy the stack, then run migrations against the database.

```bash
sam build && sam deploy --stack-name lingorise-dev \
  --parameter-overrides Stage=dev --capabilities CAPABILITY_IAM
aws cloudformation describe-stacks --stack-name lingorise-dev \
  --query "Stacks[0].Outputs" --region ap-southeast-1
```

The stack outputs give the **Amazon API Gateway** invoke URL and the **Amazon Cognito** user pool ID, which are the two values the frontend build needs on **AWS Amplify Hosting**. Documenting that dependency chain was the point: the frontend cannot be configured until CloudFormation has finished, so the runbook orders the steps accordingly. I tagged the commit as a release so the report references a fixed state of the code.

#### 3. Clean up AWS resources and control cost

I opened **AWS Cost Explorer** grouped by service for the previous three months and worked down the list. Several resources from the networking and compute weeks were still alive: spare **Amazon EC2** instances, unattached **Amazon EBS** volumes, old AMIs with their backing snapshots, and a **NAT Gateway** that had been charging per hour long after the lab that needed it was over. I terminated the instances, deleted the volumes and snapshots, deregistered the stale AMIs, and removed the NAT Gateway.

What remained showed the real shape of the bill. **Amazon RDS** at `db.t4g.micro` and **AWS WAF** on the API Gateway stage are the dominant fixed cost, because both charge on time and rules rather than on traffic. The serverless parts — Lambda, API Gateway, **Amazon S3** — barely registered at development traffic levels. That contrast is worth recording in the report: choosing serverless compute moved almost all of the spend into the two always-on components.

I then set **AWS Budgets** alerts with thresholds at 50, 80, and 100 percent of the monthly figure, notifying by email so any unexpected growth surfaces early instead of at the end of the billing period.

#### 4. Complete the Final Internship Report in English

I wrote the report in the order the work actually happened: AWS service survey, architecture design, implementation, test results, then lessons learned. The architecture section describes LingoRise as a single request path rather than a list of services — a Vietnamese-first Next.js client on Amplify Hosting calls API Gateway, WAF filters the request, Lambda verifies the Cognito JWT with `aws-jwt-verify`, resolves entitlement from `user_subscriptions`, and reads or writes RDS through a `pg.Pool` singleton, with S3 holding question images and speaking submissions.

The test results section reuses the evidence from Week 9: the exam engine's session resume behaviour, Cambridge band scoring for IELTS and scaled bands for TOEIC, and the OCR job queue drained by the scheduled worker using `FOR UPDATE SKIP LOCKED`. I also standardised the Hugo site content so the worklog pages and the printed report tell the same story with the same terminology in both English and Vietnamese.

### AWS Service Integration This Week:

* **AWS SAM + CloudFormation:** The `lingorise-dev` stack became the single reproducible unit of deployment described in the runbook.
* **CloudFormation Outputs + AWS Amplify Hosting:** The API Gateway URL and Cognito pool ID flow from stack outputs into the frontend build configuration.
* **Parameter Store + AWS SAM:** Secrets stay outside the repository and are resolved at deploy time, which is what made a public handover safe.
* **Cost Explorer + Amazon RDS/AWS WAF:** Cost analysis identified the two always-on services as the dominant fixed spend of an otherwise serverless design.
* **AWS Budgets + Billing:** Threshold alerts now watch the account so idle resources cannot quietly accumulate charges.
* **Amazon EC2 + Amazon EBS + NAT Gateway:** Deleting compute also required deleting its attached storage, snapshots, and network path to actually stop the billing.

### Week 10 Achievements:

* Handed over a packaged repository with deploy steps, `.env.example`, documented migrations, and a tagged release.
* Produced a redeploy runbook that takes an empty account to a working `lingorise-dev` stack.
* Removed unused EC2 instances, unattached EBS volumes, stale AMIs and snapshots, and the NAT Gateway.
* Confirmed RDS and WAF as the dominant fixed cost and documented why the serverless components contribute so little.
* Configured AWS Budgets alerts at three thresholds on the account.
* Completed the Final Internship Report in English and aligned the Hugo site content with it.

### Challenges Faced:

* Deleting resources safely took care, because terminating an instance does not remove its EBS volumes or the snapshots behind its AMIs, and those keep charging silently.
* Writing a runbook exposed steps I had only ever done by hand, so several implicit ordering assumptions had to be discovered and written down before the sequence worked from scratch.
* Compressing ten weeks of work into one English report meant deciding what to leave out without losing the architecture narrative.

### Lessons Learned and Next Steps:

* A cloud project is only handed over when someone else can redeploy it from the repository alone; anything held in my head is an undocumented dependency.
* Cost discipline is an architecture concern. Going serverless does not reduce spend by itself when a managed database and a WAF stay on around the clock.
* In the following week, I would analyse user experience metrics and error rates from real trials, then refine the **Next.js**/React frontend and fix the unexpected issues that feedback surfaces.
