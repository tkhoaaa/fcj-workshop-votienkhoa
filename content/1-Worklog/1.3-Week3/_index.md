---
title: "Week 3 Worklog"
date: 2026-05-04
weight: 3
chapter: false
pre: "<b>1.3. </b>"
---

### Week 3 Objectives:

* Reuse the custom VPC built in Week 2 to run a real workload instead of creating new network resources.
* Install and configure **Nginx** on Amazon Linux 2023 as a public web server.
* Build the first event-driven flow on AWS by connecting **Amazon S3** Event Notifications to **AWS Lambda**.
* Create an **Amazon Machine Image (AMI)** to support backup and repeated deployment.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Reuse the Week 2 custom VPC, launch an Amazon Linux 2023 instance into the public subnet, and review the existing route table and Security Group design. | 04/05/2026 | 04/05/2026 | [Amazon VPC Docs](https://docs.aws.amazon.com/vpc/) |
| **2** | Install **Nginx** with `dnf`, enable the service through `systemctl`, publish a test page, and open the HTTP inbound rule in the Security Group. | 05/05/2026 | 05/05/2026 | [Nginx on Amazon Linux](https://docs.aws.amazon.com/linux/al2023/ug/what-is-amazon-linux.html) |
| **3** | Create an S3 bucket for uploaded assets and a Lambda function, then configure an `s3:ObjectCreated:*` Event Notification to invoke it. | 06/05/2026 | 06/05/2026 | [S3 Event Notifications](https://docs.aws.amazon.com/AmazonS3/latest/userguide/NotificationHowTo.html) |
| **4** | Test the event-driven flow end to end by uploading files and reading the invocation records in **Amazon CloudWatch Logs**. | 07/05/2026 | 07/05/2026 | [Using AWS Lambda with S3](https://docs.aws.amazon.com/lambda/latest/dg/with-s3.html) |
| **5** | Create an **AMI** from the configured web server, then launch a new instance from that image to prove repeatable deployment. | 08/05/2026 | 08/05/2026 | [Create an AMI from an EC2 instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html) |

### Detailed Implementation:

#### 1. Reuse the Week 2 VPC instead of building a new network

The first decision this week was to not create a new VPC. I launched the **Amazon EC2** instance into the public subnet of the custom **Amazon VPC** from Week 2, which already had an Internet Gateway, a public route table, and a private subnet reserved for data resources.

That reuse is exactly why the network was designed carefully the week before. Because the subnet layout and routing were already correct, the only network work left this week was one Security Group rule. In the LingoRise architecture the same principle applies: the RDS for PostgreSQL instance stays in the private subnets and only the compute layer sits where traffic can reach it.

I confirmed the placement before installing anything:

* Instance in the **public subnet** with auto-assigned public IPv4
* Route `0.0.0.0/0` pointing to the **Internet Gateway**
* Security Group with SSH restricted to my own IP

#### 2. Install and configure Nginx on Amazon Linux 2023

I connected over SSH and installed **Nginx** from the Amazon Linux 2023 repositories, then enabled it so the service would survive a reboot.

```bash
sudo dnf install -y nginx
sudo systemctl enable --now nginx
echo "LingoRise web node - week 3" | sudo tee /usr/share/nginx/html/index.html
curl -I http://localhost
```

`curl` from inside the instance returned `200 OK` while the browser still timed out, which made the split between OS-level and network-level configuration very clear. The page only became reachable after I added an inbound rule for TCP `80` from `0.0.0.0/0` in the **Security Group**. I then verified from my laptop with `curl -I http://<public-ip>` so the test did not depend on browser caching.

#### 3. Connect S3 Event Notifications to Lambda

The second half of the week moved from a server I manage to an event-driven pattern with no server at all. I created an S3 bucket for uploaded assets and an **AWS Lambda** function, then added an Event Notification on the bucket for `s3:ObjectCreated:*` targeting that function.

Two details mattered:

* S3 does not use an execution role to call Lambda. The console added a **resource-based policy** on the function allowing `s3.amazonaws.com` to invoke it, scoped by source account and bucket ARN.
* The event payload is a list. The handler has to iterate `event.Records` and read `record.s3.object.key`, because one notification can carry more than one record.

This is directly the shape of the LingoRise asset pipeline. When an admin uploads a question image or a scanned DOCX to S3, the upload itself should enqueue the downstream OCR job rather than requiring a separate API call. This week was the minimal working version of that idea.

#### 4. Verify invocations and capture the server as an AMI

I uploaded several files with `aws s3 cp` and checked **Amazon CloudWatch Logs** for the function's log group. Each upload produced its own log stream entry with the bucket name and object key, which confirmed the trigger fired per object and not per batch of uploads.

Finally I created an **AMI** from the configured EC2 instance. The image is not a single file: AWS took **Amazon EBS** snapshots of the attached volumes and registered them behind the AMI ID. To prove the image was actually usable, I launched a second instance from it into the same subnet and confirmed Nginx was already running and serving the test page with no manual setup. That closed the loop between a manually configured server and a repeatable deployment artifact.

### AWS Service Integration This Week:

* **EC2 + VPC/Subnet:** The web server was launched into the Week 2 public subnet, reusing the existing routing instead of new network resources.
* **EC2 + Security Group:** HTTP reachability depended on an inbound rule for port 80, separate from the Nginx service state inside the OS.
* **S3 + Lambda:** An `s3:ObjectCreated:*` Event Notification invoked the function automatically on every upload.
* **S3 + Lambda resource policy:** S3 was granted `lambda:InvokeFunction` through a resource-based policy scoped to the bucket ARN and source account.
* **Lambda + CloudWatch Logs:** Every invocation wrote the object key to a log stream, which was the only practical way to confirm the trigger worked.
* **EC2 + EBS + AMI:** The AMI captured EBS snapshots of the instance volumes and served as the template for a second identical instance.

### Week 3 Achievements:

* Ran a real workload inside the previously built custom VPC without creating any new network components.
* Deployed a working Nginx web server on Amazon Linux 2023 and verified it externally with `curl`.
* Built a functioning event-driven pipeline where an S3 upload automatically triggers a Lambda function.
* Learned that S3 to Lambda permission flows through a resource-based policy on the function, not an execution role.
* Created an AMI and launched a new instance from it, confirming repeatable deployment.
* Mapped the S3 to Lambda pattern onto the LingoRise asset and OCR job queue design.

### Challenges Faced:

* The web page was unreachable from outside while working correctly on `localhost`, which took a Security Group fix rather than an Nginx fix.
* The first Lambda invocations failed silently because the handler assumed a single object instead of iterating `event.Records`.
* The AMI creation step took longer than expected since it waits for EBS snapshots of every attached volume to complete.

### Lessons Learned and Next Steps:

* Investing in network design early pays off later. Reusing the Week 2 VPC turned a full networking setup into a one-rule change.
* Event-driven integration removes glue code. The upload itself becomes the trigger, which is more reliable than asking a client to call an extra endpoint.
* In the next week, I would move from manual provisioning to **DevOps on AWS**, working with **AWS CodeCommit**, **AWS CodeBuild**, and **AWS CodePipeline** to build a CI/CD pipeline, plus **Amazon CloudWatch** dashboards and alarms to monitor it.
