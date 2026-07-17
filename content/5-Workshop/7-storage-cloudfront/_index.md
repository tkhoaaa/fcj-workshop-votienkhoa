---
title : "Storage & CloudFront (Bonus)"
date : 2024-01-01
weight : 7
chapter : false
pre : " <b> 7. </b> "
---

#### Overview

This is a **bonus** chapter — an optional, deeper dive. In Chapter 5 the SAM stack created a private asset bucket (`lingorise-assets-dev-<accountId>`) for LingoRise's learning assets: audio clips, question images, and learner uploads. In `dev` the backend serves those assets straight from S3 and everything works. So why add CloudFront?

Three reasons. **Latency** — CloudFront caches assets at edge locations near your learners, so a listening-comprehension audio clip loads fast whether the user is in Hanoi or Ho Chi Minh City. **Signed access** — you can keep the bucket fully private and still hand out short-lived, signed URLs so only an authorised, logged-in learner can fetch an asset. **Offloading Lambda** — once a URL is signed it is static for its TTL, so repeat fetches hit the CloudFront cache instead of round-tripping through your API and S3 on every request.

The backend supports three asset URL modes, selected by the `ASSET_URL_MODE` environment variable:

| Mode | Latency | Cost | Setup |
|---|---|---|---|
| `public` | Direct `storage_url` | S3 GET | Public bucket |
| `signed` | One pre-sign per response | S3 GET | Just IAM credentials |
| `cloudfront_signed` | None (URL is static for TTL) | CloudFront cache | Key pair + distribution |

{{% notice note %}}
This chapter is **optional**. The LingoRise app runs perfectly well in `dev` with assets served directly from S3 (`ASSET_URL_MODE=public` or `signed`). Add CloudFront only when you want edge caching and CDN-signed, private asset delivery. You can skip straight to Chapter 8 and come back to this later.
{{% /notice %}}

#### The asset bucket

Recall **gap #3** from Chapter 5: the SAM template provisioned a private S3 bucket named `lingorise-assets-dev-<accountId>`. It is locked down — public access is fully blocked — and the Lambda execution role is granted just enough S3 permissions to manage assets.

If you ever need to create the bucket by hand (for example a separate `prod` bucket), the pattern is:

```bash
aws s3api create-bucket \
  --bucket lingorise-assets-dev-<accountId> \
  --region ap-southeast-1 \
  --create-bucket-configuration LocationConstraint=ap-southeast-1

aws s3api put-public-access-block \
  --bucket lingorise-assets-dev-<accountId> \
  --public-access-block-configuration BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true
```

The IAM permissions the backend needs on this bucket:

- `s3:PutObject`, `s3:DeleteObject` — upload and clean up assets.
- `s3:ListBucket` — used by the cleanup job.
- `s3:GetObject` — only when `ASSET_URL_MODE=signed` (the backend pre-signs S3 URLs itself).

The backend's storage driver is wired up through these environment variables (already in SSM under `/lingorise/dev/`):

```bash
ASSET_STORAGE_DRIVER=s3
AWS_REGION=ap-southeast-1
ASSET_S3_BUCKET=lingorise-assets-dev-<accountId>
ASSET_S3_PREFIX=question-assets    # optional
```

Because the bucket blocks all public access, CloudFront cannot read it with a plain public-read policy. Instead you grant CloudFront access through **Origin Access Control (OAC)** — the modern replacement for the legacy Origin Access Identity (OAI). OAC lets a specific distribution read the private bucket while everyone else stays locked out.

{{% notice info %}}
If your frontend fetches assets from a different origin than the API, remember the bucket (or CloudFront) needs a **CORS** configuration that allows your Amplify domain. In `dev` the assets are typically referenced from the same app origin, so no extra CORS is required — add it only if the browser reports a cross-origin block.
{{% /notice %}}

#### Create a CloudFront distribution

Open the **AWS CloudFront Console** and choose **Create distribution**.

![Create a CloudFront distribution in the console](/images/Workshop-LingoRise/7-storage-cloudfront/cloudfront-create-distribution.png)

1. **Origin domain** — select the private asset bucket `lingorise-assets-dev-<accountId>`.
2. **Origin access** — choose **Origin access control settings (recommended)**, then **Create control setting** to make a new OAC. Accept the defaults and sign requests with SigV4.
3. After the distribution is created, CloudFront shows a **bucket policy statement** to copy — paste it into the bucket's policy so CloudFront (and only this distribution) can `s3:GetObject`. Keep the public-access block fully on; OAC does not require public read.
4. **Viewer protocol policy** — **Redirect HTTP to HTTPS** (HTTPS only).
5. **Cache behaviour** — respect the `Cache-Control` headers that the backend sets on each object, so assets are cached at the edge for their intended lifetime.

![Configure Origin Access Control to the private bucket](/images/Workshop-LingoRise/7-storage-cloudfront/cloudfront-oac.png)

When the distribution finishes deploying (it takes a few minutes), note its domain name — something like `d123abc.cloudfront.net`. That is the host your signed URLs will point at.

{{% notice tip %}}
Because CloudFront is a global service, distributions are managed from the global (N. Virginia) console scope even though your origin bucket lives in `ap-southeast-1`. The bucket region does not change — only the CloudFront control plane is global.
{{% /notice %}}

#### Signed URLs / private assets

With CloudFront in front of a private bucket, you want only authorised, logged-in learners to fetch assets. CloudFront **signed URLs** solve this: the backend signs each asset URL with a private key, and CloudFront rejects any request whose signature is missing, tampered with, or expired.

First, create a CloudFront key pair:

1. AWS Console → **CloudFront → Public keys → Create public key**, uploading a freshly generated PEM public key.
2. Keep the matching **private** PEM on the backend host, or in **AWS Secrets Manager** / SSM — never commit it.
3. Add the public key to a **Key group**, then set that key group as the distribution behaviour's **trusted key groups**.

Then point the backend at CloudFront by setting these variables (in SSM under `/lingorise/dev/`):

```bash
ASSET_URL_MODE=cloudfront_signed
CLOUDFRONT_DOMAIN=d123abc.cloudfront.net
CLOUDFRONT_KEY_PAIR_ID=K2EXAMPLE
# Pick ONE of:
CLOUDFRONT_PRIVATE_KEY_PATH=/etc/lingorise/cloudfront.pem
CLOUDFRONT_PRIVATE_KEY_BASE64=<paste base64 of the PEM>
CLOUDFRONT_SIGNED_URL_TTL_SECONDS=3600
```

{{% notice tip %}}
On Lambda the filesystem is read-only, so prefer **`CLOUDFRONT_PRIVATE_KEY_BASE64`** over a file path. To rotate a key, swap the env var and the key in CloudFront — the backend reads the key at call time, so there is no redeploy needed. The signed-URL TTL is clamped to the range `[60, 604800]` seconds.
{{% /notice %}}

Switching modes is safe at any time. The `storage_url` and `storage_path` persisted in the `question_assets` table are mode-agnostic, so changing `ASSET_URL_MODE` takes effect on the very next response — no database migration, and in-flight exam sessions keep working because URLs are resolved on read, not at session start.

#### Verify

Confirm the backend is actually signing through CloudFront by hitting the health endpoint:

```bash
curl -s https://<your-api-id>.execute-api.ap-southeast-1.amazonaws.com/dev/admin/system/health \
  -H "Authorization: Bearer <paste-admin-token>" | jq .assetUrlMode
# → "cloudfront_signed"
```

The `/admin/system-health` panel also lists any config issues (a missing key pair, unparseable base64, and so on) without ever exposing the key contents.

Finally, fetch a real asset through the CloudFront domain and confirm it returns `200`:

```bash
curl -sI "https://d123abc.cloudfront.net/question-assets/<some-asset>?Expires=...&Signature=...&Key-Pair-Id=K2EXAMPLE"
# → HTTP/2 200
```

![An asset served through the CloudFront CDN domain](/images/Workshop-LingoRise/7-storage-cloudfront/asset-via-cdn.png)

{{% notice note %}}
If assets return **403** in the browser despite `cloudfront_signed` mode, the key pair is almost certainly not attached to the distribution's trusted key groups. If the backend logs `cloudfront signed URL fallback`, the private key failed to load — check the `/admin/system-health` issues panel. And if learners cannot see new uploads, confirm the bucket policy allows the backend's `PutObject` and that the CloudFront domain matches `ASSET_PUBLIC_BASE_URL`.
{{% /notice %}}

With storage and CDN sorted, head to Chapter 8 to run an end-to-end smoke test of the whole platform.
