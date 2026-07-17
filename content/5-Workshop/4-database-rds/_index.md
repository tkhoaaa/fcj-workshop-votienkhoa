---
title : "Database — RDS PostgreSQL"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

#### Overview

LingoRise stores every learner profile, exam session, question bank entry and payment record in **PostgreSQL**. In this chapter you will stand up a managed **Amazon RDS for PostgreSQL 16.4** instance (`lingorise-dev-db`), lock its network access down to your own workstation, wire the connection string into **SSM Parameter Store**, and finally run the idempotent migration suite that builds out the ~20+ tables the backend expects.

Everything runs in **ap-southeast-1** against the `lingorise-dev` CLI profile. PowerShell commands use the backtick (`` ` ``) line-continuation character; if you are on bash/zsh, swap the backtick for a backslash (`\`).

![RDS instance being created in the console](/images/Workshop-LingoRise/4-database-rds/rds-creating.png)

#### Create a security group

RDS will live in your account's **default VPC**. First look up that VPC id, then create a dedicated security group for the database.

PowerShell:

```powershell
$VpcId = aws ec2 describe-vpcs `
  --filters "Name=isDefault,Values=true" `
  --query "Vpcs[0].VpcId" `
  --output text `
  --profile lingorise-dev `
  --region ap-southeast-1

$SgId = aws ec2 create-security-group `
  --group-name lingorise-dev-db `
  --description "LingoRise dev database access" `
  --vpc-id $VpcId `
  --query "GroupId" `
  --output text `
  --profile lingorise-dev `
  --region ap-southeast-1
```

bash:

```bash
VpcId=$(aws ec2 describe-vpcs \
  --filters "Name=isDefault,Values=true" \
  --query "Vpcs[0].VpcId" \
  --output text \
  --profile lingorise-dev \
  --region ap-southeast-1)

SgId=$(aws ec2 create-security-group \
  --group-name lingorise-dev-db \
  --description "LingoRise dev database access" \
  --vpc-id "$VpcId" \
  --query "GroupId" \
  --output text \
  --profile lingorise-dev \
  --region ap-southeast-1)
```

Now open TCP **5432** to your workstation only. Look up your current public IP and authorize just that `/32`.

PowerShell:

```powershell
$MyIp = (Invoke-RestMethod -Uri "https://checkip.amazonaws.com").Trim()

aws ec2 authorize-security-group-ingress `
  --group-id $SgId `
  --protocol tcp `
  --port 5432 `
  --cidr "$MyIp/32" `
  --profile lingorise-dev `
  --region ap-southeast-1
```

bash:

```bash
MyIp=$(curl -s https://checkip.amazonaws.com)

aws ec2 authorize-security-group-ingress \
  --group-id "$SgId" \
  --protocol tcp \
  --port 5432 \
  --cidr "$MyIp/32" \
  --profile lingorise-dev \
  --region ap-southeast-1
```

{{% notice note %}}
Home and office IP addresses change. If migrations later fail to connect, re-run the `authorize-security-group-ingress` command with your new public IP — the old `/32` rule may no longer match.
{{% /notice %}}

#### Create the DB instance

Provision a small, cost-friendly `db.t4g.micro` running **PostgreSQL 16.4**, with the master user `lingorise`. Choose a strong master password and keep it handy for the connection string in a moment.

PowerShell:

```powershell
aws rds create-db-instance `
  --db-instance-identifier lingorise-dev-db `
  --db-instance-class db.t4g.micro `
  --engine postgres `
  --engine-version 16.4 `
  --master-username lingorise `
  --master-user-password "<your-strong-password>" `
  --allocated-storage 20 `
  --db-name lingorise `
  --vpc-security-group-ids $SgId `
  --publicly-accessible `
  --backup-retention-period 7 `
  --deletion-protection `
  --profile lingorise-dev `
  --region ap-southeast-1
```

bash:

```bash
aws rds create-db-instance \
  --db-instance-identifier lingorise-dev-db \
  --db-instance-class db.t4g.micro \
  --engine postgres \
  --engine-version 16.4 \
  --master-username lingorise \
  --master-user-password "<your-strong-password>" \
  --allocated-storage 20 \
  --db-name lingorise \
  --vpc-security-group-ids "$SgId" \
  --publicly-accessible \
  --backup-retention-period 7 \
  --deletion-protection \
  --profile lingorise-dev \
  --region ap-southeast-1
```

{{% notice note %}}
`--deletion-protection` is on by design so nobody accidentally drops the database. The trade-off is that teardown needs an extra step: you must disable deletion protection before you can delete the instance. We cover exactly that in **Chapter 9 (Cleanup)**.
{{% /notice %}}

#### Capture the endpoint

Provisioning takes several minutes. Block until the instance reports **available**, then read back its connection endpoint.

PowerShell:

```powershell
aws rds wait db-instance-available `
  --db-instance-identifier lingorise-dev-db `
  --profile lingorise-dev `
  --region ap-southeast-1

$DbHost = aws rds describe-db-instances `
  --db-instance-identifier lingorise-dev-db `
  --query "DBInstances[0].Endpoint.Address" `
  --output text `
  --profile lingorise-dev `
  --region ap-southeast-1
```

bash:

```bash
aws rds wait db-instance-available \
  --db-instance-identifier lingorise-dev-db \
  --profile lingorise-dev \
  --region ap-southeast-1

DbHost=$(aws rds describe-db-instances \
  --db-instance-identifier lingorise-dev-db \
  --query "DBInstances[0].Endpoint.Address" \
  --output text \
  --profile lingorise-dev \
  --region ap-southeast-1)
```

![RDS instance in available state with its endpoint](/images/Workshop-LingoRise/4-database-rds/rds-available.png)

#### Store DATABASE_URL in SSM

Assemble the full PostgreSQL connection URL and push it into **SSM Parameter Store** as a `SecureString` under the `/lingorise/dev/` root. The backend and the migration runner both read `DATABASE_URL` from here.

PowerShell:

```powershell
$DbUrl = "postgresql://lingorise:<your-strong-password>@$($DbHost):5432/lingorise?sslmode=require"

aws ssm put-parameter `
  --name "/lingorise/dev/DATABASE_URL" `
  --type SecureString `
  --value $DbUrl `
  --overwrite `
  --profile lingorise-dev `
  --region ap-southeast-1
```

bash:

```bash
DbUrl="postgresql://lingorise:<your-strong-password>@${DbHost}:5432/lingorise?sslmode=require"

aws ssm put-parameter \
  --name "/lingorise/dev/DATABASE_URL" \
  --type SecureString \
  --value "$DbUrl" \
  --overwrite \
  --profile lingorise-dev \
  --region ap-southeast-1
```

{{% notice warning %}}
`sslmode=require` is **mandatory**. RDS PostgreSQL enforces TLS in transit, and the LingoRise backend refuses to open an unencrypted connection. Drop the `?sslmode=require` suffix and both the migrations and the running app will fail to connect.
{{% /notice %}}

#### Run migrations

The migration runner is idempotent — it tracks which migrations have already been applied in a `_migrations` table, so re-running it is safe and only executes what is new. Move into the backend, export the same connection string locally, install dependencies, and run the suite.

PowerShell:

```powershell
cd lingorise-backend

$env:DATABASE_URL = $DbUrl

npm install
npm run migrate
```

bash:

```bash
cd lingorise-backend

export DATABASE_URL="$DbUrl"

npm install
npm run migrate
```

![Migration runner output listing applied migrations](/images/Workshop-LingoRise/4-database-rds/migrations-output.png)

#### Verify

Connect with `psql` and list the tables. A healthy schema shows **~20+ tables** — users, exam sessions, question banks, payments, testimonials, and more.

```bash
psql "$DbUrl" -c "\dt"
```

You should see a table listing similar to:

```text
             List of relations
 Schema |       Name        | Type  |   Owner
--------+-------------------+-------+-----------
 public | _migrations       | table | lingorise
 public | exam_sessions     | table | lingorise
 public | payments          | table | lingorise
 public | question_bank     | table | lingorise
 public | testimonials      | table | lingorise
 public | users             | table | lingorise
 ...
(20+ rows)
```

If the `_migrations` table is present and the row count is 20 or more, your database layer is ready. Next up, in **Chapter 5**, we ship the backend that talks to it.
