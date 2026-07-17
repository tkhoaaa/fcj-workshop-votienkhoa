---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---
# LingoRise — Vietnamese-First IELTS & TOEIC Preparation Platform
## An AWS Serverless Solution for AI-Assisted Test Preparation and Admin Content Operations

### 1. Executive Summary
LingoRise is a Vietnamese-first web platform that helps learners prepare for IELTS Academic, IELTS General Training, and TOEIC. It pairs an AI-assisted personalization layer with an admin content pipeline that turns existing DOCX, PDF, and scanned-PDF archives into a structured, gradable question bank. The platform is built entirely on AWS Serverless services — API Gateway, AWS Lambda (Node.js 20), Amazon Cognito, Amazon RDS for PostgreSQL, Amazon S3, and AWS Amplify Hosting — so it scales to demand without server management and keeps operational cost proportional to usage. Learners get adaptive daily recommendations, mistake-pattern analytics, Cambridge band scoring, and per-question review; content managers get an import + OCR pipeline with a review queue and audit log; admins get subscription, payment, and user management from a single source of truth.

### 2. Problem Statement
### What's the Problem?
Vietnamese learners preparing for IELTS and TOEIC face three persistent friction points. First, **practice material is scattered** across PDFs, DOCX files, and one-off websites, and importing it into a structured exam engine is manual and error-prone. Second, **feedback is generic** — most platforms surface a band score but never tell the learner *which patterns* they get wrong or *what to study next*. Third, **premium tiers are opaque** — subscriptions are bolted onto products without a clear entitlement model, so admins lose track of who paid for what.

### The Solution
LingoRise addresses all three. An admin content pipeline converts DOCX (text + inline images), PDF (JPEG/JP2 stream extraction), and scanned PDFs (page rasterization → OCR → shell questions) into approved questions, with duplicate detection and a review queue. A learner experience mines exam answers into mistake patterns by skill and section, then feeds a recommendation engine ("Hôm nay nên học gì?" daily plan, weak-skill cards, vocabulary notebook). A subscription and payment model resolves entitlement at request time from `user_subscriptions`, giving admins one source of truth for premium access. The exam engine supports persistent resume, Cambridge band scoring for IELTS, scaled bands for TOEIC, and per-question review with explanations and listening transcripts.

### Benefits and Return on Investment
The platform removes hours of manual question-entry work per exam set through the DOCX/PDF import pipeline, and replaces generic scoring with actionable, per-skill feedback that improves learner retention. Because it runs on AWS Serverless, there are no idle server costs: Lambda, API Gateway, and S3 bill per request, and the only always-on component is a small `db.t4g.micro` RDS instance. Estimated infrastructure cost at dev/early-production scale is roughly **$30–40 USD/month** (see Section 6), dominated by RDS and WAF; compute and storage remain near the free tier at launch traffic. The break-even case rests on time saved in content operations and on premium subscription conversion driven by the personalization layer.

### 3. Solution Architecture
LingoRise uses a fully serverless AWS architecture in **ap-southeast-1**. User traffic hits two surfaces: the Next.js frontend on Amplify Hosting, and the REST API on API Gateway. AWS Lambda is the only compute tier — every backend handler is a stateless Node.js 20 function. Lambda verifies Cognito-issued JWTs locally, reads and writes relational data in RDS PostgreSQL, stores assets (question images, speaking submissions, draft DOCX assets) in S3, and pulls encrypted secrets from SSM Parameter Store. An optional CloudFront distribution serves private S3 assets via signed URLs, and AWS WAF protects the API surface. A separately scheduled OCR worker drains the OCR job queue against RDS and S3.

![LingoRise System Architecture](/images/2-Proposal/architecture.jpg)

### Product Demo

Video walkthrough of the **LingoRise** web application: learner experience, exam flow, and key product screens running on the AWS serverless stack described above.

Open the demo folder on Google Drive (video file inside):

👉 **[Demo video folder — LingoRise](https://drive.google.com/drive/folders/1p3dhFRWf3rbgyNnlVk-R2KkELfWrA1Ce?usp=drive_link)**

<iframe src="https://drive.google.com/embeddedfolderview?id=1p3dhFRWf3rbgyNnlVk-R2KkELfWrA1Ce#list" style="width:100%; height:360px; border:1px solid #ddd; border-radius:8px;"></iframe>

Direct file in the folder:

- [Demo - LingoRise.rar](https://drive.google.com/file/d/1E_IZUdorBI1V4vdOZ9DhxBGWPW_uLa1q/view?usp=sharing) (download / open from Drive)

{{% notice note %}}
If the embedded folder view does not load in your browser, use the Google Drive link above. For in-page playback later, upload the extracted `.mp4` (not `.rar`) with **Anyone with the link** access.
{{% /notice %}}

### AWS Services Used
- **Amazon API Gateway (REST)**: Public HTTPS endpoint fronting every Lambda handler.
- **AWS Lambda (Node.js 20)**: Stateless compute for all backend handlers (auth, exams, courses, admin, payments, health).
- **Amazon Cognito**: End-user authentication; issues JWTs verified by Lambda against the Cognito JWKS.
- **Amazon RDS for PostgreSQL**: Primary data store — users, courses, exams, question bank, payments, OCR jobs, audit log.
- **Amazon S3**: Asset storage for question images, draft DOCX assets, and speaking submissions.
- **Amazon CloudFront**: Optional CDN + signed URLs for private assets (`PriceClass_200`).
- **AWS Systems Manager Parameter Store**: Encrypted secrets (DB URL, payment keys, AI keys).
- **Amazon CloudWatch Logs**: Lambda log destination and structured access logs.
- **AWS Amplify Hosting**: Next.js frontend hosting (SSR + static assets).
- **AWS WAF**: Web ACL protecting the API Gateway stage.
- **AWS IAM**: Lambda execution roles and least-privilege deployer policies.

### Component Design
- **Frontend**: Next.js (App Router) on Amplify Hosting, Vietnamese-first UI, React Query, calls the REST API with a Bearer JWT.
- **API + Auth**: API Gateway routes to Lambda; Lambda verifies the Cognito JWT and resolves the local `users` row for role/active flags.
- **Compute**: One Lambda per handler group; a scheduled OCR worker drains `question_asset_ocr_jobs` with `FOR UPDATE SKIP LOCKED`.
- **Data**: RDS PostgreSQL (UUID PKs, JSONB metadata, idempotent migrations); S3 asset bucket; SSM for secrets.
- **Exam engine**: `POST /exams/start` builds a session from a curated fixed test set or a random pull from the question bank, then resolves asset URLs (public storage URL or 15-minute presigned S3 URL).
- **Payments**: `POST /payments/create` records a pending transaction; provider IPN webhooks verify HMAC before flipping status to paid and activating a subscription.

### 4. Technical Implementation
**Implementation Phases**
The platform is delivered across four phases:
1. **Design & foundation**: Data model (35 idempotent SQL migrations), AWS SAM template, Cognito user pool, RDS, S3, IAM roles.
2. **Core learner + exam engine**: Course catalog, exam catalog with availability states, session start/resume, Cambridge band scoring, per-question review.
3. **Admin content pipeline**: DOCX/PDF/scanned-PDF import, duplicate detection, review queue, question-bank CRUD, per-asset OCR queue + worker, audit log.
4. **Monetization & operations**: Subscription plans, entitlement resolution, payment console (mock checkout + provider IPN), health endpoints, structured logging, security headers, WAF, CloudFront signed assets.

**Technical Requirements**
- **Backend**: Node.js 20 handlers bundled with esbuild into Lambda-ready artifacts, deployed via AWS SAM to the `lingorise-dev` CloudFormation stack. Secrets resolved from SSM at deploy time (`{{resolve:ssm:/lingorise/${Stage}/...}}`).
- **Database**: PostgreSQL 14+ on RDS (`db.t4g.micro`, 20 GB), accessed through a `pg.Pool` singleton; migrations tracked in a `_migrations` table.
- **Auth**: Cognito JWT verification in production (`aws-jwt-verify`), with a dev-token fallback for local development.
- **OCR**: Tesseract.js background worker; jobs enqueued by admin actions and drained serially.
- **Frontend**: Next.js App Router on Amplify Hosting; CI/CD auto-builds on push to the connected branch.

### 5. Timeline & Milestones
**Project Timeline**
- **Phase 1 — Design & foundation** (Weeks 1–2): Migrations, SAM template, Cognito/RDS/S3 provisioning.
- **Phase 2 — Learner + exam engine** (Weeks 3–5): Catalog, sessions, scoring, review, resume.
- **Phase 3 — Admin content pipeline** (Weeks 6–8): Import, OCR queue + worker, review queue, audit log.
- **Phase 4 — Monetization & operations** (Weeks 9–10): Subscriptions, payments, health, security hardening (WAF, CloudFront signed URLs).
- **Post-launch**: Bridge real VNPay/MoMo IPN onto the transaction table, adaptive practice micro-sessions, mobile app on the same API surface.

### 6. Budget Estimation
The platform runs on AWS Serverless, so cost is dominated by the always-on RDS instance and WAF; Lambda, API Gateway, and S3 stay near the free tier at launch traffic in **ap-southeast-1**.

### Infrastructure Costs (estimated, dev / early-production scale)
- **Amazon RDS (`db.t4g.micro`, 20 GB gp2)**: ~$15/month.
- **AWS WAF (Web ACL + rules)**: ~$6–7/month.
- **AWS Amplify Hosting**: ~$1–5/month (build minutes + served requests).
- **Amazon API Gateway (REST)**: ~$1–3/month.
- **AWS Lambda**: ~$0–1/month (near free tier at launch volume).
- **Amazon S3 + CloudFront (`PriceClass_200`)**: ~$1–2/month.
- **Amazon Cognito**: $0 within the free MAU tier.
- **SSM Parameter Store (standard) + CloudWatch Logs**: ~$0.5/month.

**Total: roughly $30–40 USD/month** at dev/early-production scale, scaling with RDS class and traffic. Costs can be estimated precisely with the [AWS Pricing Calculator](https://calculator.aws/).

### 7. Risk Assessment
#### Risk Matrix
- **RDS as a single point of contention**: High impact, low probability.
- **AI provider unavailability (Ollama/OpenRouter)**: Medium impact, medium probability.
- **Payment gateway integration drift**: Medium impact, medium probability.
- **Cost overrun from traffic spikes**: Medium impact, low probability.

#### Mitigation Strategies
- **RDS**: Connection pooling via `pg.Pool`; Multi-AZ standby and read replica available on the production hardening path.
- **AI**: Cloud fallback (OpenRouter) when the local provider is unreachable; pre-generation + cache strategy reduces request-path dependency.
- **Payments**: Authoritative state change happens via HMAC-verified IPN webhook, not the browser return; mock checkout + admin mark-paid supports demo flow without touching live money paths.
- **Cost**: AWS Budgets alerts; serverless billing means idle cost stays near zero.

#### Contingency Plans
- Fall back to public S3 asset mode if CloudFront signing misconfigures.
- Roll back a bad deploy via CloudFormation stack update; migrations are idempotent so re-runs are safe.

### 8. Expected Outcomes
#### Technical Improvements
Manual question entry is replaced by a DOCX/PDF/scanned-PDF import pipeline with OCR and duplicate detection. Generic band scores are replaced by per-skill mistake-pattern analytics and adaptive recommendations. The whole stack scales serverlessly with demand.
#### Long-term Value
A single-source-of-truth entitlement model for premium access, a reusable REST API surface for a future mobile app, and a foundation for adaptive practice micro-sessions and real-time AI-graded speaking.
