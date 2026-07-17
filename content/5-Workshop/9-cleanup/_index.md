---
title : "Clean up"
date : 2024-01-01
weight : 9
chapter : false
pre : " <b> 9. </b> "
---

{{% notice warning %}}
This chapter **permanently deletes every resource and all data** you created in this workshop — the RDS database, the S3 asset bucket and its objects, the SSM parameters, and the whole `lingorise-dev` stack. This cannot be undone. Before you run anything below, make sure you have backed up anything you want to keep (a final RDS snapshot, downloaded assets, exported parameter values).
{{% /notice %}}

#### Overview

Now that you have deployed, tested, and operated LingoRise, the last step is to tear it all down so you stop incurring cost. AWS bills the RDS instance, the S3 storage, and other resources for as long as they exist, whether or not anyone is using them — so a clean teardown is the difference between a quiet dev account and a surprise invoice.

Order matters. CloudFormation refuses to delete a stack that owns dependent resources it cannot remove cleanly — an S3 bucket with objects still in it, for example, or an RDS instance with deletion protection on. So we work from the outside in: empty the bucket first, then delete the stack, then remove the standalone resources you created by hand in earlier chapters (the RDS instance, the SSM parameters, and the security group). Follow the five steps in order and everything comes down without dependency errors.

#### Teardown steps

**1. Empty the asset bucket**

CloudFormation (and `sam delete`) cannot delete a non-empty S3 bucket, so clear it out first. This removes every object under the LingoRise asset bucket.

```powershell
aws s3 rm s3://lingorise-assets-dev-<accountId> --recursive `
  --region ap-southeast-1 --profile lingorise-dev
```

```bash
aws s3 rm s3://lingorise-assets-dev-<accountId> --recursive \
  --region ap-southeast-1 --profile lingorise-dev
```

**2. Delete the SAM/CloudFormation stack**

With the bucket empty, tear down the whole backend stack — Lambda functions, API Gateway, Cognito, IAM roles, and the bucket itself — in one command.

```powershell
sam delete --stack-name lingorise-dev `
  --region ap-southeast-1 --profile lingorise-dev
```

```bash
sam delete --stack-name lingorise-dev \
  --region ap-southeast-1 --profile lingorise-dev
```

SAM will ask you to confirm before it deletes the stack and the deployment artifacts. Answer `y` to both prompts.

![sam delete confirmation prompt](/images/Workshop-LingoRise/9-cleanup/sam-delete-confirm.png)

**3. Delete the RDS instance**

The RDS instance (`lingorise-dev-db`) was created by hand in Chapter 4, so it is not part of the stack and must be deleted separately.

{{% notice warning %}}
In **Chapter 4** you created the database with `--deletion-protection` turned on so nobody could drop it by accident. That protection blocks `delete-db-instance`, so you must **turn it off first**. Run the `modify-db-instance --no-deletion-protection` command below and wait for it to apply, then run the delete. Skipping this step will cause the delete to fail with an `InvalidParameterCombination` error.
{{% /notice %}}

First disable deletion protection:

```powershell
aws rds modify-db-instance `
  --db-instance-identifier lingorise-dev-db `
  --no-deletion-protection `
  --apply-immediately `
  --region ap-southeast-1 --profile lingorise-dev
```

```bash
aws rds modify-db-instance \
  --db-instance-identifier lingorise-dev-db \
  --no-deletion-protection \
  --apply-immediately \
  --region ap-southeast-1 --profile lingorise-dev
```

Then delete the instance. `--skip-final-snapshot` deletes it immediately with no final backup — appropriate for a throwaway `dev` database, but drop this flag (and pass `--final-db-snapshot-identifier`) if you want a snapshot kept.

```powershell
aws rds delete-db-instance `
  --db-instance-identifier lingorise-dev-db `
  --skip-final-snapshot `
  --region ap-southeast-1 --profile lingorise-dev
```

```bash
aws rds delete-db-instance \
  --db-instance-identifier lingorise-dev-db \
  --skip-final-snapshot \
  --region ap-southeast-1 --profile lingorise-dev
```

**4. Delete the SSM parameters**

Remove every parameter under the `/lingorise/dev/` path. This loop lists the parameter names first, then deletes each one.

```powershell
$names = aws ssm get-parameters-by-path `
  --path "/lingorise/dev/" --recursive `
  --query "Parameters[].Name" --output text `
  --region ap-southeast-1 --profile lingorise-dev
foreach ($n in ($names -split "\s+")) {
  if ($n) {
    aws ssm delete-parameter --name $n `
      --region ap-southeast-1 --profile lingorise-dev
  }
}
```

```bash
for n in $(aws ssm get-parameters-by-path \
  --path "/lingorise/dev/" --recursive \
  --query "Parameters[].Name" --output text \
  --region ap-southeast-1 --profile lingorise-dev); do
  aws ssm delete-parameter --name "$n" \
    --region ap-southeast-1 --profile lingorise-dev
done
```

**5. Delete the security group**

The `lingorise-dev-db` security group from Chapter 4 is the last standalone resource. It can only be deleted once the RDS instance that used it is fully gone, so wait for the delete in Step 3 to finish before running this.

```powershell
$SgId = aws ec2 describe-security-groups `
  --filters "Name=group-name,Values=lingorise-dev-db" `
  --query "SecurityGroups[0].GroupId" --output text `
  --region ap-southeast-1 --profile lingorise-dev
aws ec2 delete-security-group --group-id $SgId `
  --region ap-southeast-1 --profile lingorise-dev
```

```bash
SgId=$(aws ec2 describe-security-groups \
  --filters "Name=group-name,Values=lingorise-dev-db" \
  --query "SecurityGroups[0].GroupId" --output text \
  --region ap-southeast-1 --profile lingorise-dev)
aws ec2 delete-security-group --group-id "$SgId" \
  --region ap-southeast-1 --profile lingorise-dev
```

{{% notice note %}}
If `delete-security-group` returns a `DependencyViolation`, the RDS instance is still detaching. Give it another minute, confirm the instance is gone (see Verify below), and run the command again.
{{% /notice %}}

#### Verify everything is gone

Run these three checks. Each one should come back empty (or without the LingoRise resource), confirming there is nothing left to bill you for.

The stack should no longer appear as active:

```powershell
aws cloudformation list-stacks `
  --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE `
  --query "StackSummaries[?StackName=='lingorise-dev']" `
  --region ap-southeast-1 --profile lingorise-dev
```

```bash
aws cloudformation list-stacks \
  --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE \
  --query "StackSummaries[?StackName=='lingorise-dev']" \
  --region ap-southeast-1 --profile lingorise-dev
```

The RDS instance should be gone (this returns a `DBInstanceNotFound` error once deletion completes):

```powershell
aws rds describe-db-instances `
  --db-instance-identifier lingorise-dev-db `
  --region ap-southeast-1 --profile lingorise-dev
```

```bash
aws rds describe-db-instances \
  --db-instance-identifier lingorise-dev-db \
  --region ap-southeast-1 --profile lingorise-dev
```

The asset bucket should no longer list (the stack delete removed it after Step 1 emptied it):

```powershell
aws s3 ls s3://lingorise-assets-dev-<accountId> `
  --region ap-southeast-1 --profile lingorise-dev
```

```bash
aws s3 ls s3://lingorise-assets-dev-<accountId> \
  --region ap-southeast-1 --profile lingorise-dev
```

![empty account after teardown](/images/Workshop-LingoRise/9-cleanup/empty-account.png)

{{% notice tip %}}
When all three checks come back empty, your `dev` account is clean and cost stops accruing. That is the end of the workshop — congratulations on deploying, operating, and tearing down LingoRise end to end on AWS.
{{% /notice %}}
