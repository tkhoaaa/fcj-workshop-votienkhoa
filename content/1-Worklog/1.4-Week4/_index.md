---
title: "Week 4 Worklog"
date: 2026-05-11
weight: 4
chapter: false
pre: "<b>1.4. </b>"
---
### Week 4 Objectives:

* Understand the modern software delivery workflow (DevOps) on AWS through CodeCommit, CodeBuild, and CodePipeline.
* Build a CI/CD pipeline that automates testing and deployment of source code to a running environment.
* Use Amazon CloudWatch to create a monitoring dashboard and configure alarms for the workload.
* Deploy a complete frontend and backend web application through the automated pipeline with high availability in mind.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Study the DevOps delivery model on AWS and the source, build, test, deploy stages; create an AWS CodeCommit repository for the lab application. | 11/05/2026 | 11/05/2026 | [AWS CodeCommit Docs](https://docs.aws.amazon.com/codecommit/) |
| **2** | Create an AWS CodeBuild project, write `buildspec.yml`, and store build artifacts in an Amazon S3 artifact bucket. | 12/05/2026 | 12/05/2026 | [AWS CodeBuild Docs](https://docs.aws.amazon.com/codebuild/) |
| **3** | Assemble an AWS CodePipeline with source, build, and deploy stages, and review the IAM service roles each stage assumes. | 13/05/2026 | 13/05/2026 | [AWS CodePipeline Docs](https://docs.aws.amazon.com/codepipeline/) |
| **4** | Build a CloudWatch dashboard for the workload and configure alarms on CPU and error metrics with an Amazon SNS notification target. | 14/05/2026 | 14/05/2026 | [Amazon CloudWatch Docs](https://docs.aws.amazon.com/cloudwatch/) |
| **5** | Deploy a full frontend and backend web application through the pipeline end to end and verify availability after an automated release. | 15/05/2026 | 15/05/2026 | [FCAJ Workshop](https://awsstudygroup.com/) |

### Detailed Implementation:

#### 1. Map the DevOps delivery model onto AWS services

Before touching the console, I wrote out the delivery stages and matched each one to a service. **AWS CodeCommit** holds the Git source, **AWS CodeBuild** compiles and tests it, and **AWS CodePipeline** orchestrates the order in which those stages run.

The mental model I settled on was:

* **Source** -> a commit to a CodeCommit branch triggers the pipeline
* **Build** -> CodeBuild runs install, test, and package commands in a managed container
* **Deploy** -> the packaged artifact is released to the target environment

The important detail is that the pipeline itself stores nothing. Every stage passes an artifact to the next stage through **Amazon S3**, and each stage acts under an **IAM** service role rather than my own credentials.

#### 2. Define the build with buildspec.yml and an artifact bucket

I created a CodeBuild project pointing at the CodeCommit repository and described the build in `buildspec.yml` committed alongside the code. Keeping the build definition in the repository means the build changes together with the application.

```yaml
version: 0.2
phases:
  install:
    runtime-versions:
      nodejs: 20
    commands: [npm ci]
  build:
    commands: [npm test, npm run build]
artifacts:
  files: ['**/*']
  base-directory: dist
```

CodeBuild wrote the output to the S3 artifact bucket that CodePipeline provisioned, and streamed each phase into a **CloudWatch Logs** log group. When the test phase failed, the pipeline stopped at the build stage and the deploy stage never ran, which is exactly the safety property CI/CD is supposed to give.

#### 3. Assemble the pipeline and review its IAM service roles

In CodePipeline I chained the three stages and confirmed the trust relationships. The pipeline role can read the source and start builds; the CodeBuild role can write logs, read and write the artifact bucket, and reach the deployment target. Each role is separate, so a permission problem in the build stage cannot affect the source stage.

I then pushed a commit and watched the whole run without opening a terminal:

```bash
git push origin main
aws codepipeline get-pipeline-state --name lingorise-lab-pipeline
```

This was the first week where a change reached a running environment without me deploying anything by hand.

#### 4. Monitor the workload with a CloudWatch dashboard and alarms

With releases automated, the next question was whether the deployed application was healthy. I built a custom **Amazon CloudWatch** dashboard combining EC2 CPU utilization, request counts, and error metrics on one screen, then created alarms on CPU utilization and on the error metric. Both alarms publish to an **Amazon SNS** topic subscribed by email, so a breach becomes a notification instead of something I discover later.

For the end-of-week practice I deployed a complete web application, frontend and backend, through the pipeline and confirmed it stayed reachable across an automated release. This is the rehearsal for the LingoRise delivery pipeline: **AWS SAM** packaging the Node.js 20 Lambda backend into the `lingorise-dev` stack, and **AWS Amplify Hosting** running its own CI/CD build on every push to the Next.js frontend. The service names differ, but the stages, the artifact handoff, and the CloudWatch alarms are the same shape.

### AWS Service Integration This Week:

* **CodeCommit + CodePipeline:** A commit to the tracked branch became the automatic trigger for the entire release.
* **CodeBuild + S3:** Build output was published as a versioned artifact that later stages consumed.
* **CodeBuild + CloudWatch Logs:** Every build phase streamed into a log group, making failed tests readable after the fact.
* **CodePipeline + IAM:** Each stage assumed a scoped service role instead of running under a developer identity.
* **CloudWatch Alarms + SNS:** Threshold breaches on CPU and error metrics were delivered as email notifications.
* **Pipeline + Deployed Application:** The running frontend and backend were updated only through the automated path, never manually.

### Week 4 Achievements:

* Built a working CI/CD pipeline covering source, build, test, and deploy on **CodeCommit**, **CodeBuild**, and **CodePipeline**.
* Moved the build definition into the repository as `buildspec.yml` so builds are reproducible and reviewable.
* Understood how artifacts flow between pipeline stages through an **S3** bucket rather than being rebuilt each time.
* Created a custom **CloudWatch** dashboard plus CPU and error alarms wired to an **SNS** topic.
* Deployed a complete frontend and backend application through the pipeline and verified it survived an automated release.
* Connected the lab pipeline to the planned LingoRise delivery path using **AWS SAM** and **Amplify Hosting**.

### Challenges Faced:

* The first pipeline runs failed on permissions rather than on code, because the CodeBuild service role could not write to the artifact bucket. Reading the CloudWatch Logs output was faster than guessing at the policy.
* Choosing alarm thresholds was harder than creating the alarms. Too tight and every deployment triggers a notification; too loose and a real problem goes unnoticed.

### Lessons Learned and Next Steps:

* Automation only helps if it fails loudly. A pipeline that stops at a failed test and an alarm that reaches someone are the two halves of the same idea.
* Keeping the build definition and the deployment configuration in the repository makes the delivery process reviewable like any other code change.
* In the next week, I would move from individual labs to the **project kickoff**: splitting tasks across the team, writing the project proposal, drawing the system architecture diagram, and scaffolding the source code together with the infrastructure as code.
