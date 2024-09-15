---
title: Amazon EC2 Auto Scaling
published: false
description: Handling Traffic Spikes - The Dynamic Scaling of EC2 Instances
tags: 'AWS, EC2, AutoScaling' # do not use - (hyphen) in tags
cover_image: <path to cover image> # Ex : ./assets/01-aws-site-to-site-vpn/00-aws-site-to-site-vpn-architecture-1000x420-devto.png
canonical_url: null
---

# Amazon EC2 Auto Scaling

<img src="./assets/04-aws-ec2-asg/01-asg-az.png" alt="ASG" width="80%">

<!-- ![ASG](./assets/04-aws-ec2-asg/02-asg.png) -->

<img src="./assets/04-aws-ec2-asg/02-asg.png" alt="ASG" width="60%">

## Auto Scaling Groups (ASG)

- An Auto Scaling Group is a **collection of EC2 instances** that are treated as a **logical grouping** for the purposes of **automatic scaling and management**
- ASG helps you automatically manage the scaling(in/out) of EC2 instances to meet the dynamic workload of your applications
- ASG are ***`horizontal`*** (increase or decrease in EC2 instance Count) Scaling
- There are **no additional fees** with Amazon EC2 Auto Scaling, You only pay for the AWS resources (for example, EC2 instances, EBS volumes, and CloudWatch alarms) that you use.

### Advantages of ASG
    
- **Fault tolerance**
    - Amazon EC2 Auto Scaling **can detect when an instance is unhealthy, terminate it, and launch an instance to replace it**
    - You can also configure Amazon EC2 Auto Scaling to **use multiple Availability Zones**. If one Availability Zone becomes unavailable, Amazon EC2 Auto Scaling can launch instances in another one to compensate.

- **High Availability**
    - ASG helps ensure that your application **always has the right amount of capacity to handle the current traffic demand**

- **Better Cost Management**
    - Amazon EC2 Auto Scaling can dynamically increase and decrease capacity as needed. 
    - You pay for the EC2 instances you use, you save money by launching instances when they are needed and terminating them when they aren't.

## Amazon EC2 Auto Scaling instance lifecycle

<img src="./assets/04-aws-ec2-asg/03-asg-lifecycle.png" alt="ASG Lifecycle" width="60%">






## Launch Templates Vs Launch Configuration

| Feature | Launch Template | Launch Configuration |
|---------|-----------------|----------------------|
| Introduction | Newer and more flexible | Older and less flexible |
| Modification | Can be modified after creation | Cannot be modified once created |
| Versioning | Supports versioning | Does not support versioning |
| Use Cases | EC2 instances, Spot Fleets, and ASGs | Only ASGs |
| Configuration Changes | Allows partial changes | Requires full configuration for changes |
| Instance Types | Supports multiple instance types | Limited to a single instance type |
| AMI and Instance Type | Can be specified at launch time | Must be specified when creating |
| AWS Recommendation | Recommended for new deployments | Legacy option |
| Flexibility | More flexible and versatile | Less flexible |
| Updates | Can be updated | Cannot be updated, must create new |







## References

- https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html#instancedata-data-categories

- https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html#instance-metadata-retrieval-examples

