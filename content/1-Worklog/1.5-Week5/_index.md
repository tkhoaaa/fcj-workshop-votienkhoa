---
title: "Week 5 Worklog"
date: 2026-05-18
weight: 5
chapter: false
pre: "<b>1.5. </b>"
---

### Week 5 Objectives:

* Kick off the capstone project with the team: split the work, agree on the idea, and write the Project Proposal.
* Design the LingoRise system architecture diagram on AWS using a microservices-style decomposition.
* Scaffold the source code repository for both the Next.js frontend and the Node.js Lambda backend.
* Describe the whole infrastructure as code with AWS SAM so every environment can be rebuilt from one template.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Hold the team kickoff meeting, agree on the LingoRise idea, and split the work between frontend, backend, and infrastructure. | 18/05/2026 | 18/05/2026 | Team meeting notes |
| **2** | Write the Project Proposal covering the problem statement, the serverless solution, the AWS service list, the monthly budget, and the main risks. | 19/05/2026 | 19/05/2026 | [AWS Pricing Calculator](https://calculator.aws/) |
| **3** | Draw the system architecture diagram in Draw.io and Figma, from Amplify Hosting through API Gateway and Lambda down to RDS, S3, and SSM. | 20/05/2026 | 20/05/2026 | [AWS Architecture Icons](https://aws.amazon.com/architecture/icons/) |
| **4** | Initialize the repository: Next.js App Router frontend, Node.js 20 Lambda handlers bundled with esbuild, shared database module. | 21/05/2026 | 21/05/2026 | Project repository |
| **5** | Write the first `template.yaml` for AWS SAM, define the SSM parameter naming convention, and create the idempotent SQL migration files. | 22/05/2026 | 22/05/2026 | [AWS SAM Developer Guide](https://docs.aws.amazon.com/serverless-application-model/) |

### Detailed Implementation:

#### 1. Team kickoff and the LingoRise Project Proposal

The team met on day one to agree on the capstone idea and split ownership. We chose **LingoRise**, a Vietnamese-first IELTS and TOEIC preparation platform, because the problem was easy to state and easy to test: practice material is scattered across many sources, feedback after a test is generic, and premium tiers are opaque about what the learner actually gets.

I took the infrastructure and backend scaffolding, one teammate took the Next.js frontend, and one took content and exam data modelling. The Project Proposal then documented the parts a reviewer would ask about:

* the problem statement and the target learner
* the serverless solution and the AWS service list per layer
* an estimated budget of roughly 30 to 40 USD per month, dominated by **Amazon RDS** and **AWS WAF**
* risks such as content quality, exam scoring accuracy, and cost drift on the always-on database

Writing the budget before writing code changed some design decisions. Choosing `db.t4g.micro` with 20 GB instead of a larger instance, and keeping WAF on a single API stage, were both proposal-level decisions rather than later cleanups.

#### 2. System architecture diagram on AWS

I drew the architecture in Draw.io and refined the layout in Figma so the diagram could go straight into the proposal. The request path is a single line that is easy to explain:

**Amplify Hosting** serves the Next.js App Router frontend, the browser calls **Amazon API Gateway** (REST), API Gateway invokes **AWS Lambda**, and Lambda reads and writes **Amazon RDS for PostgreSQL**, **Amazon S3**, and **AWS Systems Manager Parameter Store**. **Amazon Cognito** issues the JWT that Lambda verifies, and **Amazon CloudFront** fronts private assets with signed URLs.

The microservices part of the design is a decomposition inside Lambda rather than a fleet of containers. I grouped handlers by domain: auth, exams, courses, admin, payments, and health. Each group has its own function and its own IAM role, so a payments bug cannot read speaking submissions in S3 and an exam handler cannot touch subscription tables it does not own. Everything runs in **ap-southeast-1** to keep latency low for Vietnamese learners.

#### 3. Repository scaffolding for frontend and backend

I initialized the repository with a clear split between the Next.js App Router frontend and the backend handlers. Backend code is Node.js 20 bundled with esbuild, which keeps the deployment package small and cold starts short. A shared module owns the `pg.Pool` singleton so a warm Lambda container reuses its database connection instead of opening a new one per request.

I also set the first migration convention. Migrations are plain SQL files, written to be safe to run twice, and applied rows are tracked in a `_migrations` table:

```sql
CREATE TABLE IF NOT EXISTS _migrations (
  id          SERIAL PRIMARY KEY,
  filename    TEXT NOT NULL UNIQUE,
  applied_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

That table is small but it decides how the rest of the project ships database changes, so it went in before any feature table.

#### 4. Infrastructure as code with AWS SAM

The last day went to `template.yaml`. **AWS SAM** describes the API, the Lambda functions, the S3 bucket, and the RDS instance, and deploys them as the single `lingorise-dev` CloudFormation stack. Nothing in the account is created by hand from the Console, so the stack can be torn down and rebuilt without losing the configuration.

Secrets never appear in the template. I agreed on the naming convention `/lingorise/<stage>/...` in Parameter Store as SecureString, and the template resolves them at deploy time:

```yaml
Environment:
  Variables:
    DB_HOST: '{{resolve:ssm:/lingorise/${Stage}/db/host}}'
    DB_PASSWORD: '{{resolve:ssm:/lingorise/${Stage}/db/password}}'
    COGNITO_USER_POOL_ID: '{{resolve:ssm:/lingorise/${Stage}/cognito/user-pool-id}}'
```

The `Stage` parameter means the same template can produce a dev stack and, later, a production stack from the same source of truth.

### AWS Service Integration This Week:

* **Amplify Hosting + API Gateway:** the Next.js frontend calls the REST API over HTTPS instead of talking to any backend resource directly.
* **API Gateway + Lambda:** each domain route group maps to its own Node.js 20 function, which is where the microservice boundary lives.
* **Lambda + RDS for PostgreSQL:** handlers share a `pg.Pool` singleton so warm containers reuse connections against `db.t4g.micro`.
* **Lambda + Systems Manager Parameter Store:** database and Cognito configuration is resolved from SecureString parameters at deploy time, never committed.
* **Cognito + Lambda:** the user pool issues the JWT and the handler verifies it before any query runs.
* **AWS SAM + CloudFormation:** the whole architecture diagram is expressed as one template that builds the `lingorise-dev` stack.

### Week 5 Achievements:

* Agreed on the LingoRise idea with the team and produced a signed-off Project Proposal with scope, AWS services, budget, and risks.
* Completed the system architecture diagram in Draw.io and Figma, with a per-domain Lambda decomposition and one IAM role per domain.
* Scaffolded the repository: Next.js App Router frontend, esbuild-bundled Node.js 20 handlers, and a shared database module.
* Wrote the first `template.yaml` for AWS SAM and deployed the initial `lingorise-dev` stack in `ap-southeast-1`.
* Established the `_migrations` table and the idempotent SQL migration workflow.
* Fixed the `/lingorise/<stage>/...` SSM naming convention so no secret is stored in the repository.

### Challenges Faced:

* Deciding how far to split the Lambda functions took discussion. Too many functions meant duplicated bundling and more cold starts, too few meant IAM roles that were wider than the domain they served.
* The budget estimate was hard to pin down because RDS runs continuously while Lambda and API Gateway costs depend on traffic we could not predict yet.
* Getting SAM to deploy cleanly the first time required several template fixes, mostly around resource dependencies and parameter resolution order.

### Lessons Learned and Next Steps:

* Writing the proposal and drawing the architecture before coding removed a lot of later rework, because sizing and cost decisions were already settled.
* Infrastructure as code is worth the extra day at the start. Once `template.yaml` existed, every following change became a reviewable diff instead of a Console click nobody remembers.
* In the following week, I would start coding the core features on top of this scaffolding: user management, **Cognito** authentication and authorization, the data-processing services, and the first APIs exposed through **API Gateway** with CORS configured for the Amplify frontend.
