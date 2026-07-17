---
title: "Blog 1 - LingoRise Serverless Architecture"
date: 2026-07-20
weight: 1
chapter: false
pre: " <b> 1. </b> "
---
# BUILDING LINGORISE ON AWS — A SERVERLESS ARCHITECTURE FOR IELTS/TOEIC PREP

LingoRise is an AI-powered platform for IELTS and TOEIC preparation: it generates practice exams, grades writing and speaking, synthesizes listening audio, and handles payments — all without a single server we have to patch, scale, or wake up for at 2 a.m. The whole backend runs on AWS as one serverless stack defined in a single AWS SAM template. This post walks through the architecture and, for each AWS service, the concrete benefit it buys a small team shipping a real product.

## Why serverless, and why AWS

Before writing a line of infrastructure, we made one decision that shaped everything else: **no servers to manage**. An exam-prep platform has spiky, unpredictable traffic — quiet overnight, then a surge when a cohort sits a mock test the same evening. Provisioning EC2 for the peak means paying for idle capacity 90% of the time; provisioning for the average means falling over on exam day.

AWS's serverless building blocks solve this directly:

* **Scale-to-zero economics** — we pay per request, not per hour. An idle night costs almost nothing.
* **Elastic by default** — Lambda fans out to hundreds of concurrent executions during a traffic spike with no pre-warming or capacity planning.
* **Managed everything** — Cognito handles auth, S3 handles storage, Polly handles speech. We write business logic, not undifferentiated infrastructure.
* **One template, one deploy** — the entire stack (API, auth, storage, WAF, IAM) is declared in `template.yaml` and shipped with a single `sam deploy`.

## The request flow, end to end

A request from a learner's browser travels through a chain of AWS services, each doing one job well:

```
Browser
  → Route 53 (DNS)  → ACM + Amplify (Next.js frontend, TLS)
  → AWS WAF (L7 shield)
  → API Gateway (REST, "Prod" stage, throttled)
  → AWS Lambda (single Express app via serverless-http)
  → PostgreSQL  •  Cognito  •  S3  •  Polly
  → CloudWatch (structured JSON logs)
```

The design choice worth calling out: rather than one Lambda per route, LingoRise runs **a single Lambda** that hosts an Express app and lets the framework route internally. API Gateway simply forwards every path (`/` and `/{proxy+}`) to it. This keeps cold starts rare, deployment trivial, and local development identical to production — the same Express server runs on `localhost:4000` and inside Lambda.

## Key AWS services and the benefit each one buys

* **API Gateway** — the front door. Beyond routing, it enforces stage-level throttling (steady-state 50 req/s, burst 100). Excess traffic is shed with `429` *before* it ever reaches Lambda, so a burst can't fan out into hundreds of concurrent invocations and exhaust the database connection pool.
* **AWS Lambda** — pay-per-invoke compute with zero server ops. The API function is sized at 1024 MB / 120 s to comfortably handle AI generation and file imports, while inheriting a lean 256 MB default for lighter work.
* **Amazon Cognito** — fully managed authentication. A User Pool handles sign-up, email verification codes, password policy, and issues JWTs the Lambda verifies on every protected route. We never store or hash a password ourselves.
* **Amazon S3** — durable asset storage for question images and generated audio. The bucket blocks all public access and serves content through **signed URLs** (15-minute TTL), so assets stay private but remain directly downloadable by the browser. Lifecycle rules auto-expire import drafts after 7 days.
* **Amazon CloudWatch** — structured JSON logs from every invocation, giving us searchable observability without running a logging stack.

## Security in depth

Security isn't a single feature here; it's layered so that a failure in one layer is caught by the next:

* **AWS WAF** sits in front of API Gateway with four rules evaluated in priority order: a per-IP rate-based rule (blocks any IP exceeding 2000 requests / 5 minutes), the AWS-managed Common Rule Set (OWASP-style protections), the IP Reputation List (known bad bots), and the Known Bad Inputs set (Log4j/JNDI, path traversal, host-header injection). Two Common-Rule-Set sub-rules are deliberately set to *Count* instead of *Block* because they would otherwise break legitimate large JSON imports and binary uploads.
* **Stage-level throttling** on API Gateway is the second line of defense against floods.
* **Cognito JWT** verification gates every authenticated endpoint.
* **Least-privilege IAM** — the Lambda's execution role grants only the exact actions it needs: CRUD on its own S3 bucket, a short list of Cognito admin calls scoped to the specific User Pool ARN, and `polly:SynthesizeSpeech`. Nothing more.

## Why this wins for a small team

The payoff of the serverless-on-AWS approach is measured in the problems we *don't* have:

* **No capacity planning** — traffic doubles on exam night; Lambda and API Gateway absorb it automatically.
* **No idle cost** — a quiet week costs a rounding error, because compute bills only when a request arrives.
* **No patch treadmill** — there is no OS, no runtime host, no load balancer to keep patched.
* **One-command deploys** — the whole stack lives in `template.yaml`; a single deploy ships infrastructure and code together, reproducibly.

For a lean team building an AI product, that means engineering time goes into exam quality and grading accuracy — the things learners actually feel — instead of into keeping servers alive.

...Image...

...Link...

...Guide...
