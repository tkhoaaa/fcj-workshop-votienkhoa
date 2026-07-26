---
title: "Worklog"
date: 2026-05-12
weight: 1
chapter: true
pre: "<b>1. </b>"
---

{{% notice info %}}
**Introduction:** This page summarizes my entire 12-week internship process at the **Workforce Bootcamp - First Cloud AI Journey (FCAJ)** program. The roadmap starts from AWS fundamentals — compute, networking, database, security, DevOps and monitoring — then moves into building and deploying the **LingoRise** serverless platform on AWS, and closes with testing, user feedback, documentation and the final internship report.
{{% /notice %}}

### 12-Week Roadmap Overview

The internship was split into two clear halves. In the first four weeks I studied AWS services individually and practiced them through hands-on labs. From Week 5 onward, I applied that foundation to a real product — **LingoRise**, an IELTS/TOEIC preparation platform — going through proposal, architecture design, development, integration, testing, user feedback, and final handover.

| Week | Main Topic | Task Summary |
| :--- | :--- | :--- |
| **Week 1** | [AWS fundamentals: EC2, VPC, and DRS](1.1-week1/) | Studying EC2, VPC and Elastic Disaster Recovery, completing basic labs, trial SAM deployment with secrets in Systems Manager Parameter Store. |
| **Week 2** | [VPC deep dive, AWS databases, and IAM](1.2-week2/) | Working through Subnets, Route Tables, IGW, NAT Gateway and NACLs; comparing Amazon RDS with DynamoDB; managing Users, Groups, Roles and Policies in IAM. |
| **Week 3** | [Workload on EC2, Nginx, S3 events, and AMI](1.3-week3/) | Reusing the VPC to run a workload, installing Nginx on Amazon Linux, triggering Lambda from S3 Event Notifications, creating an AMI for backup and repeated deployment. |
| **Week 4** | [DevOps on AWS: CI/CD and CloudWatch](1.4-week4/) | Studying CodeCommit, CodeBuild and CodePipeline, building a CI/CD pipeline, setting up CloudWatch dashboards and alarms, deploying a full web application automatically. |
| **Week 5** | [Project proposal and architecture design](1.5-week5/) | Team kick-off and task split, writing the LingoRise Project Proposal, designing the system architecture diagram, initializing the source code and infrastructure as code. |
| **Week 6** | [Core features, authentication, and API layer](1.6-week6/) | Coding user management and authorization with Cognito, developing the data-processing services, building the APIs between Frontend and Backend with CORS and API Gateway. |
| **Week 7** | [Backend services and Frontend foundation](1.7-week7/) | Building core services, entities, repositories and business logic on the Backend; setting up project structure, routing, state management and UI on the Frontend. |
| **Week 8** | [Frontend-Backend integration and security](1.8-week8/) | Connecting the Frontend to the Backend through the RESTful API, rendering real data, implementing authentication and authorization with JWT and Amazon Cognito. |
| **Week 9** | [Testing, bug fixing, and AI optimization](1.9-week9/) | Running functional and integration testing across Frontend, Backend and AWS services, fixing outstanding bugs, and tuning the AI processing flow for speed and accuracy. |
| **Week 10** | [Handover, cost cleanup, and final report](1.10-week10/) | Packaging the whole source code on GitHub, cleaning up unused AWS resources and setting cost limits, completing the Final Internship Report in English. |
| **Week 11** | [User feedback analysis and product refinement](1.11-week11/) | Collecting and analyzing user experience metrics and error rates from real testing, then refining the React frontend and fixing unexpected issues. |
| **Week 12** | [Final bug fixing, workshop report, and packaging](1.12-week12/) | Prioritizing and closing the remaining technical bugs, writing the Final Workshop Report with user evaluation data, and packaging the complete internship documentation. |

---

### How I implemented the worklog during the internship

I pursued the **Learning by Doing** method and maintained the worklog structure to combine learning theory, hands-on practice, and recording practical technical lessons:

1. **Theoretical Research**
   I read official documentation from [AWS Documentation](https://docs.aws.amazon.com/) and workshop materials to correctly understand concepts before operating.

2. **Hands-on Deployment**
   I created resources directly on AWS using both **AWS Management Console** and **AWS CLI** to understand both operational methods simultaneously.

3. **Observing connections between services**
   Each week, I didn't just record "what was created," but also paid attention to:
   * which service connects to which service
   * how a request travels from the browser to the database
   * how IAM permissions and Cognito tokens affect access
   * which configuration — network, policy, or environment variable — determines the final outcome

4. **Logging and Reflection**
   I updated the worklog weekly to record:
   * objectives
   * completed tasks
   * detailed deployment methods
   * achieved results
   * encountered difficulties
   * learned lessons and next directions

---

### Prominent AWS service groups in this roadmap

Over 12 weeks, the service groups I used and connected the most include:

* **Identity and Security:** IAM Users, Roles, Policies, MFA, Amazon Cognito
* **Networking:** VPC, Subnets, Route Tables, Internet Gateway, NAT Gateway, Security Groups, NACLs
* **Compute:** Amazon EC2, AWS Lambda (Node.js 20)
* **Application delivery:** Amazon API Gateway, AWS SAM, AWS Amplify Hosting
* **Database:** Amazon RDS for PostgreSQL, Amazon DynamoDB
* **Storage and Content Delivery:** Amazon S3, Amazon EBS, Amazon CloudFront
* **Observability:** Amazon CloudWatch, AWS CloudTrail
* **Configuration and Security:** AWS Systems Manager Parameter Store, AWS WAF
* **DevOps:** AWS CodeCommit, CodeBuild, CodePipeline
* **Governance and Cost Awareness:** AWS Budgets, Billing, Cost Explorer

These services are not learned in isolation. They are gradually connected into a complete chain serving one practical goal: building and shipping **LingoRise**, a serverless IELTS/TOEIC preparation platform, on AWS.

---

### Ultimate goal of the Worklog section

This worklog section is not only used to list the tasks I have completed during the internship, but also aims to demonstrate:

* My process of learning and applying AWS step by step
* How I connect AWS services into a real product architecture
* My troubleshooting mindset, security, and cost awareness during labs and project work
* My development in technical skills, teamwork, self-learning ability, and presentation skills

Through the 12-week worklog, I wish to clearly demonstrate the journey from an AWS beginner to being able to understand, deploy, explain, and hand over a complete serverless product built from many interconnected AWS components.
