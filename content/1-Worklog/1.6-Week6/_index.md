---
title: "Week 6 Worklog"
date: 2026-05-25
weight: 6
chapter: false
pre: "<b>1.6. </b>"
---

### Week 6 Objectives:

* Start coding the core features of LingoRise on top of last week's SAM scaffolding.
* Build user management with an **Amazon Cognito** user pool and a mirrored `users` table in **Amazon RDS**.
* Implement role-based authorization in the handlers and least-privilege **IAM** execution roles for each Lambda.
* Expose the first APIs through **Amazon API Gateway** with a working CORS configuration for the Amplify frontend.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Create the Cognito user pool and app client for `lingorise-dev`, then test the sign-up, confirm, and sign-in flow. | 25/05/2026 | 25/05/2026 | [Amazon Cognito User Pools](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools.html) |
| **2** | Add the `users` migration in RDS mirroring the Cognito subject, and write the handler that provisions a local user row on first authenticated request. | 26/05/2026 | 26/05/2026 | [Amazon RDS for PostgreSQL](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_PostgreSQL.html) |
| **3** | Implement role checks for learner, content manager, and admin, and tighten each Lambda IAM execution role to the resources it actually uses. | 27/05/2026 | 27/05/2026 | [IAM Least Privilege](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) |
| **4** | Write the first data-processing handlers that read and write course and question-bank records through the `pg.Pool` singleton. | 28/05/2026 | 28/05/2026 | Project repository |
| **5** | Map REST resources and methods to Lambda in API Gateway, deploy the `dev` stage, and configure CORS for the Amplify domain. | 29/05/2026 | 29/05/2026 | [API Gateway CORS](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-cors.html) |

### Detailed Implementation:

#### 1. User management split between Cognito and the database

I created the **Amazon Cognito** user pool for `lingorise-dev` with email as the sign-in alias, then an app client without a secret so the **Next.js** frontend can call it directly from the browser. I walked the full flow manually first: sign-up, confirmation code by email, sign-in, and inspection of the returned ID token.

Cognito gives me a stable subject (`sub`) but it is not a good place to keep application state. So I added a `users` table in **Amazon RDS for PostgreSQL** that mirrors that subject and carries everything the product needs:

```sql
CREATE TABLE IF NOT EXISTS users (
  id            BIGSERIAL PRIMARY KEY,
  cognito_sub   TEXT UNIQUE NOT NULL,
  email         TEXT NOT NULL,
  display_name  TEXT,
  role          TEXT NOT NULL DEFAULT 'learner',
  is_active     BOOLEAN NOT NULL DEFAULT TRUE,
  created_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

The division of responsibility is the point: Cognito owns identity and credentials, the database owns role, active flag, profile, and later the subscription link. A shared `ensureUser` helper runs on the first authenticated request and inserts the row idempotently, so a Cognito user always has exactly one local counterpart.

#### 2. Authorization: role checks in code, least privilege in IAM

Authorization happens on two levels this week. In the handler, after `aws-jwt-verify` validates the token, I load the local user row and resolve its `role` into one of three values: `learner`, `content_manager`, or `admin`. A small `requireRole()` wrapper rejects with 403 before any query runs, so an expired or downgraded account cannot reach the question bank even if its token is still valid.

At the platform level, I stopped sharing a single execution role across functions. Each **AWS Lambda** now gets its own **IAM** role in the SAM template with only what it needs: VPC networking for the RDS-facing functions, `s3:GetObject` scoped to one prefix for the asset readers, and `ssm:GetParameter` limited to `/lingorise/dev/*`. Writing those policies made the data flow explicit, because any missing permission showed up immediately as an `AccessDenied` in **CloudWatch Logs**.

#### 3. First data-processing services over the question bank

With identity and permissions in place, I wrote the first real handlers: list courses, read a course with its sections, list questions with pagination, and create or update a question. They all go through the `pg.Pool` singleton created outside the handler body, so a warm Lambda reuses its connection instead of opening a new one per invocation. That detail matters on `db.t4g.micro`, where the connection limit is small.

Write paths run inside a transaction and record who made the change using the local user id, which is the beginning of the audit log the content pipeline will need later. Question images stay in **Amazon S3**; the database only stores the object key, and the handler returns a key the frontend resolves separately.

#### 4. API Gateway resources, stage deployment, and CORS

I defined the REST resources in the SAM template so **Amazon API Gateway** stays in source control: `/courses`, `/courses/{courseId}`, `/questions`, and `/me`. Each method maps to its Lambda through proxy integration, and the whole API deploys to a `dev` stage in `ap-southeast-1`.

CORS took the most time. The Amplify domain is a different origin from the API, so the browser sends a preflight `OPTIONS` before any request carrying an `Authorization` header. I configured the allowed origin explicitly instead of `*`, allowed `Authorization` and `Content-Type` headers, and made sure `OPTIONS` is not protected by the authorizer. Verifying it from the terminal was faster than guessing in the browser:

```bash
curl -i -X OPTIONS "$API_URL/questions" \
  -H "Origin: https://dev.lingorise.app" \
  -H "Access-Control-Request-Method: POST" \
  -H "Access-Control-Request-Headers: authorization,content-type"
```

### AWS Service Integration This Week:

* **Cognito + Lambda:** The ID token issued by the user pool is verified in the handler with `aws-jwt-verify` before any business logic runs.
* **Cognito + RDS:** The Cognito `sub` links each identity to one row in the local `users` table that holds role and active state.
* **Lambda + IAM:** Every function received a dedicated execution role scoped to the exact RDS, S3, and SSM resources it touches.
* **API Gateway + Lambda:** REST resources and methods were mapped through proxy integration and released as the `dev` stage.
* **API Gateway + Amplify Hosting:** CORS was configured for the Amplify origin so the Next.js app can call the API with an `Authorization` header.
* **Lambda + S3 + RDS:** Question records live in PostgreSQL while their images stay in S3, joined by the stored object key.

### Week 6 Achievements:

* Working sign-up, confirmation, and sign-in flow against the `lingorise-dev` Cognito user pool.
* A `users` table and idempotent provisioning helper that keeps Cognito identities and application roles in sync.
* Role-based authorization enforced in the handlers for learner, content manager, and admin.
* Per-function IAM execution roles replacing the earlier shared role.
* First data-processing endpoints for courses and the question bank, running on a pooled RDS connection.
* A deployed API Gateway `dev` stage that the frontend can call across origins.

### Challenges Faced:

* A 401 from the authorizer looks exactly like a CORS failure in the browser console, because the error response carries no CORS headers. I lost time on the wrong problem until I checked the raw response with `curl`.
* Splitting one execution role into several broke two functions at deploy time; reading the `AccessDenied` entries in CloudWatch Logs was the only reliable way to find what each handler really needed.
* Deciding what belongs in Cognito attributes and what belongs in the database required a second pass on the schema.

### Lessons Learned and Next Steps:

* Identity and authorization are two different problems. Cognito answers who the caller is, the database and the code answer what that caller may do.
* Least-privilege IAM is easier to write while the feature is fresh than to retrofit later, and it documents the data flow for free.
* Next week I would go deeper on both sides: backend core services with entities, repositories, and business logic, and the frontend project structure with routing, state management, and UI.
