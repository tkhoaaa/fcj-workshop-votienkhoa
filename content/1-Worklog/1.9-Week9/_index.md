---
title: "Week 9 Worklog"
date: 2026-06-15
weight: 9
chapter: false
pre: "<b>1.9. </b>"
---

### Week 9 Objectives:

* Run functional testing over the LingoRise learner flows and the admin content flows.
* Run integration testing across the real service boundaries: **API Gateway**, **AWS Lambda**, **Amazon RDS**, **Amazon S3**, **AWS Systems Manager**, and **Amazon Cognito**.
* Fix the outstanding bugs found during testing instead of only recording them.
* Optimize the AI and OCR processing pipeline for extraction speed, accuracy, and resilience.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Write the functional test plan for the learner flow (sign-in, catalog, start exam, answer, submit, score, review) and execute it against the `lingorise-dev` API stage. | 15/06/2026 | 15/06/2026 | Project repository |
| **2** | Functional-test the admin content flows: DOCX and PDF import, duplicate detection, review queue, publish, and audit log. | 16/06/2026 | 16/06/2026 | Self-study notes |
| **3** | Integration-test the service boundaries end to end and use **Amazon CloudWatch Logs** to trace failures that happened inside Lambda. | 17/06/2026 | 17/06/2026 | [Amazon CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html) |
| **4** | Fix the bugs found in the previous three days and re-run the affected test cases after each `sam deploy`. | 18/06/2026 | 18/06/2026 | [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/using-sam-cli.html) |
| **5** | Tune the OCR job queue and Tesseract.js accuracy, then verify the AI provider fallback path from the local provider to OpenRouter. | 19/06/2026 | 19/06/2026 | [AWS Lambda Docs](https://docs.aws.amazon.com/lambda/) |

### Detailed Implementation:

#### 1. Functional testing of the learner and admin flows

I tested the learner path as a real user would walk it: sign in through **Amazon Cognito**, browse the exam catalog, start an IELTS session, answer questions, submit, receive a band score, and open the per-question review with explanations and listening transcripts. Session resume was the case I retested most, because a learner who closes the tab must come back to the same `exam_sessions` row and not to a fresh attempt.

On the admin side I imported a real DOCX file and a scanned PDF, then checked that duplicate detection flagged a question I had deliberately imported twice, that the review queue held the draft instead of publishing it directly, and that every publish action landed in the audit log table.

The important habit here was writing the expected result before running the case. Without that, a wrong band score looks plausible and passes unnoticed.

#### 2. Integration testing across the real service boundaries

Local tests only proved the handler logic. The boundaries had to be tested on deployed infrastructure, so I ran the cases against the `lingorise-dev` stack:

* **API Gateway -> Lambda:** request mapping, payload size, and the CORS preflight for the Amplify origin.
* **Lambda -> Amazon RDS:** `pg.Pool` reuse across warm invocations, and behavior on a cold start when the pool is created fresh.
* **Lambda -> Amazon S3:** presigned URL generation, expiry, and upload from the browser directly to the bucket.
* **Lambda -> AWS Systems Manager:** SecureString parameters resolved at deploy time through `{{resolve:ssm:/lingorise/dev/...}}`.
* **Amazon Cognito -> Lambda:** JWT verification with `aws-jwt-verify`, including an expired token and a token from the wrong user pool.

Two things could not be reproduced locally at all: the presigned URL signature under the deployed IAM role, and JWT verification against the real JWKS endpoint. Both only failed on AWS, and both were only diagnosable from **Amazon CloudWatch Logs**. I stopped guessing at Lambda failures and started reading the log stream first.

#### 3. Bugs found and fixed

Three bugs were representative of the whole week:

* Submitting an already-submitted session created a second score row. The fix was a conditional update on session status so the write only applies when the session is still open.
* TOEIC scaled band conversion rounded at the wrong boundary, so a raw score exactly on the edge dropped one band. The fix was in the conversion table lookup, and I added the boundary values as explicit test cases.
* A cold-start invocation occasionally failed with a connection error because the RDS pool was created before the SSM-resolved credentials were read. Ordering the initialization removed it.

```bash
sam build && sam deploy --stack-name lingorise-dev --no-confirm-changeset
aws logs tail /aws/lambda/lingorise-dev-api --since 10m --follow
```

#### 4. Optimizing the AI and OCR pipeline

The OCR pipeline is a per-asset job queue in PostgreSQL, drained by a scheduled worker that claims rows with `FOR UPDATE SKIP LOCKED`. Testing showed the worker was claiming one job per run, so a 30-page scanned PDF took far too long. I raised the claim batch size and let the worker keep draining while time remained in the invocation.

```sql
SELECT id, asset_key FROM ocr_jobs
WHERE status = 'pending'
ORDER BY created_at
FOR UPDATE SKIP LOCKED
LIMIT 5;
```

For accuracy I rasterized pages at a higher DPI before passing them to **Tesseract.js**, loaded the Vietnamese and English language data together, and added grayscale plus contrast preprocessing. That reduced the amount of manual correction needed in the review queue noticeably on scanned material.

For resilience I tested the AI provider fallback: when the local provider is unreachable, the request falls through to OpenRouter instead of returning an error. I forced the failure by pointing the provider at a dead endpoint and confirmed the request still completed, with the fallback recorded in the log so the switch is visible after the fact.

### AWS Service Integration This Week:

* **API Gateway + AWS Lambda:** Every functional test case entered through the REST stage so request mapping and CORS were exercised, not bypassed.
* **AWS Lambda + Amazon RDS:** Integration tests confirmed the `pg.Pool` singleton survives warm invocations and initializes correctly on a cold start.
* **AWS Lambda + Amazon S3:** Presigned URL upload and download were tested against the deployed execution role, which is the only place the signature is real.
* **Amazon Cognito + AWS Lambda:** JWT verification was tested with valid, expired, and wrong-pool tokens to confirm the authorizer rejects rather than defaults to allow.
* **AWS Lambda + AWS Systems Manager:** Parameter Store SecureStrings resolved at deploy time, which surfaced the initialization-order bug behind the cold-start failures.
* **Amazon CloudWatch Logs + AWS SAM:** Each `sam deploy` was followed by a log tail, which turned an opaque HTTP 500 into a specific stack trace.

### Week 9 Achievements:

* Executed a written functional test plan across the learner flows and the admin content pipeline, including exam session resume.
* Completed integration testing on the deployed `lingorise-dev` stack across all five service boundaries.
* Fixed the duplicate score row, the TOEIC band boundary rounding, and the cold-start pool initialization bug, then re-verified each with the failing case.
* Increased OCR throughput by batching job claims in the `FOR UPDATE SKIP LOCKED` worker instead of processing one asset per run.
* Improved OCR accuracy through higher page rasterization quality, combined Vietnamese and English language data, and image preprocessing.
* Verified the AI provider fallback path so a provider outage degrades the response instead of breaking the request.

### Challenges Faced:

* Several defects existed only on AWS, so the local test suite gave false confidence and CloudWatch Logs became the real debugging surface.
* OCR accuracy tuning has no single correct setting; higher rasterization DPI improved recognition but cost processing time, so the batch size had to be balanced against the worker timeout.
* Reproducing an expired Cognito token and a provider outage on demand required deliberately breaking configuration, which is slower than it sounds.

### Lessons Learned and Next Steps:

* A test is only useful if the expected result is written before the run. Plausible-looking wrong output is the failure mode that survives testing.
* Boundaries between services are where the real bugs live. Handler unit tests passed while presigned URLs and JWT verification were both broken.
* Next week I move to handover and packaging: publishing the source code on GitHub, cleaning up and capping AWS resources for cost, and writing the final internship report in English.
