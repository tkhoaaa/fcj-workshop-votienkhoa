---
title: "Week 3 Worklog"
date: 2026-05-04
weight: 3
chapter: false
pre: "<b>1.3. </b>"
---
### Week 3 Objectives:

* Deep dive into EC2 instance management and Linux administration.
* Learn how storage and compute work together in AWS.
* Practice preparing an EC2 instance for web hosting and reuse.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Study EC2 purchasing options such as On-Demand and Spot, and review instance sizing considerations. | 04/05/2026 | 04/05/2026 | [EC2 Pricing Models](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-on-demand-instances.html) |
| **2** | Launch or reuse an EC2 instance in the custom VPC and prepare it for Linux administration practice. | 05/05/2026 | 05/05/2026 | EC2 Console |
| **3** | Install and configure **Nginx** on Amazon Linux, then verify access through the instance public IP or browser test. | 06/05/2026 | 06/05/2026 | [Nginx](https://nginx.org/) |
| **4** | Create, attach, mount, and expand an **EBS volume** to understand persistent block storage. | 07/05/2026 | 07/05/2026 | [Amazon EBS Docs](https://docs.aws.amazon.com/ebs/) |
| **5** | Create an **AMI** and review how it can be used for backup and repeated deployment. | 08/05/2026 | 08/05/2026 | [Create Amazon Machine Image](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html) |

### Detailed Implementation:

#### 1. Reuse the VPC environment to host a real workload

This week connected the networking knowledge from Week 2 with actual compute operations. I launched or reused an **EC2 instance** inside the custom VPC and made sure the **Security Group** allowed HTTP and SSH access as needed.

This showed a direct service relationship:

* **EC2** depends on **subnet placement**
* **Security Group** controls application reachability
* **Route table + IGW** determine whether users can access the web server

#### 2. Install Nginx and expose the web server

After connecting to the EC2 instance through SSH, I updated packages and installed **Nginx**. Then I enabled and started the service so the instance could serve web content.

The testing flow was:

* Connect to EC2 through SSH
* Install and start Nginx
* Open port `80` in the Security Group
* Access the web page through the browser using the public IP or public DNS

This exercise connected:

* **EC2 compute**
* **Linux administration**
* **Security Group firewall rules**
* **Browser/client access**

#### 3. Add persistent storage with EBS

I created a separate **EBS volume**, attached it to the EC2 instance, and mounted it inside the operating system. This demonstrated the difference between:

* The instance itself as **compute**
* The attached volume as **persistent block storage**

I also practiced expanding the EBS volume and reviewing how operating system-level filesystem changes are needed after AWS-side storage changes.

#### 4. Capture the configured server as an AMI

Once the EC2 instance was configured, I created an **Amazon Machine Image (AMI)** from it. This made it possible to reuse the same web server setup later without repeating all installation steps manually.

This connected:

* **EC2** as the source machine
* **EBS snapshots** as the image storage mechanism behind AMI creation
* **AMI** as a reusable deployment template

### AWS Service Integration This Week:

* **EC2 + VPC/Subnet:** The instance ran inside the custom network built in Week 2.
* **EC2 + Security Group:** Web and SSH access were controlled through inbound rules.
* **EC2 + EBS:** Persistent storage was attached, mounted, and expanded.
* **EC2 + AMI:** A configured machine was converted into a reusable image.
* **Linux + AWS Infrastructure:** OS-level commands were used to make cloud-side resources actually usable.

### Week 3 Achievements:

* Improved Linux administration skills while working directly on AWS compute resources.
* Successfully deployed a simple Nginx-based web server on EC2.
* Understood the lifecycle of **EBS volumes** and how cloud storage changes interact with the operating system.
* Learned how AMIs support repeatable deployment and backup strategies.
* Built a practical bridge between networking, compute, and storage services.

### Challenges Faced:

* Some tasks required understanding both the AWS console layer and the Linux operating system layer at the same time.
* Expanding storage involved more than changing volume size in AWS because the filesystem inside the instance also had to be adjusted.

### Lessons Learned and Next Steps:

* Compute services on AWS only become useful when combined correctly with storage, security, and operating system configuration.
* In the next week, I would focus more deeply on **storage services** such as **S3** and **EFS**, and how they connect with EC2-based workloads.
