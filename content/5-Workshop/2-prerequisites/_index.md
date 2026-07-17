---
title : "Prerequisites"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

#### Overview

Before we provision anything on AWS, let's get your local toolchain and your AWS access in order. This chapter installs the four tools the workshop relies on, creates a dedicated IAM deployer identity so you never work as the account root, and wires up a named CLI profile (`lingorise-dev`) that every later command will use. Once `aws sts get-caller-identity` returns the account you expect, you're ready to build.

We'll capture two values as we go — your **AWS account ID** (`$AccountId`) and the **stage** (`$Stage = "dev"`). These feed into resource names later, most notably the S3 asset bucket `lingorise-assets-dev-<accountId>`.

#### Required tools

Install the following, then confirm each one prints a version. All four are free and cross-platform.

- **AWS CLI v2** — the command-line interface for AWS. [Install guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- **AWS SAM CLI** — packages and deploys the serverless backend stack. [Install guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)
- **Node.js 20** — the Lambda runtime; also runs the build tooling and DB migrations. [Download](https://nodejs.org/en/download)
- **PostgreSQL client 16 (`psql`)** — connects to RDS to run migrations and checks. [Download](https://www.postgresql.org/download/)

Verify each install:

```powershell
aws --version
sam --version
node --version
psql --version
```

```bash
aws --version
sam --version
node --version
psql --version
```

{{% notice tip %}}
You want **AWS CLI v2** (not v1) and **Node 20.x**. If `aws --version` reports a `1.x` build, remove it and install v2 from the link above — a few commands in this workshop behave differently on v1.
{{% /notice %}}

#### IAM permissions

Rather than deploying as the account root user, create a dedicated IAM user named **`lingorise-deployer`** and give it programmatic access (an access key). This user is the identity behind the `lingorise-dev` CLI profile.

For a **dev** environment, attach the AWS-managed **`PowerUserAccess`** policy and add a small inline policy granting **`iam:*`** — SAM needs to create the Lambda execution roles and policies as part of deploying `lingorise-dev`, and `PowerUserAccess` alone does not include IAM write actions.

Inline policy to attach to `lingorise-deployer`:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "LingoRiseIamForSam",
            "Effect": "Allow",
            "Action": "iam:*",
            "Resource": "*"
        }
    ]
}
```

![Create the lingorise-deployer IAM user](/images/Workshop-LingoRise/2-prerequisites/iam-create-user.png)

{{% notice warning %}}
Do **not** use your AWS account **root** credentials for this workshop. Root has unrestricted access to everything in the account, including billing and account closure, and its keys cannot be scoped down. Always deploy through a dedicated IAM user like `lingorise-deployer`.
{{% /notice %}}

{{% notice note %}}
`PowerUserAccess` + inline `iam:*` is convenient for a **dev** account where you're the only user. For **production**, scope this down to only the services this workshop touches (CloudFormation, Lambda, API Gateway, RDS, S3, Cognito, CloudFront, SSM, CloudWatch, Amplify) and to specific resource ARNs where practical.
{{% /notice %}}

#### Configure the CLI profile

With the access key from the previous step, create a named profile called **`lingorise-dev`**. Run `aws configure` and paste the key, secret, region, and output format when prompted:

```powershell
aws configure --profile lingorise-dev
# AWS Access Key ID [None]: <paste>
# AWS Secret Access Key [None]: <paste>
# Default region name [None]: ap-southeast-1
# Default output format [None]: json
```

```bash
aws configure --profile lingorise-dev
# AWS Access Key ID [None]: <paste>
# AWS Secret Access Key [None]: <paste>
# Default region name [None]: ap-southeast-1
# Default output format [None]: json
```

![Running aws configure for the lingorise-dev profile](/images/Workshop-LingoRise/2-prerequisites/cli-configure.png)

Next, export the profile and region into your shell so you don't have to pass `--profile` on every command. Set them for the current session:

```powershell
$env:AWS_PROFILE = "lingorise-dev"
$env:AWS_DEFAULT_REGION = "ap-southeast-1"
```

```bash
export AWS_PROFILE=lingorise-dev
export AWS_DEFAULT_REGION=ap-southeast-1
```

Now confirm the CLI is talking to the right account and capture the account ID and stage into variables you'll reuse:

```powershell
aws sts get-caller-identity
$AccountId = (aws sts get-caller-identity --query Account --output text)
$Stage = "dev"
Write-Host "AccountId=$AccountId  Stage=$Stage"
```

```bash
aws sts get-caller-identity
AccountId=$(aws sts get-caller-identity --query Account --output text)
Stage=dev
echo "AccountId=$AccountId  Stage=$Stage"
```

A successful `get-caller-identity` looks like this — note the `Arn` ends in `user/lingorise-deployer`:

```json
{
    "UserId": "AIDA...EXAMPLE",
    "Account": "<accountId>",
    "Arn": "arn:aws:iam::<accountId>:user/lingorise-deployer"
}
```

![aws sts get-caller-identity output](/images/Workshop-LingoRise/2-prerequisites/caller-identity.png)

#### Verify

You're ready to continue when all of the following are true:

- `aws --version`, `sam --version`, `node --version`, and `psql --version` all print a version (AWS CLI v2, Node 20).
- The `lingorise-deployer` IAM user exists with `PowerUserAccess` + inline `iam:*`.
- `aws sts get-caller-identity` returns your account and the `user/lingorise-deployer` ARN.
- `$AccountId` and `$Stage` are set in your shell.

{{% notice note %}}
Every command in the chapters that follow assumes the **`lingorise-dev`** profile is active — either via `$env:AWS_PROFILE` / `export AWS_PROFILE=lingorise-dev`, or by appending `--profile lingorise-dev`. If a command ever fails with an authentication or region error, re-check these two environment variables first.
{{% /notice %}}
