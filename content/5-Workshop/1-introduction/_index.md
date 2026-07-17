---
title : "Introduction & Architecture"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

#### Overview

LingoRise is an IELTS and English-learning platform: learners register, take timed mock exams (Listening, Reading, Writing, Speaking), work through a question bank, buy courses, and get AI-assisted grading and feedback. The backend is a serverless Node.js stack; the frontend is a Next.js app.

In this workshop you will deploy that whole platform onto AWS in the `ap-southeast-1` (Singapore) region, using the `dev` stage. By the end you will have a running API Gateway endpoint, Lambda functions, a Cognito user pool, an RDS PostgreSQL database, an S3 asset bucket, and encrypted configuration in SSM Parameter Store — all provisioned through a single CloudFormation stack named `lingorise-dev`. We work bottom-up: understand the architecture first, then prepare tools and credentials, then provision each piece, then wire the frontend on top.

This chapter is the map. It shows every AWS service in scope, how the pieces talk to each other, and what you will have running when you finish.

#### Architecture

LingoRise runs as a serverless topology. User traffic hits two surfaces: the **frontend** (Amplify Hosting, serving the Next.js app) and the **API** (API Gateway, fronting every backend handler). Lambda is the only compute tier — there are no long-running servers to patch. Lambda talks to RDS for relational data, S3 for assets, SSM for secrets, and reaches out to external partners for AI generation and payments. CloudWatch collects logs from every function.

![LingoRise high-level architecture](/images/Workshop-LingoRise/1-introduction/architecture.jpg)

The full service inventory below lists every AWS service in scope and what it does in LingoRise.

| Service | Purpose |
|---|---|
| API Gateway (REST) | Public HTTPS endpoint for every Lambda handler |
| AWS Lambda (Node.js 20) | Stateless compute for all backend handlers |
| Amazon Cognito User Pool | End-user authentication; issues JWTs verified by Lambda |
| Amazon RDS for PostgreSQL | Primary data store (users, courses, exams, question bank, payments, OCR jobs, audit log) |
| Amazon S3 | Asset storage (question images, draft DOCX assets, speaking submissions) |
| Amazon CloudFront | Optional CDN + signed URLs for private assets |
| AWS Systems Manager Parameter Store | Encrypted secrets (DB URL, payment keys, AI keys) |
| Amazon CloudWatch Logs | Lambda log destination |
| AWS Amplify Hosting | Next.js frontend hosting (SSR + static assets) |
| Amazon SES | Transactional email (optional, deferred) |
| AWS IAM | Lambda execution roles, deployer policies |

A few external, non-AWS dependencies still sit in the data path and must be reachable from Lambda:

- **Ollama / OpenRouter** — AI generation and chat (switch to OpenRouter for a cloud-reachable endpoint)
- **PayOS** — payment redirects and IPN callbacks
- **YouTube Data API, NewsAPI, GNews, Reddit** — content discovery handlers

{{% notice note %}}
For the `dev` stage you can deploy Lambda **outside** a VPC and use a publicly reachable RDS instance locked down with a tight security group. The private-subnet VPC layout (Lambda ENIs across two AZs, NAT for outbound, VPC endpoints for S3 and SSM) is a production-hardening path and is optional for this workshop.
{{% /notice %}}

#### Request flows

Three flows show how the pieces cooperate. Read them as step lists; the same flows are drawn as sequence diagrams in the source architecture doc.

![Auth login flow](/images/Workshop-LingoRise/1-introduction/auth-flow.png)

**1. Login and authenticated requests (Cognito)**

Cognito issues the JWT; every protected Lambda then verifies that token locally against the Cognito JWKS and resolves the matching `users` row to attach the caller's role and active flag.

1. The user submits the login form in the Next.js app.
2. The frontend calls `POST /auth/login` through API Gateway, which invokes the `auth/login` Lambda.
3. That Lambda calls Cognito `AdminInitiateAuth` (USER_PASSWORD_AUTH) and receives an idToken, accessToken, and refreshToken.
4. It looks up the local user with `SELECT users WHERE email = $1`, checks `is_active`, and attaches the role.
5. It returns `200 { token, user }`; the frontend stores the token.
6. On any later authenticated call (for example `GET /auth/me` with a Bearer token), the Lambda verifies the JWT against the Cognito JWKS, re-reads the user row, and responds.

**2. Exam start (fixed-set vs random + asset URLs)**

`POST /exams/start` builds a session, picks questions in one of two modes, then resolves asset URLs based on `ASSET_URL_MODE`.

1. The user calls `POST /exams/start { templateId, mode }`.
2. The Lambda opens a transaction and selects the template and its sections.
3. In **fixed-set** mode it pulls questions from `test_set_questions`; in **random** mode it selects from `question_bank` with a filter and `ORDER BY RANDOM()`.
4. It loads the matching `question_assets`, inserts a row into `exam_sessions` with `status='in_progress'`, and commits.
5. For each asset: in **public** mode it uses the stored `storage_url`; in **signed** mode it mints a 15-minute presigned S3 `GetObject` URL.
6. It returns `200 { sessionId, sections, questions, urls }`.

**3. Payment callback (PayOS)**

The browser redirect (return URL) is for UX only. The authoritative state change happens through the server-to-server IPN webhook, which Lambda verifies by HMAC before touching the `payments` row.

1. The user clicks "Mua khoá học"; the frontend calls `POST /payments/create`.
2. The `payments/create` Lambda inserts a `payments` row with `status='pending'`, builds an HMAC-SHA512 signed payload, and returns a `redirectUrl`.
3. The frontend redirects the browser to the payment gateway; the user completes payment.
4. **Source of truth (IPN):** the gateway calls `POST /payments/{provider}-callback`. The Lambda verifies the HMAC signature, updates the payment to `status='paid'`, and activates the enrollment/subscription.
5. **UX only (return):** the gateway redirects the browser back to `FRONTEND_URL/return`; a `GET /payments/return` call reads the current payment status and reports `paid: true | false`.

#### What you'll deploy

By the end of the workshop you will have provisioned, through the `lingorise-dev` CloudFormation stack in `ap-southeast-1`:

- An **API Gateway** REST API exposing every backend route over HTTPS.
- A set of **Lambda functions** (Node.js 20) for auth, exams, question bank, payments, OCR, and content handlers.
- A **Cognito User Pool** issuing and validating end-user JWTs.
- An **RDS PostgreSQL** instance (`lingorise-dev-db`, database `lingorise`) with the migrated schema.
- An **S3 asset bucket** (`lingorise-assets-dev-<accountId>`) for question images and submissions.
- **SSM Parameter Store** entries under `/lingorise/dev/` holding the DB URL, payment keys, and AI keys.
- **CloudWatch Logs** groups for every function, plus the **IAM** execution roles that tie it all together.
- The **Next.js frontend** on **Amplify Hosting**, wired to the API Gateway endpoint.

{{% notice info %}}
Next up: **[2. Prerequisites]** — install the AWS CLI, SAM CLI, and Node.js 20, create the `lingorise-dev` CLI profile, and confirm your account can create resources in `ap-southeast-1` before you run your first `sam deploy`.
{{% /notice %}}
