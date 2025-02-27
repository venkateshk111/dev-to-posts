---
title: Amazon EC2 Auto Scaling
published: true
description: Handling Traffic Spikes - The Dynamic Scaling of EC2 Instances
tags: 'AWS, EC2, AutoScaling'
cover_image: ./assets/04-aws-ec2-asg/00-aws-asg-1000x420-devto.png
canonical_url: null
id: 2001804
date: '2024-10-01T06:37:25Z'
---

# Amazon EC2 Auto Scaling
<!-- 
<img src="./assets/04-aws-ec2-asg/01-asg-az.png" alt="ASG" width="80%"> -->

![ASG](./assets/04-aws-ec2-asg/01-asg-az.png)

![ASG](./assets/04-aws-ec2-asg/02-asg.png)

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

### Pricing for Amazon EC2 Auto Scaling

- There are **no additional fees with Amazon EC2 Auto Scaling**. 
- You only pay for the AWS resources (for example, EC2 instances, EBS volumes, and CloudWatch alarms) that you use.


## Understanding the Need for Amazon EC2 Auto Scaling

### Variable demand
- Consider a basic web application running on AWS.
- Purpose: Allows employees to search for conference rooms for meetings.
- Usage pattern:
    - Minimal usage at the beginning and end of the week.
    - Increased usage in the middle of the week as more employees schedule meetings.
- Graph below Displays the application’s capacity usage over the course of a week.

    <!-- <img src="./assets/04-aws-ec2-asg/04-asg-variable-demand.png" alt="Variable demand" width="80%"> -->

    ![Variable demand](./assets/04-aws-ec2-asg/04-asg-variable-demand.png)

- **Traditional Capacity Planning Options:**

  - **Option 1: Add Enough Servers to Always Meet Demand**

    ![Option 1](./assets/04-aws-ec2-asg/04-option-1.png)
    
    - Ensures sufficient capacity at all times.
    - Downside: Extra capacity remains unused on low-demand days, **increasing costs**.
    - Example: Inefficiency of buying more capacity than needed.

  - **Option 2: Handle Average Demand**

    ![Option 2](./assets/04-aws-ec2-asg/04-option-2.png)

    - Less expensive as it avoids purchasing rarely used equipment.
    - Downside: Risk of **poor customer experience** when demand exceeds capacity.
    - Example: Poor customer experience due to insufficient capacity.

- **Amazon EC2 Auto Scaling:**

  - **Option 3: Dynamic Scaling**

    ![Option 3](./assets/04-aws-ec2-asg/04-option-3.png)

    - Adds **new instances only when necessary**.
    - **Terminates instances when no longer needed**.
    - **Cost-effective**: Pay only for the instances used.
    - Provides the **best customer experience** while minimizing expenses.
    - Example: Adjusting capacity as needed with Amazon EC2 Auto Scaling.

### Balancing capacity across Availability Zones

- The following image shows an overview of multi-tier architecture deployed across three Availability Zones.

    ![ASG multi AZ](./assets/04-aws-ec2-asg/01-asg-az.png)

<!-- <img src="./assets/04-aws-ec2-asg/01-asg-az.png" alt="ASG multi AZ" width="80%"> -->

- **Instance Distribution:**
  - Amazon EC2 Auto Scaling aims to **maintain equivalent numbers of instances in each enabled Availability Zone**.
  - It attempts to launch new instances in the Availability Zone with the fewest instances.
  - If multiple subnets are chosen for the Availability Zone, a subnet is selected at random.
  - If the launch attempt fails, it tries to launch instances in another Availability Zone until successful.
  - In cases where an Availability Zone becomes unhealthy or unavailable:
    - Instance distribution may become uneven across Availability Zones.
    - Upon recovery, Amazon EC2 Auto Scaling re-balances the Auto Scaling group.
    - It launches instances in the enabled Availability Zones with the fewest instances and terminates instances elsewhere.


## Amazon EC2 Auto Scaling instance lifecycle

<!-- <img src="./assets/04-aws-ec2-asg/03-asg-lifecycle.png" alt="ASG Lifecycle" width="60%"> -->

![ASG Lifecycle](./assets/04-aws-ec2-asg/03-asg-lifecycle.png)

- **Start**: The beginning of the instance lifecycle.
- **Pending**: The instance is preparing to enter service.
  - **Pending:Wait**: A lifecycle hook where custom actions can be performed before the instance becomes active.
  - **Pending:Proceed**: The instance proceeds to the next state after the custom actions are completed.
- **InService**: The instance is now serving traffic.
  - **User puts instance into Standby**: The instance is manually moved to Standby state.
- **Standby**: The instance is not serving traffic but is still part of the Auto Scaling group.
  - **User returns instance to service**: The instance is manually moved back to InService state.
- **Terminating**: The instance is in the process of being shut down.
  - **Terminating:Wait**: A lifecycle hook where custom actions can be performed before the instance is terminated.
  - **Terminating:Proceed**: The instance proceeds to termination after the custom actions are completed.
- **Terminated**: The instance has been shut down and removed from service.
- **Detaching**: The instance is being detached from the Auto Scaling group.
- **Detached**: The instance has been successfully detached from the Auto Scaling group but not terminated.
- **End**: The lifecycle of the instance has concluded.


## Scale out, `InService`, Scale in

### Scale out

![ScaleOut](./assets/04-aws-ec2-asg/06-asg-scale-out.png)

- **Scale-Out Events**: Direct Auto Scaling group to launch and attach EC2 instances.
  - **Manual Increase**: Manually increase the size of the group.
  - **Scaling Policy**: Automatically increase the size based on demand.
  - **Scheduled Scaling**: Increase the size at a specific time.

- **Scale-Out Process**:
  - Auto Scaling group launches required EC2 instances using the launch template.
  - Instances start in Pending state.
  - **Lifecycle Hook**: Perform custom actions if added.
  - Instances fully configured and pass Amazon EC2 health checks.
  - Instances attach to Auto Scaling group and enter `InService` state.
  - Counted against desired capacity of the Auto Scaling group.

- **Load Balancer Integration**:
  - If configured to receive traffic from an Elastic Load Balancing load balancer:
    - Auto Scaling automatically registers the instance with the load balancer.
    - Instance marked as `InService` after registration.

### Instances in Service, `InService` 

- Instances remain in the **`InService`** state until one of the following occurs:
    1. **Scale-In Event**: A scale-in event occurs, and Amazon EC2 Auto Scaling chooses to terminate this instance to reduce the size of the Auto Scaling group.
    2. **Standby State**: You put the instance into a Standby state.
    3. **Detach Instance**: You detach the instance from the Auto Scaling group.
    4. **Health Check Failure**: The instance fails a required number of health checks, so it is removed from the Auto Scaling group, terminated, and replaced.

### Scale in

![ScaleIn](./assets/04-aws-ec2-asg/06-asg-scale-in.png)

- **Scale-In Events**
    - **Direct Auto Scaling Group**: Detach and terminate EC2 instances.
    - **Manual Decrease**: Manually reduce the group size.
    - **Scaling Policy**: Automatically reduce the group size based on demand.
    - **Scheduled Scaling**: Reduce the group size at a specific time.

- **Scale-In Process**
    - **Instance Termination**: Auto Scaling group terminates instances using its termination policy.
    - **Terminating State**: Instances enter the Terminating state and can't be put back into service.
    - **Lifecycle Hook**: Perform custom actions on terminating instances if added.
    - **Complete Termination**: Instances are fully terminated and enter the Terminated state.

- **Load Balancer Integration**
    - **De-registration**: If using an Elastic Load Balancing load balancer, Auto Scaling automatically de-registers terminating instances.
    - **Request Redirection**: New requests are redirected to other instances, while existing connections continue until the de-registration delay expires.


## Launch Templates

- Launch Templates are **instance configuration template** that an Auto Scaling group uses to launch EC2 instances
- Launch templates are similar to launch configurations

- Key components of launch templates:
  - AMI ID
  - Instance type
  - Key pair
  - Security groups
  - Other EC2 instance parameters

- Advantages over launch configurations:
  - Support for multiple versions
  - Ability to create subsets of parameters
  - Reusability across versions

- Versioning benefits:
  - Can create a base configuration without AMI or user data
  - Add specific AMI and user data in new versions
  - Maintain general configuration parameters separately
  - Delete testing versions when no longer needed

- Recommended over launch configurations for:
  - Access to latest features and improvements
  - Support for advanced features like:
    - Mixed Spot and On-Demand Instances
    - Multiple instance types in one Auto Scaling group

- Compatible with newer EC2 features:
  - Systems Manager parameters (AMI ID)
  - Current generation EBS Provisioned IOPS volumes (io2)
  - EBS volume tagging
  - T2 Unlimited instances
  - Capacity Reservations
  - Capacity Blocks
  - Dedicated Hosts

- Template creation:
  - All parameters are optional
  - Without an AMI specified, you can't add one when creating the Auto Scaling group
  - If AMI is specified but no instance type, you can add instance types when creating the group

### Launch Template Versioning

- The diagram below shows a single launch template with three versions:
- Each version can add or modify parameters
- Default version (Version 2 in this case) is used unless another version is specified
- Allows for flexible configurations within a single launch template

  ![Launch Template Versioning](./assets/04-aws-ec2-asg/07-asg-lt-versions.png)

#### Version 1
- Includes:
  - t2.micro instance type
  - ami-1a2b
  - subnet-1111
  - key-pair-1
- Basic configuration without security group

#### Version 2 (Default)
- Builds on Version 1
- Adds:
  - sg-2222 (security group)
- Set as the default version
- Will be used if no specific version is requested when launching an instance

#### Version 3
- Changes some parameters from previous versions
- Uses:
  - t2.medium instance type
  - ami-3c4d
- Keeps:
  - subnet-1111
  - key-pair-1 from Version 1
- Adds:
  - sg-3333 (different security group from Version 2)


### Launch Templates Vs Launch Configuration

| Feature | Launch Template | Launch Configuration |
|---------|-----------------|----------------------|
| State | Newer and more flexible | Older and less flexible |
| Modification | Can be modified after creation | Cannot be modified once created |
| Versioning | Supports versioning | Does not support versioning |
| Use Cases | EC2 instances, Spot Fleets, and ASGs | Only ASGs |
| Configuration Changes | Allows partial changes | Requires full configuration for changes |
| Instance Types | Supports multiple instance types | Limited to a single instance type |
| AMI and Instance Type | Can be specified at launch time | Must be specified when creating |
| AWS Recommendation | Recommended for new deployments | Legacy option |
| Flexibility | More flexible and versatile | Less flexible |
| Updates | Can be updated | Cannot be updated, must create new |


## ASG LifeCycle hooks

- Lifecycle hooks are like a ***`pause`*** button in Amazon EC2 Auto Scaling activity
- Lifecycle hooks puts instances into a **`wait state` to perform custom actions** during launch or before termination.
- Instances remain in a wait state until the lifecycle action is completed or the timeout period ends.
- default Timeout: 1 hour
- Use cases:
    - **Launch (Scale-Out) Hook**: **Installing software**, **run scripts**,  or **configuring the instance** before an instance goes into service.
      - Newly launched instance enters a wait state after startup
      - Run scripts to download and install necessary software
      - Ensure instance is fully ready before receiving traffic
      - Use `complete-lifecycle-action` command to continue
    
    - **Terminate (Scale-In) Hook**: **Downloading logs**, **backup data**, or perform any **clean-up tasks** before an instance is terminated.
      - Instance pauses before termination
      - Send notification via Amazon EventBridge
      - Allows actions like invoking AWS Lambda functions or connecting to the instance
      - Opportunity to download logs or other data before full termination

- Examples:
  - Controlling instance registration with Elastic Load Balancing
  - Ensuring bootstrap scripts complete successfully
  - Verifying applications are ready to accept traffic
  - Registering instances to the load balancer after lifecycle hook completion


  ![LifeCycle hooks](./assets/04-aws-ec2-asg/05-asg-lifecycle-hooks.png)


- **Scale-Out Event**:
  - Instances launch and start in **Pending** state.
  - With `autoscaling:EC2_INSTANCE_LAUNCHING` hook:
    - Move to **Pending:Wait** state.
    - Complete lifecycle action.
    - Move to **Pending:Proceed** state.
  - Fully configured instances attach to Auto Scaling group and enter **InService** state.

- **Scale-In Event**:
  - Instances terminate and detach from Auto Scaling group, entering **Terminating** state.
  - With `autoscaling:EC2_INSTANCE_TERMINATING` hook:
    - Move to **Terminating:Wait** state.
    - Complete lifecycle action.
    - Move to **Terminating:Proceed** state.
  - Fully terminated instances enter **Terminated** state.

## ASG Warm Pool

- *warm pool* helps to **decreases latency for applications with long boot times**.
- *warm pool* **ensures instances are ready to quickly start serving application traffic during a scale-out event**.
- Instances in the warm pool count toward the desired capacity when they leave the pool (known as a warm start).
- **Instance States**: Instances in the warm pool can be in one of three states: Stopped, Running, or Hibernated.
  - **Stopped**: Minimizes costs, pay only for volumes and Elastic IP addresses.
  - **Hibernated**: Saves RAM contents to Amazon EBS root volume, pay for EBS volumes and Elastic IP addresses.
  - **Running**: Discouraged to avoid unnecessary charges.

- **Warm Pool Size**:
  - **Default size**: maximum capacity - desired capacity = Default warm pool size
    - Example: if maximum capacity = 10 and Desired capacity = 6 than Warm pool size = 10-6 = 4.
  - **Custom size**: Use `MaxGroupPreparedCapacity` option to set a custom value.
    - Example : Maximum capacity = 20, Desired capacity = 6, custom capacity = 8 than Warm pool size = 2.

- **Lifecycle Hooks**:
  - Put instances into a wait state for custom actions during launch or termination.
  - Delay instances from being stopped or hibernated until they finish initializing.

- **Instance Reuse Policy**:
  - Default: Terminates instances when scaling in and launches new instances into the warm pool.
  - Reuse Policy: Return instances to the warm pool instead of terminating them, ensuring the pool is not over-provisioned.


## ASG Scaling

- ***Scaling*** in AWS Auto Scaling Group (ASG) refers to the **automatic adjustment of the number of EC2 instances** in response to changes in demand for your application. 
- AWS ASG ensures that your application has the right amount of compute capacity at any given time by **scaling out** (increasing instances) or **scaling in** (decreasing instances) based on predefined conditions or policies
- **Scaling Out**: Adding more instances to handle an increase in traffic or workload.
- **Scaling In** : Reducing the number of instances when the demand decreases, saving costs.


### Types of Scaling

### 1. Manual Scaling
  - Manually adjust the number of instances in the ASG.
  - Useful for predictable workloads or during testing.

    ![Manual Scaling](./assets/04-aws-ec2-asg/08-asg-ms.png)

### 2. Automatic Scaling

- ### Scheduled Scaling:
  - **How it Works**: Scale based on a schedule.Scales the number of instances up or down at predetermined times.
  - **Example**: 
    - Maintain 4 desired instance 6 max and 2 min at specific time of the day.
    - Add 5 instances at 8 AM every weekday, remove 5 instances at 6 PM.
  - **Use Case**: 
    - Ideal for predictable load changes, such as daily or weekly traffic patterns.

    ![Scheduled Scaling](./assets/04-aws-ec2-asg/09-asg-schs.png)

- ### Predictive Scaling:
   - Uses machine learning to analyze historical load patterns.
   - Proactively scales capacity up or down based on predictions.

      ![Predictive Scaling](./assets/04-aws-ec2-asg/10-asg-pc.png)

- ### Dynamic Scaling:
    - Automatically scale based on real-time metrics.
    - **Uses CloudWatch alarms** to trigger scaling actions.
    - 3 Types of Dynamic Scaling Policies
      1. Simple Scaling
      2. Step Scaling
      3. Target Tracking Scaling

        
      ![Dynamic Scaling](./assets/04-aws-ec2-asg/13-asg-dynamic.png)

#### 1. Simple Scaling Policy
   - **How it Works**: Adds or removes a fixed number of instances when a specific *metric* breaches a threshold.
   - **Example**: Add 1 instance when CPU utilization exceeds 75%, remove 1 instance when CPU falls below 30%.
   - **Use Case**: Suitable for basic scaling needs.

     ![Simple Scaling](./assets/04-aws-ec2-asg/08-asg-sc.png)

#### 2. Step Scaling Policy
   - **How it Works**: Scales in steps based on how much the monitored *metric* deviates from the threshold.
   - **Example**: 
     - CPU usage > 75%, add 2 instances.
     - CPU usage > 80%, add 4 instances.
     - CPU usage < 30%, remove 1 instance.
   - **Use Case**: Ideal for handling variable demand with predefined increments.

     ![Step Scaling](./assets/04-aws-ec2-asg/11-asg-step.png)

#### 3. Target Tracking Scaling Policy
   - **How it Works**: Adjusts the instance count to maintain a target value for a specific CloudWatch *metric* (e.g., CPU utilization).
   - **Example**: Set a target CPU utilization to 50%. ASG will scale instances to maintain this target.
   - **Use Case**: Best for maintaining consistent performance.
    
      ![alt text](./assets/04-aws-ec2-asg/12-asg-target.png)


### Comparison of Scaling Policies

| **Scaling Policy**         | **Trigger**                             | **Scaling Behavior**                     | **Use Case**                          |
|----------------------------|-----------------------------------------|------------------------------------------|---------------------------------------|
| **Scheduled Scaling**       | Predefined time intervals               | Scaling occurs at specified times        | Predictable workload patterns         |
| **Simple Scaling**          | Metric breaches a threshold             | Fixed increase or decrease in instances  | Basic threshold-based scaling         |
| **Step Scaling**            | Metric exceeds/falls below thresholds   | Scales in increments based on metric deviations | Variable traffic with sharp demand changes |
| **Target Tracking**         | CloudWatch metric reaching a target     | Scales to maintain a target metric       | Continuous, steady-state applications |


## References

- https://aws.amazon.com/ec2/autoscaling/
- https://docs.aws.amazon.com/autoscaling/ec2/userguide/auto-scaling-benefits.html

- https://aws.amazon.com/blogs/compute/scaling-your-applications-faster-with-ec2-auto-scaling-warm-pools/

