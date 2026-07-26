---
title: "Week 11 Worklog"
date: 2026-06-29
weight: 11
chapter: false
pre: "<b>1.11. </b>"
---
### Week 11 Objectives:

* Consolidate user experience metrics and the error rate observed during the real trial of LingoRise.
* Separate product friction from technical faults so each finding had a clear owner.
* Refine the **Next.js** frontend based on tester feedback, focusing on exam navigation, loading states, and Vietnamese wording.
* Fix the unexpected defects that surfaced during the trial and verify each fix against the same metric.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Collect trial data from the tester group: task completion rate for starting and finishing a practice exam, time to first question, and drop-off points. | 29/06/2026 | 29/06/2026 | Self-study notes |
| **2** | Pull the technical side of the picture from **Amazon CloudWatch**: API Gateway 4xx/5xx counts, Lambda p95 duration, and error entries in the log groups. | 30/06/2026 | 30/06/2026 | [Amazon CloudWatch Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html) |
| **3** | Analyse the combined data set and split findings into product friction and technical faults, then rank them by how many testers each one blocked. | 01/07/2026 | 01/07/2026 | Project repository |
| **4** | Refine the React UI: clearer exam navigation and answer sheet, real loading and error states, mobile layout, and reworked Vietnamese copy. | 02/07/2026 | 02/07/2026 | [Next.js App Router](https://nextjs.org/docs/app) |
| **5** | Fix the unexpected defects found during the trial and re-measure the error rate after deploying to the `lingorise-dev` stack. | 03/07/2026 | 03/07/2026 | [AWS SAM Docs](https://docs.aws.amazon.com/serverless-application-model/) |

### Detailed Implementation:

#### 1. Collect experience metrics from the real trial

The trial group ran full practice exams on their own devices, so the data was closer to real usage than my own testing had been. For each tester I recorded whether they managed to start an exam, whether they finished it, how long the app took to show the first question, and where they stopped when they did not finish.

The numbers that mattered most:

* task completion rate for the start-exam flow and for the finish-and-submit flow
* time to first question, measured from the click on "Bắt đầu" to the rendered question
* the point in the session where a tester abandoned the attempt
* free-text feedback in Vietnamese, kept verbatim so I would not smooth over the wording

That gave me the product-side view. The technical view had to come from AWS.

#### 2. Read the error rate out of CloudWatch

**Amazon CloudWatch** already held the other half of the story. I looked at the `4XXError` and `5XXError` metrics on the **Amazon API Gateway** stage, the p95 `Duration` of each **AWS Lambda** function, and the raw error lines in the log groups. A Logs Insights query over the exam function was the fastest way to see which routes were actually failing:

```sql
fields @timestamp, level, route, err.name, err.message
| filter level = "error"
| stats count(*) as errors by route, err.name
| sort errors desc
| limit 20
```

The measured API error rate was low in absolute terms, but it was not evenly spread. Almost all of it sat on two routes: the exam session start and the asset fetch for listening audio. Lambda p95 confirmed the same thing, with the session-start function well above the others, partly cold start and partly a slow query.

#### 3. Split product friction from technical faults

Combining both data sets made the analysis straightforward. Product friction showed up as testers hesitating or taking a wrong path with no error in the logs. Technical faults showed up as a log entry or a latency spike that matched the moment a tester gave up.

The ranked list came out as:

* **Product friction:** the answer sheet did not make it obvious which questions were still unanswered; the wording "Nộp bài" was ambiguous next to "Lưu tạm"; the reading passage and the questions competed for space on mobile; the loading state was a bare spinner with no context.
* **Technical faults:** the session-start query scanned `exam_questions` without a usable index; a **React Query** cache key omitted the session id, so resuming an exam sometimes rendered the previous attempt; the presigned URL for listening audio expired mid-exam on longer sections; the entitlement lookup against `user_subscriptions` ran on every request in a page instead of once.

Two testers reported the same "audio stopped working halfway" symptom that I had never reproduced, and the expiring URL explained it.

#### 4. Refine the frontend and fix the defects

On the UI side I reworked the answer sheet so unanswered questions are visually distinct and reachable in one tap, renamed the two save actions to unambiguous Vietnamese, gave the exam shell a skeleton state that shows what is loading, added a retry affordance to the error state instead of a dead-end message, and stacked the passage above the question list below the mobile breakpoint. All of it deployed through **AWS Amplify Hosting** on push, so each round of feedback was live within minutes.

On the technical side each fault got a named fix:

* added a composite index so the session-start query stopped scanning, applied as an idempotent migration tracked in `_migrations`:

```sql
CREATE INDEX IF NOT EXISTS idx_exam_questions_exam_order
  ON exam_questions (exam_id, question_order);
```

* corrected the React Query cache key to include the session id, which removed the stale-attempt rendering
* raised the TTL on the **Amazon S3** presigned URLs for listening assets so a URL outlasts the longest section
* memoized the entitlement resolution per request so one page load hits `user_subscriptions` once, not repeatedly

After redeploying the backend with **AWS SAM** to the `lingorise-dev` stack, I re-ran the same measurements. The error rate on both problem routes dropped to zero across the re-test, and p95 on session start fell far enough that the time-to-first-question complaint disappeared from the feedback.

### AWS Service Integration This Week:

* **CloudWatch Logs + Lambda:** structured error logs from each function were queried with Logs Insights to attribute failures to specific routes.
* **CloudWatch Metrics + API Gateway:** 4xx/5xx counts on the stage gave the API error rate that the tester reports were compared against.
* **RDS for PostgreSQL + Lambda:** the index migration cut the session-start query time, which showed up directly as lower Lambda p95.
* **S3 + CloudFront:** the presigned URL TTL for listening audio was raised so asset access survived a full exam section.
* **Amplify Hosting + Next.js:** each UI refinement was pushed and built automatically, keeping the feedback loop with testers short.
* **SAM + CloudFormation:** the backend fixes went out as one deployment to `lingorise-dev` so before-and-after metrics stayed comparable.

### Week 11 Achievements:

* Produced a single measured view of the trial, combining completion rate and tester feedback with API error rate and p95 latency.
* Attributed every complaint to either product friction or a technical fault instead of treating them all as bugs.
* Shipped a clearer exam interface: distinct unanswered questions, unambiguous Vietnamese actions, real loading and error states, and a usable mobile layout.
* Fixed four concrete technical defects, including the intermittent audio failure that only the metrics explained.
* Confirmed the fixes by re-measuring the same metrics rather than by assuming the change worked.

### Challenges Faced:

* The most damaging defect was intermittent and never reproduced on my machine, so it only became visible when tester reports were lined up against CloudWatch timestamps.
* Feedback arrived as opinions about the interface, and turning that into a ranked list with technical causes took more time than writing the fixes.

### Lessons Learned and Next Steps:

* Metrics and user feedback are not interchangeable. The logs said which route failed; only the testers said which failure actually cost someone an exam.
* Naming the specific cause of each fix, an index, a cache key, a URL TTL, keeps the changelog honest and makes regressions easier to spot later.
* Next week I would close out the remaining bugs in priority order, write the closing workshop report with the user evaluation data included, and package the whole project and internship dossier for acceptance.
