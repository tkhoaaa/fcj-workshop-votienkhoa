---
title: "Blogs Posted"
date: 2026-07-20
weight: 3
chapter: false
pre: " <b> 3. </b> "
---
This section lists and introduces the blogs about how **LingoRise** — an AI-powered IELTS/TOEIC preparation platform — is built and operated on AWS.

###  [Blog 1 - Building LingoRise on AWS: A Serverless Architecture for IELTS/TOEIC Prep](3.1-Blog1/)
This blog walks through the end-to-end serverless architecture behind LingoRise: Route53 + ACM + Amplify serving the Next.js frontend, API Gateway forwarding to a single Express-on-Lambda backend, Cognito for managed authentication, S3 for asset storage, and AWS WAF as an L7 shield. It explains the concrete benefit each AWS service buys a small team — pay-per-invoke compute, scale-to-zero cost, managed auth, and defense in depth — and why serverless wins for spiky, exam-day traffic.

###  [Blog 2 - Serverless AI on AWS: How LingoRise Generates & Grades Exams](3.2-Blog2/)
This blog covers the AI content pipeline: how Lambda orchestrates AI calls with a pre-generate-and-cache strategy, why the function is sized at 1024 MB / 120 s, how Amazon Polly neural TTS synthesizes listening-section audio without a server, and how S3 signed URLs plus lifecycle rules deliver and expire generated assets. It closes on cost and resilience — caching to cut AI spend, rate limiting, and graceful fallback.
