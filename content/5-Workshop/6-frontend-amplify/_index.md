---
title : "Frontend — AWS Amplify Hosting"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

#### Overview

With the backend live behind API Gateway and Cognito wired up, it is time to put a web app in front of it. In this chapter you will host the LingoRise frontend on **AWS Amplify Hosting**. Amplify connects directly to your GitHub repository, auto-detects the **Next.js** app inside the monorepo, builds it on every push, and serves it over HTTPS from a global CDN — no build server to manage.

You will connect the repo, feed Amplify the environment variables that point the web app at the resources you created in Chapter 5, run the first build to get a hosted URL, and finally tighten CORS on the backend so only your Amplify domain can call the API.

{{% notice info %}}
Everything here targets the **`dev`** stage in **`ap-southeast-1`**, using the CloudFormation stack **`lingorise-dev`**. The frontend lives in the **`lingorise-app`** folder of the monorepo and builds from the **`votienkhoa-fullstack`** branch.
{{% /notice %}}

#### Connect the repository

Open the **AWS Amplify Console** in `ap-southeast-1`, then choose **New app → Host web app**. Pick **GitHub** as the source provider and authorize Amplify to access your account.

1. Select the **LingoRise** repository.
2. Choose the **`votienkhoa-fullstack`** branch.
3. Name the app **`lingorise-dev`**.
4. Because this is a monorepo, set the **app root (monorepo root)** to **`lingorise-app`**.
5. Amplify auto-detects **Next.js** and fills in the build settings for you — leave the generated `amplify.yml` as-is.

![Connect the LingoRise GitHub repository in the Amplify Console](/images/Workshop-LingoRise/6-frontend-amplify/amplify-connect-repo.png)

{{% notice note %}}
If Amplify does not detect the framework automatically, confirm the app root is set to `lingorise-app` — pointing it at the repo root will make the Next.js project invisible to the build.
{{% /notice %}}

#### Configure environment variables

The frontend reads a handful of `NEXT_PUBLIC_*` variables at build time so it knows where the API lives and how to talk to Cognito. In the **Environment variables** section of the app settings, add the following:

| Variable | Value | Source |
|---|---|---|
| `NEXT_PUBLIC_API_URL` | `<ApiUrl output>` | `ApiUrl` from the `lingorise-dev` stack |
| `NEXT_PUBLIC_COGNITO_USER_POOL_ID` | `<UserPoolId output>` | `UserPoolId` from the `lingorise-dev` stack |
| `NEXT_PUBLIC_COGNITO_CLIENT_ID` | `<UserPoolClientId output>` | `UserPoolClientId` from the `lingorise-dev` stack |
| `NEXT_PUBLIC_COGNITO_REGION` | `ap-southeast-1` | Fixed — the workshop region |
| `NEXT_PUBLIC_USE_MOCK` | `false` | Fixed — talk to the real API, not mocks |

![Set the NEXT_PUBLIC environment variables in Amplify](/images/Workshop-LingoRise/6-frontend-amplify/amplify-env-vars.png)

{{% notice tip %}}
The values for `NEXT_PUBLIC_API_URL`, `NEXT_PUBLIC_COGNITO_USER_POOL_ID`, and `NEXT_PUBLIC_COGNITO_CLIENT_ID` come **straight from the Chapter 5 stack outputs**. If you closed that terminal, print them again:

```powershell
aws cloudformation describe-stacks `
  --stack-name lingorise-dev `
  --region ap-southeast-1 `
  --profile lingorise-dev `
  --query "Stacks[0].Outputs"
```

```bash
aws cloudformation describe-stacks \
  --stack-name lingorise-dev \
  --region ap-southeast-1 \
  --profile lingorise-dev \
  --query "Stacks[0].Outputs"
```
{{% /notice %}}

#### Run the first build

Save the environment variables and choose **Save and deploy**. Amplify clones the branch, installs dependencies, runs the Next.js build, and deploys the output to its managed CDN. You can follow **Provision → Build → Deploy → Verify** live in the console.

When the pipeline turns green, Amplify shows the hosted URL — something like `https://votienkhoa-fullstack.<app-id>.amplifyapp.com`. Open it and confirm the LingoRise landing page renders.

![Amplify build succeeded with the hosted URL](/images/Workshop-LingoRise/6-frontend-amplify/amplify-build-success.png)

Copy that hosted URL — you need it in the next step to lock down CORS.

{{% notice note %}}
On this first load the app can reach the API, but the backend still allows a permissive CORS origin from earlier deploys. Sign-in and API calls that depend on a strict origin may misbehave until you complete the next step.
{{% /notice %}}

#### Tighten CORS

Right now the backend was deployed without knowing the frontend's real address. Redeploy the SAM stack, passing the Amplify URL so API Gateway and S3 only accept requests from your web app:

```powershell
sam deploy `
  --stack-name lingorise-dev `
  --region ap-southeast-1 `
  --profile lingorise-dev `
  --parameter-overrides FrontendUrl=<amplify-url>
```

```bash
sam deploy \
  --stack-name lingorise-dev \
  --region ap-southeast-1 \
  --profile lingorise-dev \
  --parameter-overrides FrontendUrl=<amplify-url>
```

Replace `<amplify-url>` with the hosted URL from the previous step (for example `https://votienkhoa-fullstack.<app-id>.amplifyapp.com`). This single override updates two things at once:

- **API Gateway CORS** — the `Access-Control-Allow-Origin` for the REST API is set to your Amplify domain.
- **S3 CORS** — the asset bucket `lingorise-assets-dev-<accountId>` accepts browser uploads and reads only from your frontend origin.

{{% notice warning %}}
Use the exact scheme and host (`https://…`, no trailing slash) that the browser sends. A mismatched origin is the most common cause of `CORS`-blocked requests after this step.
{{% /notice %}}

#### Verify

Reload the Amplify URL, open the browser dev tools **Network** tab, and try signing up or signing in. API calls to `NEXT_PUBLIC_API_URL` should return `200`/`401` (not a CORS error), and the responses should carry an `Access-Control-Allow-Origin` header matching your Amplify domain. If they do, the frontend and backend are correctly bound and locked down.
