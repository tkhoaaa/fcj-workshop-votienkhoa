---
title : "Secrets & SSM Parameter Store"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

#### Overview

LingoRise leans on a fistful of third-party services — VNPay and MoMo for payments, OpenRouter for AI grading, plus a handful of content APIs (YouTube, NewsAPI, GNews, Reddit) — and every one of them ships an API key or a signing secret. Rather than baking those values into code or environment files, we keep them in **AWS Systems Manager (SSM) Parameter Store** as `SecureString` parameters.

`SecureString` gives you two things for free. First, the value is **encrypted at rest with a KMS key** (here, the default AWS-managed key), so nobody browsing the console sees the plaintext. Second, retrieval is audited and gated — you have to explicitly ask for decryption with `--with-decryption` to read the value back.

There's a hard ordering reason to do this *now*, before you deploy anything. The SAM template references these parameters with `{{resolve:ssm:...}}` expressions, and CloudFormation resolves them **at deploy time**. If any referenced parameter is missing, the stack fails on the spot. So we seed Parameter Store first, then build on top of it in the chapters that follow.

Everything runs in **ap-southeast-1** against the `lingorise-dev` CLI profile. The examples use PowerShell (Windows) with a bash equivalent alongside; the `aws ssm put-parameter` invocations below are single-line, so they read identically in either shell.

![SSM Parameter Store list showing the /lingorise/dev/ parameters](/images/Workshop-LingoRise/3-secrets-ssm/ssm-parameters-list.png)

#### Create the parameters

Seed every secret except `DATABASE_URL` under the `/lingorise/dev/` root. Use `--type SecureString` so each value is encrypted at rest. Swap the `<your-...>` placeholders for the real sandbox credentials as you paste them in.

{{% notice info %}}
`DATABASE_URL` is intentionally **not** here. You don't have an RDS endpoint yet — we assemble and store the full connection string in **Chapter 4 (Database — RDS PostgreSQL)** once the instance is up.
{{% /notice %}}

PowerShell:

```powershell
# VNPay sandbox
aws ssm put-parameter --name "/lingorise/dev/VNPAY_HASH_SECRET" --type SecureString --value "<your-vnpay-hash-secret>" --profile lingorise-dev --region ap-southeast-1

# MoMo sandbox
aws ssm put-parameter --name "/lingorise/dev/MOMO_ACCESS_KEY" --type SecureString --value "<your-momo-access-key>" --profile lingorise-dev --region ap-southeast-1
aws ssm put-parameter --name "/lingorise/dev/MOMO_SECRET_KEY" --type SecureString --value "<your-momo-secret-key>" --profile lingorise-dev --region ap-southeast-1

# AI provider keys
aws ssm put-parameter --name "/lingorise/dev/OPENROUTER_API_KEY" --type SecureString --value "<your-openrouter-key>" --profile lingorise-dev --region ap-southeast-1

# Content APIs
aws ssm put-parameter --name "/lingorise/dev/YOUTUBE_API_KEY" --type SecureString --value "<your-youtube-key>" --profile lingorise-dev --region ap-southeast-1
aws ssm put-parameter --name "/lingorise/dev/NEWS_API_KEY" --type SecureString --value "<your-newsapi-key>" --profile lingorise-dev --region ap-southeast-1
aws ssm put-parameter --name "/lingorise/dev/GNEWS_API_KEY" --type SecureString --value "<your-gnews-key>" --profile lingorise-dev --region ap-southeast-1
aws ssm put-parameter --name "/lingorise/dev/REDDIT_CLIENT_ID" --type SecureString --value "<your-reddit-client-id>" --profile lingorise-dev --region ap-southeast-1
aws ssm put-parameter --name "/lingorise/dev/REDDIT_CLIENT_SECRET" --type SecureString --value "<your-reddit-client-secret>" --profile lingorise-dev --region ap-southeast-1

# Database password (used in Chapter 4 by RDS)
aws ssm put-parameter --name "/lingorise/dev/DB_PASSWORD" --type SecureString --value "<your-strong-password>" --profile lingorise-dev --region ap-southeast-1
```

bash:

```bash
# VNPay sandbox
aws ssm put-parameter --name "/lingorise/dev/VNPAY_HASH_SECRET" --type SecureString --value "<your-vnpay-hash-secret>" --profile lingorise-dev --region ap-southeast-1

# MoMo sandbox
aws ssm put-parameter --name "/lingorise/dev/MOMO_ACCESS_KEY" --type SecureString --value "<your-momo-access-key>" --profile lingorise-dev --region ap-southeast-1
aws ssm put-parameter --name "/lingorise/dev/MOMO_SECRET_KEY" --type SecureString --value "<your-momo-secret-key>" --profile lingorise-dev --region ap-southeast-1

# AI provider keys
aws ssm put-parameter --name "/lingorise/dev/OPENROUTER_API_KEY" --type SecureString --value "<your-openrouter-key>" --profile lingorise-dev --region ap-southeast-1

# Content APIs
aws ssm put-parameter --name "/lingorise/dev/YOUTUBE_API_KEY" --type SecureString --value "<your-youtube-key>" --profile lingorise-dev --region ap-southeast-1
aws ssm put-parameter --name "/lingorise/dev/NEWS_API_KEY" --type SecureString --value "<your-newsapi-key>" --profile lingorise-dev --region ap-southeast-1
aws ssm put-parameter --name "/lingorise/dev/GNEWS_API_KEY" --type SecureString --value "<your-gnews-key>" --profile lingorise-dev --region ap-southeast-1
aws ssm put-parameter --name "/lingorise/dev/REDDIT_CLIENT_ID" --type SecureString --value "<your-reddit-client-id>" --profile lingorise-dev --region ap-southeast-1
aws ssm put-parameter --name "/lingorise/dev/REDDIT_CLIENT_SECRET" --type SecureString --value "<your-reddit-client-secret>" --profile lingorise-dev --region ap-southeast-1

# Database password (used in Chapter 4 by RDS)
aws ssm put-parameter --name "/lingorise/dev/DB_PASSWORD" --type SecureString --value "<your-strong-password>" --profile lingorise-dev --region ap-southeast-1
```

{{% notice tip %}}
Adding `--overwrite` lets you re-run any of these commands to update a value in place. Leave it off the first time so you get an error if a parameter name already exists — a cheap guard against clobbering something by accident.
{{% /notice %}}

#### Generate the DB password

Don't hand-type the database master password. Generate a strong random string locally and paste it into the `DB_PASSWORD` parameter above.

bash:

```bash
openssl rand -base64 32
```

PowerShell:

```powershell
[Convert]::ToBase64String((1..32 | ForEach-Object { Get-Random -Max 256 } | ForEach-Object {[byte]$_}))
```

{{% notice warning %}}
**Save the generated password in a password manager before you paste it into SSM.** A `SecureString` parameter cannot be viewed again in plaintext from the console — reading it back requires `aws ssm get-parameter ... --with-decryption`, and even then only if your IAM identity is allowed to decrypt. If you lose it and don't have `--with-decryption` access, you'll be resetting the RDS master password to recover. Chapter 4 reuses this exact value when it builds `DATABASE_URL`.
{{% /notice %}}

#### Verify

List every parameter under the `/lingorise/dev/` root and confirm all ten are present. `describe-parameters` returns metadata only — never the values — so this is safe to run and paste into a ticket.

```powershell
aws ssm describe-parameters --query "Parameters[?starts_with(Name,'/lingorise/dev/')].Name" --profile lingorise-dev --region ap-southeast-1
```

You should see the nine secrets plus `DB_PASSWORD` — ten names in all. `DATABASE_URL` will appear here only after Chapter 4. With the parameter store seeded, you're ready to stand up the database that `DATABASE_URL` will point at.
