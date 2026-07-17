---
title: "Blog 2 - Serverless AI on AWS"
date: 2026-07-20
weight: 2
chapter: false
pre: " <b> 2. </b> "
---
# SERVERLESS AI ON AWS — HOW LINGORISE GENERATES AND GRADES EXAMS

The hard part of an AI-powered test-prep platform is not the website — it is producing IELTS/TOEIC content and grading it at scale without owning a GPU fleet. LingoRise solves this entirely with serverless AWS building blocks: AWS Lambda orchestrates the AI work, Amazon Polly synthesizes listening audio, and Amazon S3 stores and delivers everything. This post walks through that AI and content pipeline and the reasoning behind how it is sized and secured.

## The problem: AI at scale without a server to babysit

Generating a reading passage with questions, or grading a free-text Writing answer, means calling a large language model and doing non-trivial post-processing. Those calls are bursty and occasionally slow. Running them on an always-on server would mean paying for idle GPUs and capacity-planning for exam-night spikes. The serverless approach flips this: compute exists only for the duration of the request, and scales out automatically when a whole class hits "start" at once.

## The AI pipeline on Lambda

All AI orchestration lives inside the single Express-on-Lambda API function. When exam content is needed, the handler calls an OpenRouter-compatible chat endpoint, then validates and normalizes the model output before it ever reaches a student.

Two design choices make this work within serverless limits:

* **Pre-generate and cache.** Content is generated ahead of time and cached rather than produced live on the critical path. A student starting an exam reads pre-built, validated questions — the slow, expensive AI call already happened, so the request is fast and cheap. Caching also directly cuts the AI bill, since identical content is never paid for twice.
* **Right-sized compute.** The API function runs at **1024 MB with a 120-second timeout**, deliberately larger than the lightweight 256 MB default, precisely because AI generation and bulk file import are the heaviest things it does. More memory on Lambda also means more CPU, which shortens each generation.

For the grading path, the model output is parsed and scored against a validated answer key, with null-safe handling so a malformed AI response degrades gracefully instead of crashing a student's result.

## Listening audio without servers: Amazon Polly

The Listening section needs spoken audio. In local development we used `edge-tts`, but that spawns a Python process and **cannot run on Lambda**. Amazon Polly solved this cleanly: it is a pure SDK call, no subprocess, no binary to bundle.

* Polly runs in **neural** engine mode with the `Joanna` voice, producing natural speech for the "Nghe thử" (listen preview) and "Xác nhận nhập" (confirm import) buttons.
* Because it is just an API call, the same code path works identically in production without the local CLI dependency.
* The Lambda's IAM role grants exactly one Polly action — `polly:SynthesizeSpeech` — and nothing more.

## Assets and delivery: Amazon S3

Everything the pipeline produces — generated audio, question images, import drafts — lands in a private S3 bucket:

* The bucket **blocks all public access** and uses `BucketOwnerEnforced` ownership, so nothing is ever accidentally world-readable.
* Content is served through **signed URLs with a 15-minute TTL**, letting the browser download assets directly from S3 without the Lambda proxying the bytes, while keeping the objects private.
* A **lifecycle rule** auto-expires `import-drafts/` after 7 days and aborts incomplete multipart uploads after 1 day, so transient working files clean themselves up.

## Cost and resilience

The pipeline is engineered so that the expensive parts happen rarely and the cheap parts happen often:

* **Caching** ensures the AI model is paid for once per unique piece of content, not once per student.
* **Rate limiting** — a per-Lambda in-memory limiter by default, upgradeable to a shared window backed by serverless Redis (e.g. Upstash over TLS) with no code change — keeps abusive traffic from turning into a runaway AI bill.
* **Graceful fallback** — null-safe grading and validated generation mean a bad model response never takes down a student session.

The result is an AI platform where the heavy lifting — content generation, grading, audio synthesis — runs on demand, costs money only when used, and scales itself. No GPU fleet, no audio server, no capacity plan. Just Lambda, Polly, and S3 doing one job each.

...Image...

...Link...

...Guide...
