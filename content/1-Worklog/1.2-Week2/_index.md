---
title: "Week 2 Worklog"
date: 2026-04-27
weight: 2
chapter: false
pre: "<b>1.2. </b>"
---
### Week 2 Objectives:

* Master VPC components and networking on AWS.
* Build a custom network topology for cloud workloads.
* Understand how compute resources connect to the Internet and to private subnets.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| **1** | Research VPC components such as Subnets, Route Tables, Internet Gateway, NAT Gateway, and Network ACLs. | 27/04/2026 | 27/04/2026 | [Amazon VPC Docs](https://docs.aws.amazon.com/vpc/) |
| **2** | Create a custom VPC with CIDR `10.0.0.0/16`, then define public and private subnets across Availability Zones. | 28/04/2026 | 28/04/2026 | [VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) |
| **3** | Configure an Internet Gateway and a NAT Gateway to control outbound connectivity for different subnet types. | 29/04/2026 | 30/04/2026 | [NAT Gateway Docs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) |
| **4** | Create Security Groups and Network ACLs to compare stateful and stateless security behavior. | 01/05/2026 | 02/05/2026 | [Security in VPC](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Security.html) |
| **5** | Launch test EC2 instances into public and private subnets and verify connectivity. | 03/05/2026 | 03/05/2026 | EC2 and VPC Console |

### Detailed Implementation:

#### 1. Design the VPC before launching workloads

Instead of using the default VPC, I created a custom **VPC** with CIDR `10.0.0.0/16`. I then divided it into:

* **Public subnet** for resources that need Internet access
* **Private subnet** for resources that should not be directly reachable from the Internet

This was the first time I designed the network layer intentionally rather than accepting the AWS default configuration.

#### 2. Build Internet access using Internet Gateway and NAT Gateway

To make the public subnet reachable from the Internet, I attached an **Internet Gateway (IGW)** to the VPC and updated the public route table with a default route `0.0.0.0/0` pointing to the IGW.

For the private subnet, I created a **NAT Gateway** in the public subnet. Then I updated the private route table so private instances could reach the Internet for updates or package installation without exposing them to inbound public access.

This created a practical service chain:

* **Private EC2** -> **Private Route Table** -> **NAT Gateway** -> **Internet Gateway** -> Internet

#### 3. Connect EC2 with network and security controls

I launched EC2 instances into both subnet types to test the architecture:

* One instance in the **public subnet**
* One instance in the **private subnet**

The public instance could be reached via SSH, while the private instance required a controlled path for outbound traffic. This demonstrated how **EC2**, **subnets**, **route tables**, and **gateways** work together in a real environment.

#### 4. Apply network security at multiple layers

I configured:

* **Security Groups** at the instance level
* **Network ACLs** at the subnet level

This allowed me to compare:

* **Stateful filtering** with Security Groups
* **Stateless filtering** with NACLs

By testing allowed and blocked traffic, I gained a practical understanding of which layer is better suited for fine-grained instance access and which is useful for broader subnet-level control.

### AWS Service Integration This Week:

* **VPC + Subnets:** The VPC was segmented into public and private network zones.
* **Subnets + Route Tables:** Traffic behavior was defined through different route associations.
* **VPC + Internet Gateway:** Public subnet resources were able to reach the Internet.
* **Private Subnet + NAT Gateway:** Private EC2 instances gained outbound-only Internet access.
* **EC2 + Security Groups + NACLs:** Access control was implemented at both instance and subnet levels.

### Week 2 Achievements:

* Successfully built a custom AWS network topology instead of relying on the default VPC.
* Understood how **IGW** and **NAT Gateway** serve different connectivity purposes.
* Practiced placing workloads into the correct subnet based on exposure requirements.
* Learned the difference between **stateful** and **stateless** network security controls.
* Created a strong networking foundation for all later services including load balancing, storage access, and private endpoints.

### Challenges Faced:

* Route table behavior was initially difficult to visualize because each subnet can behave differently depending on its associated route.
* It took time to understand that a private subnet can still access the Internet indirectly through a NAT Gateway without becoming publicly accessible.

### Lessons Learned and Next Steps:

* AWS networking is the backbone of cloud architecture, and every later service depends on proper subnet and routing design.
* In the next week, I would continue by deepening compute knowledge through **EC2**, **Linux administration**, and **EBS storage management** inside this networked environment.
