---
title: "Week 12 Worklog"
date: 2026-07-06
weight: 12
chapter: false
pre: "<b>1.12. </b>"
---

### Week 12 Objectives:

* Triage every bug found during real user testing by severity and impact, then fix the blockers completely.
* Verify each fix on the exam and payment paths before declaring the final LingoRise build ready.
* Write the closing workshop report and aggregate the attendee and user evaluation data.
* Package the project: final technical documentation, clean source on GitHub, and the full internship dossier for acceptance.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Collect all open bugs from user feedback and **CloudWatch Logs** queries, then classify them as blocker, major, or minor by severity and business impact. | 06/07/2026 | 06/07/2026 | [CloudWatch Logs Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html) |
| **2** | Fix the blocker bugs on the exam session and payment webhook paths, redeploy the backend with **AWS SAM**, and verify each fix end to end. | 07/07/2026 | 07/07/2026 | [AWS SAM Docs](https://docs.aws.amazon.com/serverless-application-model/) |
| **3** | Write the final report for the closing workshop and aggregate the evaluation data collected from attendees and pilot users. | 08/07/2026 | 08/07/2026 | Workshop notes |
| **4** | Finalize technical documentation, push a clean source tree to GitHub, and assemble the whole internship dossier for acceptance. | 09/07/2026 | 09/07/2026 | Project repository |

### Detailed Implementation:

#### 1. Triage the remaining bug backlog by severity and impact

I started from a single list instead of fixing whatever was loudest. The list combined user feedback from Week 11 with error entries pulled from **Amazon CloudWatch Logs** for the Lambda functions in the `lingorise-dev` stack, so every item had either a reproduction step or a request ID behind it.

I then labelled each item:

* **blocker** — the user cannot finish an exam session or cannot complete a payment
* **major** — the feature works but the result is wrong or confusing (band score rounding, review-page transcript ordering)
* **minor** — cosmetic or low-traffic issues (long Vietnamese labels wrapping, admin table column widths)

Grouping errors by function and message made the priority obvious. Two blockers and five majors accounted for almost all of the failed requests, while the minor list was long but harmless.

#### 2. Fix and verify the blockers on the exam and payment paths

The first blocker was a session resume failure: when a candidate reloaded the page mid-test, the handler read the session row before the last answer write had committed, so the resumed session lost one answer. I moved the answer write and the session cursor update into one transaction against **Amazon RDS for PostgreSQL** so both succeed or neither does.

The second blocker was on the payment **IPN webhook**. A provider retry produced a duplicate row in `user_subscriptions`, which then confused entitlement resolution. I made the webhook idempotent on the provider transaction id after HMAC verification.

```sql
ALTER TABLE user_subscriptions
  ADD CONSTRAINT user_subscriptions_provider_txn_key UNIQUE (provider, provider_txn_id);
```

The migration is idempotent and tracked in the `_migrations` table like every earlier change. After deploying the fixed handlers with **AWS SAM**, I replayed both scenarios against the API Gateway stage and confirmed in CloudWatch that the previous error signatures no longer appeared. The minor items stayed open on purpose, recorded in the issue list with a short note on why they were deferred.

#### 3. Write the closing workshop report and aggregate the evaluation data

I wrote the final report for the closing FCJ workshop, "Deploying LingoRise — A Serverless English-Learning Platform on AWS". The report follows the same order as the live walkthrough:

* store secrets as SecureString parameters in **AWS Systems Manager Parameter Store**
* create the **Amazon RDS for PostgreSQL** instance and run the migrations
* build and deploy the **AWS Lambda** and **Amazon API Gateway** backend with **AWS SAM**
* connect the **Next.js** frontend on **AWS Amplify Hosting**
* upload question images and audio to **Amazon S3** and serve them through **Amazon CloudFront**
* run the smoke test, then follow the cleanup steps to avoid leftover cost

Alongside the walkthrough I aggregated the evaluation data: the attendee feedback on the workshop itself and the usability scores collected from pilot users in Week 11. Reporting both together made the numbers more useful, because the same friction points appeared in the workshop questions and in the user comments.

#### 4. Package the project and the internship dossier for acceptance

The last step was packaging. I finished the technical documentation — architecture overview, environment variables and their SSM parameter paths, migration procedure, and the deploy and rollback runbook — then cleaned the repository before the final push: no committed `.env`, no build artifacts, README updated with the real deploy sequence.

```bash
sam build && sam deploy --stack-name lingorise-dev --no-confirm-changeset
aws cloudformation describe-stacks --stack-name lingorise-dev \
  --query "Stacks[0].StackStatus" --region ap-southeast-1
```

I then assembled the internship dossier itself: the twelve worklog weeks, the LingoRise proposal, the technical blogs, the event write-ups, the workshop report, and the self-evaluation, all published on this Hugo site so a reviewer can read the whole internship in one place.

### AWS Service Integration This Week:

* **CloudWatch Logs + Lambda:** Log queries turned scattered user complaints into a ranked bug list with request IDs attached.
* **Lambda + RDS for PostgreSQL:** The session resume fix moved the answer write and cursor update into one transaction so a reload can no longer drop an answer.
* **API Gateway + Lambda + RDS:** The idempotent payment webhook made provider retries safe, keeping `user_subscriptions` clean for entitlement checks.
* **AWS SAM + CloudFormation:** Every fix reached the `lingorise-dev` stack through the same build and deploy path, so the final build is reproducible.
* **SSM Parameter Store + SAM:** Documenting the parameter paths meant the deploy runbook needs no secret values in the repository.
* **Amplify Hosting + S3 + CloudFront:** The frontend and its media assets were verified together during the final smoke test.

### Week 12 Achievements:

* Produced a triaged bug list where severity was tied to real user impact instead of report order.
* Fixed and verified both blockers on the exam and payment paths, with the schema change tracked as an idempotent migration.
* Deferred the minor items deliberately and recorded the reason, instead of leaving them undocumented.
* Completed the closing workshop report with the full deployment walkthrough and the aggregated evaluation data.
* Delivered final technical documentation and a clean source tree on GitHub.
* Assembled the complete internship dossier — worklog, proposal, blogs, events, workshop, self-evaluation — ready for acceptance.

### Challenges Faced:

* Deciding what not to fix was harder than fixing. Accepting known minor bugs in a final build required a written justification rather than a feeling.
* The duplicate subscription bug only appeared under provider retries, so reproducing it meant replaying webhook payloads instead of clicking through the UI.
* Writing documentation that someone else can deploy from forced me to find the undocumented steps I had been carrying in my head.

### Lessons Learned and Next Steps:

* Across twelve weeks the path was continuous: AWS fundamentals and account security, then networking, compute and storage, then DevOps and IaC, and finally designing, building, and shipping LingoRise as a serverless application. Each stage only made sense because of the one before it.
* Shipping is a separate skill from building. Triage, verification, documentation, and packaging took a full week and were what turned working code into a deliverable product.
* My next learning direction is concrete: prepare for the AWS Solutions Architect certification, go deeper on Infrastructure as Code, and harden LingoRise for production with Multi-AZ RDS, a read replica for reporting queries, and tighter CloudWatch alarms.
