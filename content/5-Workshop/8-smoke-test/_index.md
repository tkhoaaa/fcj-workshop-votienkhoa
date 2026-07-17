---
title : "Smoke Test & Operations"
date : 2024-01-01
weight : 8
chapter : false
pre : " <b> 8. </b> "
---

#### Overview

The stack is deployed and the frontend is wired to it. Before you call the deployment done, you want proof that the whole request path works end to end: API Gateway routes the request, Lambda runs, the Cognito authorizer accepts (or correctly rejects) tokens, and RDS returns data. This chapter walks a smoke test against the live API, then a real user journey through the Amplify-hosted app while you watch the logs. It closes with the day-two operational bits: the three maintenance scripts, how to schedule them, and a rough cost picture with an AWS Budgets alarm so a runaway bill never surprises you.

Grab the API base URL and the Amplify URL you noted from the earlier chapters. In the commands below, `$ApiUrl` is the API Gateway invoke URL for `lingorise-dev` (for example `https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev`).

#### Smoke test the API

Start with the cheapest possible check: hit a protected route with no token and confirm you get a `401`. Then hit a public route and confirm a `200`. These two calls alone prove that API Gateway, Lambda, and the authorizer are all wired correctly.

PowerShell:

```powershell
$ApiUrl = "https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev"

# Protected route, no token → expect 401
curl.exe -i "$ApiUrl/auth/me"

# Public route → expect 200
curl.exe -i "$ApiUrl/courses"
```

Bash (same calls):

```bash
ApiUrl="https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev"

# Protected route, no token → expect 401
curl -i "$ApiUrl/auth/me"

# Public route → expect 200
curl "$ApiUrl/courses"
```

![Smoke test curl output showing 401 and 200](/images/Workshop-LingoRise/8-smoke-test/smoke-test-curl.png)

{{% notice tip %}}
A `401 Unauthorized` from `/auth/me` is the **expected healthy response**, not a failure. It proves the full chain is live: API Gateway received the request, invoked the Lambda, and the Cognito authorizer correctly rejected the call because no valid Bearer token was present. If you got a `403`, `500`, or a connection error instead, the wiring is off — check the stack outputs and the CloudWatch logs before moving on.
{{% /notice %}}

Now prove the auth flow works with real credentials. Register a user, then log in and confirm you get a JWT back.

PowerShell:

```powershell
# Register a new user
curl.exe -i -X POST "$ApiUrl/auth/register" `
  -H "Content-Type: application/json" `
  -d '{\"email\":\"smoke@example.com\",\"password\":\"<your-password>\",\"name\":\"Smoke Test\"}'

# Log in → expect 200 with a token in the body
curl.exe -s -X POST "$ApiUrl/auth/login" `
  -H "Content-Type: application/json" `
  -d '{\"email\":\"smoke@example.com\",\"password\":\"<your-password>\"}'
```

Bash:

```bash
# Register a new user
curl -i -X POST "$ApiUrl/auth/register" \
  -H "Content-Type: application/json" \
  -d '{"email":"smoke@example.com","password":"<your-password>","name":"Smoke Test"}'

# Log in → expect 200 with a token in the body
curl -s -X POST "$ApiUrl/auth/login" \
  -H "Content-Type: application/json" \
  -d '{"email":"smoke@example.com","password":"<your-password>"}'
```

The login response body contains a `token` field — that is the JWT. Copy it and confirm the protected route now returns `200` with your user record instead of `401`:

```bash
TOKEN="<paste>"
curl -i "$ApiUrl/auth/me" -H "Authorization: Bearer $TOKEN"
```

#### Walk a user journey

Curl proves the plumbing; a real journey proves the product. Open the Amplify URL in a browser and walk the same path a learner would take, keeping the CloudWatch logs open in a second window so you can see each handler fire.

1. **Log in** with the `smoke@example.com` user you just created. The app should store the token and land you on the dashboard.
2. **Browse courses** — the course list is served by the same public `/courses` route you smoke-tested.
3. **Start an exam** — pick a mock test and start it. This exercises `POST /exams/start`, which builds a session, selects questions, and resolves asset URLs, so it touches RDS and S3 in one call.

While you click through, tail the Lambda log groups. Every function logs under `/aws/lambda/lingorise-dev-*`:

PowerShell:

```powershell
aws logs tail /aws/lambda/lingorise-dev-auth-login --follow `
  --region ap-southeast-1 --profile lingorise-dev
```

Bash:

```bash
aws logs tail /aws/lambda/lingorise-dev-exams-start --follow \
  --region ap-southeast-1 --profile lingorise-dev
```

![CloudWatch log tail during the user journey](/images/Workshop-LingoRise/8-smoke-test/cloudwatch-logs.png)

{{% notice note %}}
If a page hangs or returns an error, the CloudWatch log line for that handler almost always names the cause — a missing SSM parameter, a database connection timeout (usually a security-group rule), or an unhandled exception with a stack trace. Fix the root cause rather than retrying blindly.
{{% /notice %}}

#### Operational scripts

LingoRise ships three maintenance scripts that keep data tidy over time. Run them from the backend project directory with the `lingorise-dev` profile active. Each one is safe to run on demand, and each is a good candidate for a scheduled job once you are in steady state.

```bash
# Process pending OCR jobs (question-image extraction)
npm run worker:ocr

# Remove stale import drafts left behind by incomplete uploads
npm run cleanup:import-drafts

# Expire subscriptions whose paid period has ended
npm run subscriptions:expire
```

{{% notice info %}}
On demand is fine for a workshop, but in a real `dev` or production environment you would schedule these. The serverless-native way is an **Amazon EventBridge** cron rule that invokes a Lambda wrapping each script — for example, run `subscriptions:expire` once a day and `worker:ocr` every few minutes. EventBridge cron expressions look like `cron(0 17 * * ? *)` (daily at 17:00 UTC) or `rate(5 minutes)`. Wiring the schedules is optional for this workshop; knowing the three jobs exist is the point.
{{% /notice %}}

#### Cost & monitoring

Because the stack is serverless and scales to zero when idle, a lightly used `dev` environment is cheap. The rough monthly picture:

| Service | Est. monthly cost (dev) |
|---|---|
| Lambda + API Gateway | ~$0–2 (scales to zero when idle) |
| RDS PostgreSQL (`db.t4g.micro`) | ~$13–15 (runs 24/7) |
| S3 + CloudFront | ~$1–3 |
| Cognito | ~$0 (well within free tier) |
| CloudWatch Logs | ~$1–2 |
| **Total** | **~$18–25 / month** |

The single biggest line is RDS, because the database runs around the clock while everything else idles at zero. If you leave the stack up between sessions, that is the cost that accrues.

Set a guardrail so the bill can never surprise you. Create an AWS Budgets alarm that emails you when spend crosses a threshold:

PowerShell:

```powershell
aws budgets create-budget `
  --account-id <accountId> `
  --budget '{\"BudgetName\":\"lingorise-dev-monthly\",\"BudgetLimit\":{\"Amount\":\"30\",\"Unit\":\"USD\"},\"TimeUnit\":\"MONTHLY\",\"BudgetType\":\"COST\"}' `
  --notifications-with-subscribers '[{\"Notification\":{\"NotificationType\":\"ACTUAL\",\"ComparisonOperator\":\"GREATER_THAN\",\"Threshold\":80},\"Subscribers\":[{\"SubscriptionType\":\"EMAIL\",\"Address\":\"<your-email>\"}]}]' `
  --profile lingorise-dev
```

Bash:

```bash
aws budgets create-budget \
  --account-id <accountId> \
  --budget '{"BudgetName":"lingorise-dev-monthly","BudgetLimit":{"Amount":"30","Unit":"USD"},"TimeUnit":"MONTHLY","BudgetType":"COST"}' \
  --notifications-with-subscribers '[{"Notification":{"NotificationType":"ACTUAL","ComparisonOperator":"GREATER_THAN","Threshold":80},"Subscribers":[{"SubscriptionType":"EMAIL","Address":"<your-email>"}]}]' \
  --profile lingorise-dev
```

![AWS Budgets alarm configuration](/images/Workshop-LingoRise/8-smoke-test/budget-alarm.png)

#### Verify

You are done with this chapter when: `GET /auth/me` returns `401` without a token and `200` with one, `GET /courses` returns `200`, you have walked login → courses → start exam in the Amplify app and watched the matching handlers fire in `/aws/lambda/lingorise-dev-*`, and your AWS Budgets alarm is created.

{{% notice info %}}
Next up: **[9. Cleanup]** — tear down everything you provisioned so the RDS meter stops running and no leftover resources keep billing after the workshop.
{{% /notice %}}
