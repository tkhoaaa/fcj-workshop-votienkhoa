---
title : "Backend — AWS SAM Deploy"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

#### Overview

AWS SAM (Serverless Application Model) is a CloudFormation extension purpose-built for serverless apps. You describe your backend in a single `template.yaml` — Lambda functions, an API Gateway HTTP API, a Cognito user pool, IAM policies, and an S3 bucket — and SAM turns that declaration into a real CloudFormation stack.

For LingoRise, `template.yaml` maps every backend route to a `AWS::Serverless::Function` (handler code compiled from `src/handlers/**`), fronts them with a shared API Gateway, and wires authentication through a `CognitoUserPool` + `CognitoUserPoolClient`. Sensitive values are pulled at deploy time from SSM Parameter Store using `{{resolve:ssm:/lingorise/${Stage}/...}}`, so no secrets live in the template.

Before we deploy, the template needs five readiness gaps closed. We apply them in a deliberate order, validate, build, then run the guided deploy.

{{% notice info %}}
Everything in this chapter runs from the `lingorise-backend/` directory with the `lingorise-dev` CLI profile against region `ap-southeast-1`. Stage is `dev`; the stack is named `lingorise-dev`.
{{% /notice %}}

#### Apply the readiness gaps

The template scaffolding (Lambda, API Gateway, Cognito, CORS, SSM resolution) is already in good shape. The gaps below are the pieces that must be closed before the asset, OCR, and payment routes will work at runtime. Apply them in this order.

**Gap 2 — fix the build script first.** The current esbuild command externalizes pure-JS libraries (`mammoth`, `pdf-parse`, `tesseract.js`, `@anthropic-ai/sdk`) that SAM will not ship, so `/admin/import/parse` and the OCR worker would throw `MODULE_NOT_FOUND` at runtime. Drop those externals and keep only the native binaries excluded. Edit `lingorise-backend/package.json`:

```json
"build": "esbuild src/handlers/**/*.ts --bundle --platform=node --target=node20 --outdir=dist --external:pg-native --external:@napi-rs/canvas --external:@napi-rs/canvas-*"
```

This produces self-contained per-handler bundles. `pg-native` and `@napi-rs/canvas` stay external because they are native binaries that must not be bundled.

**Gap 3 — declare the S3 asset bucket.** Code reads `ASSET_S3_BUCKET`, but no bucket resource exists. Add this to `Resources:`:

```yaml
  AssetBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "lingorise-assets-${Stage}-${AWS::AccountId}"
      OwnershipControls:
        Rules:
          - ObjectOwnership: BucketOwnerEnforced
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
      CorsConfiguration:
        CorsRules:
          - AllowedMethods: [GET, HEAD]
            AllowedOrigins:
              - !Ref FrontendUrl
            AllowedHeaders: ["*"]
            MaxAge: 3000
      LifecycleConfiguration:
        Rules:
          - Id: ExpireImportDrafts
            Status: Enabled
            Prefix: import-drafts/
            ExpirationInDays: 7
          - Id: AbortIncompleteMultipart
            Status: Enabled
            AbortIncompleteMultipartUpload:
              DaysAfterInitiation: 1
```

…and surface the name under `Outputs:`:

```yaml
  AssetBucketName:
    Description: S3 bucket holding question assets and import drafts
    Value: !Ref AssetBucket
```

The bucket is private by default (`BlockPublicAcls=true`), which pairs with presigned URLs from the backend.

**Gap 4 — propagate asset & OCR env vars.** None of the `ASSET_*` / `OCR_*` variables the code reads are in `Globals`. Add them to the existing `Globals.Function.Environment.Variables` block:

```yaml
        ASSET_STORAGE_DRIVER: "s3"
        ASSET_S3_BUCKET: !Ref AssetBucket
        ASSET_S3_PREFIX: ""
        ASSET_PUBLIC_BASE_URL: ""
        ASSET_S3_FORCE_PATH_STYLE: "false"
        ASSET_URL_MODE: "signed"
        ASSET_SIGNED_URL_TTL_SECONDS: "900"
        OCR_ENABLED: "false"
        OCR_PROVIDER: "disabled"
        OCR_RUN_MODE: "queue"
        OCR_LANG: "eng"
        OCR_TIMEOUT_MS: "60000"
```

A private bucket plus presigned URLs (`ASSET_URL_MODE: signed`) is the safest starting state for dev. OCR ships disabled; enable it per-stage later by flipping `OCR_ENABLED=true` and `OCR_PROVIDER=tesseract`.

**Gap 5 — grant IAM for Lambda → S3 and Lambda → Cognito.** SAM does not auto-grant on `{{resolve:ssm:...}}`, and the asset/auth code needs bucket and user-pool permissions. Add a `Policies:` block to `Globals.Function` so every function inherits least-privilege access:

```yaml
Globals:
  Function:
    # ...existing entries unchanged
    Policies:
      - S3CrudPolicy:
          BucketName: !Ref AssetBucket
      - Statement:
          - Effect: Allow
            Action:
              - cognito-idp:AdminInitiateAuth
              - cognito-idp:AdminGetUser
              - cognito-idp:AdminCreateUser
              - cognito-idp:AdminSetUserPassword
              - cognito-idp:ListUsers
            Resource: !GetAtt CognitoUserPool.Arn
```

`S3CrudPolicy` is a SAM-managed policy template scoped to the named bucket; it also covers `s3:ListBucket` for the import-draft cleanup.

**Gap 1 — audit route coverage last.** The template declares only a subset of the routes registered in `server.ts`. Before deploying, walk `template.yaml` against `server.ts` and add a `AWS::Serverless::Function` entry for every route you intend to expose. Handlers follow the existing pattern: `dist/handlers/{group}/{file}.{export}`. Routes commonly missing include the admin OCR/import endpoints (`/admin/ocr/jobs`, `/admin/import/jobs`), question-bank asset routes, `/admin/audit-logs`, and the VNPay v2 / SePay payment endpoints (`/payments/vnpay/**`, `/payments/sepay/**`).

{{% notice tip %}}
Run gaps 2 through 5 first because they change the template and build script directly. Save the route audit (gap 1) for last so you review the final resource list against `server.ts` right before validating.
{{% /notice %}}

#### Validate the template

With the gaps applied, lint the template before building. This catches typos, bad indentation, and unresolved references early.

```powershell
sam validate --lint
```

```bash
sam validate --lint
```

A clean run reports the template is valid. Fix any reported issues before continuing.

#### Build

First compile the TypeScript handlers with esbuild (this is what the gap 2 change affects), then let SAM package everything for Lambda.

```powershell
npm run build
sam build
```

```bash
npm run build
sam build
```

`npm run build` should finish without any warnings about missing modules — that confirms the gap 2 fix bundled the JS-only dependencies. `sam build` then assembles the deployment artifacts into `.aws-sam/`.

#### Deploy

Run the guided deploy. SAM walks you through the parameters interactively and can persist your answers to `samconfig.toml` for subsequent deploys.

```powershell
sam deploy --guided --stack-name lingorise-dev --region ap-southeast-1
```

```bash
sam deploy --guided --stack-name lingorise-dev --region ap-southeast-1
```

Answer the wizard prompts as follows:

| Parameter | Value |
|---|---|
| Stack Name | `lingorise-dev` |
| AWS Region | `ap-southeast-1` |
| Stage | `dev` |
| FrontendUrl | `<your-frontend-url>` |
| VnpayTmnCode | `<paste>` |
| MomoPartnerCode | `<paste>` |
| AiProvider | `openrouter` |
| OllamaBaseUrl | (leave default / empty) |
| OpenRouterModel | `<your-openrouter-model>` |
| LogRetentionDays | `30` |
| Confirm changes before deploy | `Y` |
| Allow SAM CLI IAM role creation | `Y` |
| Save arguments to samconfig.toml | `Y` |

{{% notice warning %}}
`AiProvider` must be `openrouter`. Lambda functions cannot reach `localhost:11434`, so the Ollama provider will never work in the deployed environment — it only exists for local development. Set it to `openrouter` (with a valid `OpenRouterModel`) or AI-backed features will fail at runtime.
{{% /notice %}}

SAM shows the CloudFormation changeset — the exact resources it will create or modify. Review it, then confirm to execute.

![SAM deploy changeset review](/images/Workshop-LingoRise/5-backend-sam/sam-deploy-changeset.png)

CloudFormation provisions the stack: Lambda functions, the API Gateway, the Cognito user pool, IAM roles, and the S3 asset bucket. Wait for the `CREATE_COMPLETE` status.

![CloudFormation stack CREATE_COMPLETE](/images/Workshop-LingoRise/5-backend-sam/cfn-stack-complete.png)

{{% notice note %}}
Because you answered `Y` to saving arguments, a `samconfig.toml` is written to `lingorise-backend/`. Future deploys are just `sam build && sam deploy` — no `--guided` needed unless a parameter changes.
{{% /notice %}}

#### Capture the outputs

When the deploy finishes, SAM prints the stack outputs. You can also fetch them any time with the CLI:

```powershell
aws cloudformation describe-stacks --stack-name lingorise-dev --region ap-southeast-1 --profile lingorise-dev --query "Stacks[0].Outputs" --output table
```

```bash
aws cloudformation describe-stacks --stack-name lingorise-dev --region ap-southeast-1 --profile lingorise-dev --query "Stacks[0].Outputs" --output table
```

Record these four values — the frontend and later chapters depend on them:

- **ApiUrl** — the API Gateway base URL your frontend will call.
- **UserPoolId** — the Cognito user pool ID for authentication.
- **UserPoolClientId** — the Cognito app client ID the frontend uses to sign in.
- **AssetBucketName** — the S3 bucket holding question assets and import drafts (from gap 3).

![CloudFormation stack outputs](/images/Workshop-LingoRise/5-backend-sam/stack-outputs.png)

#### Verify

The backend is live when the stack is `CREATE_COMPLETE` and all four outputs are populated. A quick smoke test: hit a public endpoint on the `ApiUrl` (for example a health route) and confirm you get a response through API Gateway. Keep `ApiUrl`, `UserPoolId`, and `UserPoolClientId` handy — you'll wire them into the frontend in the next chapter.
