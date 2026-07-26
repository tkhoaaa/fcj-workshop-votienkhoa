---
title: "Week 8 Worklog"
date: 2026-06-08
weight: 8
chapter: false
pre: "<b>1.8. </b>"
---

### Week 8 Objectives:

* Connect the LingoRise **Next.js** frontend to the **API Gateway** REST endpoints through one typed API client.
* Render real data from **Amazon RDS for PostgreSQL** in the course catalog, exam catalog, and exam session screens.
* Implement authentication and authorization with **Amazon Cognito** JWT verified inside **AWS Lambda**.
* Make the exam session resume flow survive a page reload without losing answers.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Build the typed API client in the Next.js app, centralize the base URL per stage, and attach `Authorization: Bearer <jwt>` to every request. | 08/06/2026 | 08/06/2026 | [Amazon API Gateway](https://aws.amazon.com/api-gateway/) |
| **2** | Wire React Query hooks for the course catalog and the exam catalog with availability states, plus loading, error, and empty rendering. | 09/06/2026 | 09/06/2026 | Project repository |
| **3** | Verify Cognito JWT inside Lambda with `aws-jwt-verify` against the user pool JWKS, and keep a dev-token fallback for local runs. | 10/06/2026 | 10/06/2026 | [Amazon Cognito Docs](https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-verifying-a-jwt.html) |
| **4** | Resolve the verified token subject to the local `users` row and enforce role and active checks in a shared authorizer helper. | 11/06/2026 | 11/06/2026 | [IAM and Authorization](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-control-access-to-api.html) |
| **5** | Integrate the exam session start/resume flow and the per-question review screen, then debug a 401 caused by token expiry and clock skew. | 12/06/2026 | 12/06/2026 | Self-study notes |

### Detailed Implementation:

#### 1. One typed API client between Next.js and API Gateway

Until this week the frontend used local fixtures. I replaced them with a single `apiFetch` wrapper that reads the **API Gateway** invoke URL from `NEXT_PUBLIC_API_BASE_URL`, injects `Authorization: Bearer <jwt>` from the current **Amazon Cognito** session, and normalizes the error envelope the Lambda handlers already returned.

Centralizing the client meant every screen shared the same behavior for stage selection, timeouts, and error mapping. When the API moved from the local SAM port to the `lingorise-dev` stage URL, only one environment variable in **AWS Amplify Hosting** changed.

```ts
const res = await fetch(`${BASE}${path}`, {
  ...init,
  headers: { "Content-Type": "application/json", Authorization: `Bearer ${token}`, ...init?.headers },
});
if (!res.ok) throw new ApiError(res.status, (await res.json()).message);
return res.json() as Promise<T>;
```

#### 2. React Query hooks for catalog, session, and review data

On top of the client I added React Query hooks: `useCourses`, `useExams`, `useSession`, and `useSessionReview`. The exam catalog needed more than a list, because each exam carries an availability state that depends on the entitlement resolved server-side from `user_subscriptions`. I mapped those states to three renderings: available, locked with an upgrade prompt, and unpublished.

Every hook now has explicit loading, error, and empty branches. The empty state mattered for a Vietnamese-first UI, since an empty exam list previously looked like a broken page rather than a course with no published exams yet.

#### 3. Cognito JWT verification inside Lambda

Authorization happens in the **AWS Lambda** handler rather than at the edge, because each request also needs the local user row. I used `aws-jwt-verify` to validate the access token against the Cognito user pool JWKS, checking issuer, audience, `token_use`, and expiry. The pool id and client id come from **AWS Systems Manager Parameter Store** at deploy time, so no identifiers are hardcoded in the SAM template.

After verification, the shared authorizer helper resolves `sub` to the matching row in the `users` table and reads `role` and `is_active`. That gave a clean split: an invalid or missing token returns 401, while a valid token whose user lacks the admin role, or whose account is deactivated, returns 403. Local development keeps a dev-token fallback that is only enabled when the stage is not production.

#### 4. Exam session start, resume, and per-question review

The session flow was the hardest part to integrate. `POST /sessions` either starts a new attempt or returns the in-progress one, so the frontend treats start and resume as the same call. Answers are persisted per question, and on mount the client rehydrates from the server response instead of local state. Reloading the page mid-exam now restores the same question index, the saved answers, and the remaining time computed from the server-side `started_at`.

The review screen consumes the per-question payload with the correct answer, the explanation, and the listening transcript, all served from **Amazon RDS for PostgreSQL** with images signed from **Amazon S3**. Review data is only returned after the attempt is submitted, which the handler enforces rather than the UI.

### AWS Service Integration This Week:

* **Next.js on Amplify Hosting + API Gateway:** The frontend called the REST stage URL through one typed client configured per environment.
* **Amazon Cognito + AWS Lambda:** Access tokens were verified inside the handler with `aws-jwt-verify` against the user pool JWKS.
* **Cognito + Amazon RDS for PostgreSQL:** The verified token subject was resolved to a `users` row for role and active checks.
* **API Gateway + Lambda + RDS:** Catalog, session, and review requests returned live database rows instead of frontend fixtures.
* **Lambda + Amazon S3:** Question images and listening assets were served to the review screen through signed URLs.
* **Parameter Store + AWS SAM:** Cognito pool and client identifiers were resolved at deploy time into the `lingorise-dev` stack.

### Week 8 Achievements:

* Replaced all frontend fixtures with live data from the API Gateway plus Lambda plus RDS path.
* Delivered end-to-end authentication with Cognito tokens verified server-side and protected routes on the client.
* Separated 401 and 403 semantics consistently across every handler through a shared authorizer helper.
* Made the exam session resume flow reliable across page reloads, including remaining time.
* Shipped loading, error, and empty states for each React Query hook so no screen renders a blank page.

### Challenges Faced:

* A batch of requests failed with 401 only on the deployed stage. The token was valid, but my local machine's clock had drifted about two minutes ahead, so freshly issued tokens looked not-yet-valid. I fixed the clock and set a small tolerance in the verifier, then added a refresh-and-retry path for genuinely expired tokens.
* The browser preflight started failing after I added a custom request header for the client version, because `Authorization` alone was listed in `Access-Control-Allow-Headers` on the API Gateway CORS configuration.
* Deciding where authorization belongs took some rework, since a Cognito authorizer at the gateway would still have required a database lookup for role and entitlement.

### Lessons Learned and Next Steps:

* Verifying tokens next to the data lookup keeps authorization decisions in one place, at the cost of a few milliseconds per request.
* Clear 401 versus 403 semantics saved time on the frontend, because the client could decide between refreshing a session and showing an upgrade prompt.
* Next week I will move into testing: functional and integration tests across the frontend, the Lambda backend, and the AWS services behind them, fixing the bugs those tests expose and optimizing the AI processing pipeline.
