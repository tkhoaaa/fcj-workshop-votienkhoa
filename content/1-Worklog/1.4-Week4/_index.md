---
title: "Week 4 Worklog"
date: 2026-05-11
weight: 4
chapter: false
pre: "<b>1.4. </b>"
---
### Week 4 Objectives:

* Explore AWS storage services such as S3 and EFS.
* Understand the difference between object storage and file storage.
* Connect storage services to compute workloads in meaningful ways.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Create Amazon S3 buckets and review bucket naming, object structure, and access control basics. | 11/05/2026 | 11/05/2026 | [Amazon S3 Docs](https://docs.aws.amazon.com/s3/) |
| **2** | Enable Versioning and configure Lifecycle Rules for archival behavior. | 12/05/2026 | 12/05/2026 | [S3 Versioning](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html) |
| **3** | Configure bucket access settings and review Bucket Policies for secure object access. | 13/05/2026 | 13/05/2026 | [S3 Bucket Policies](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-policy-language-overview.html) |
| **4** | Host a static website using Amazon S3 and test public object delivery. | 14/05/2026 | 14/05/2026 | [Static Website Hosting](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html) |
| **5** | Create and mount Amazon EFS to EC2 instances to test shared file storage behavior. | 15/05/2026 | 15/05/2026 | [Amazon EFS Docs](https://docs.aws.amazon.com/efs/) |

### Detailed Implementation:

#### 1. Store application and report assets in Amazon S3

I created one or more **S3 buckets** to understand how AWS object storage works. Instead of attaching storage directly to a server, S3 stores objects independently and exposes them through APIs or URLs.

I practiced:

* Uploading files
* Organizing objects by key path
* Reviewing bucket-level and object-level access settings

This helped me understand that S3 is better suited for:

* Static assets
* Backups
* Public website files
* Durable object storage

#### 2. Apply object storage lifecycle and protection features

To make storage management more realistic, I enabled:

* **Versioning** to keep multiple versions of the same object
* **Lifecycle Rules** to move older objects to colder storage classes over time

This connected storage operations with governance and cost optimization. It also showed how S3 can be used not just for storing files, but also for automating retention behavior.

#### 3. Use S3 for static website hosting

I configured the S3 bucket for **static website hosting** and uploaded simple web content. This turned the bucket into a lightweight hosting platform for HTML and static assets.

The integration path this week became:

* **Local files** -> **S3 bucket**
* **Bucket policy/public access settings** -> control browser accessibility
* **Web browser** -> retrieve static objects directly from S3

This was a practical example of how a storage service can act as a simple web delivery layer.

#### 4. Connect EC2 instances to shared file storage with EFS

Unlike S3, **Amazon EFS** provides shared file storage that multiple EC2 instances can mount at the same time. I created or reviewed an EFS file system and mounted it to EC2 to understand shared application data patterns.

This demonstrated the difference between:

* **EBS** for block storage attached to one instance
* **EFS** for shared file storage across multiple instances
* **S3** for object-based storage accessed over APIs

### AWS Service Integration This Week:

* **S3 + Bucket Policy:** Object access behavior was controlled through storage-level policies.
* **S3 + Static Website Hosting:** Storage was used directly as a content delivery mechanism for static files.
* **EC2 + EFS:** Compute instances mounted a shared network file system.
* **S3 + Lifecycle/Versioning:** Storage management was combined with protection and archival automation.
* **Storage + Cost Awareness:** Storage class and lifecycle behavior affected long-term cost efficiency.

### Week 4 Achievements:

* Built a stronger understanding of the differences between **object**, **block**, and **file** storage on AWS.
* Successfully used **S3** for object hosting and static web delivery.
* Practiced protecting data through **Versioning** and **Lifecycle Rules**.
* Mounted **EFS** to EC2 and understood how shared storage can support multiple workloads.
* Prepared storage knowledge that would later be important for private S3 access in the workshop.

### Challenges Faced:

* S3 access behavior was initially confusing because public access settings, bucket policies, and object permissions all influence the final result.
* EFS required understanding both AWS-side setup and OS-level mount configuration on EC2.

### Lessons Learned and Next Steps:

* Different storage services solve very different problems, and choosing the correct one depends on the application access pattern.
* In the next week, I would shift back toward **security and access control**, especially around IAM and permission design for AWS services.
