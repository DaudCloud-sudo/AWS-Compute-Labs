# AWS Compute LABS

## LAB 1: Deploying Employee Directory Application to EC2

This lab demonstrates how to deploy a simple Employee Directory application to an Amazon EC2 instance using a user data script. The steps are part of an AWS SAA preparation course lab.

## Table of Contents

1. [Lab 1: Deploying Employee Directory Application to EC2](#lab-1-deploying-employee-directory-application-to-ec2)
   - [Prerequisites](#prerequisites)
   - [Launching an EC2 Instance](#launching-an-ec2-instance)
     - [Using a Custom VPC](#using-a-custom-vpc)
     - [Configuring EC2 Settings](#configuring-ec2-settings)
     - [Configuring Network Settings](#configuring-network-settings)
     - [Setting Up User Data](#setting-up-user-data)
   - [Viewing the Application](#viewing-the-application)
   - [Stopping the Instance](#stopping-the-instance)
   - [Resources](#resources)

2. [Lab 2: Load Balancing and Auto Scaling on AWS to EC2](#lab-1-load-balancing-and-auto-scaling-on-aws-to-ec2)
   - [Introduction](#introduction)
   - [Prerequisites](#prerequisites)
   - [Again Launching an EC2 Instance](#again-launching-an-ec2-instance)
   - [Creating the Application Load Balancer](#creating-the-application-load-balancer)
   - [Creating the Launch Template](#creating-the-launch-template)
   - [Creating the Auto Scaling Group](#creating-the-auto-scaling-group)
   - [Testing the Application](#testing-the-application)
   - [Deleting the Resources](#deleting-the-resources)
   - [Conclusion](#conclusion)
   
---

## Prerequisites

- AWS account with necessary permissions.
- Familiarity with Amazon EC2 and AWS Management Console.
- Basic understanding of IAM, VPC, and Security Groups.

## Launching an EC2 Instance

### Using a Custom VPC

If you wish to use a custom VPC, follow the instructions in the [Custom VPC Setup Guide](https://github.com/DaudCloud-sudo/AWS-Networking-and-Content-Delivery-Labs/tree/main). Otherwise, you can use the default VPC provided by AWS for simplicity.

### Configuring EC2 Settings

1. **Log in** to the **AWS Management Console** as the IAM Admin user.
2. In the **Services** search bar, type **EC2** and open the **Amazon EC2 console**.
3. In the navigation pane, under **Instances**, choose **Instances**, then select **Launch instances**.
4. For **Name**, enter `employee-directory-app`.
5. Under **Application and OS Images (Amazon Machine Image)**, select the default **Amazon Linux 2023**.
6. For **Instance type**, choose `t2.micro`.
7. Under **Key pair (login)**, choose the `app-key-pair`.

### Configuring Network Settings

8. Under **Network settings**, choose **Edit** and configure the following settings:

   - **VPC**: Select `app-vpc` if using a custom VPC, or the default VPC if not.
   - **Subnet**: Choose **Public Subnet 1**.
   - **Auto-assign Public IP**: Select **Enable**.

   ![image](https://github.com/user-attachments/assets/796f6751-91ed-4165-adb8-e387a078c3f5)
   
9. Under **Firewall (security groups)**, choose **Create security group**. Use `web-security-group` as the **Security group name** and update **Description** to "Enable HTTP access".

10. Under **Inbound security groups rules**, choose **Remove** above the **SSH** rule.
11. Choose **Add security group rule**, set **Type** to **HTTP**, and under **Source type**, select **Anywhere**.

12. Choose **Add security group rule**, set **Type** to **HTTPS**, and under **Source type**, select **Anywhere**.

   ![Security Group Rules](https://github.com/user-attachments/assets/a6f333be-095b-426a-ad75-d9e2c2b735fd)

### Setting Up User Data

13. Expand **Advanced details**, and under **IAM instance profile**, select `S3DynamoDBFullAccessRole`.

14. In the **User data** box, paste the following script:

    ```bash
    #!/bin/bash -ex
    wget https://aws-tc-largeobjects.s3-us-west-2.amazonaws.com/DEV-AWS-MO-GCNv2/FlaskApp.zip
    unzip FlaskApp.zip
    cd FlaskApp/
    yum -y install python3-pip
    pip install -r requirements.txt
    yum -y install stress
    export PHOTOS_BUCKET=${SUB_PHOTOS_BUCKET}
    export AWS_DEFAULT_REGION=<INSERT REGION HERE>
    export DYNAMO_MODE=on
    FLASK_APP=application.py /usr/local/bin/flask run --host=0.0.0.0 --port=80
    ```

   ![User Data](https://github.com/user-attachments/assets/b0502842-25d9-4e2e-b837-60b041fd62b0)

15. Update the line `export AWS_DEFAULT_REGION=<INSERT REGION HERE>` with your AWS region, for example:

    ```bash
    export AWS_DEFAULT_REGION=us-west-2
    ```

16. Choose **Launch instance**.

17. Select **View all instances**.

18. Wait for the **Instance state** to change to **Running** and the **Status check** to update to **2/2 checks passed**.

   ![image](https://github.com/user-attachments/assets/2ee63031-b938-4ce5-b407-9f4ecb477b6a)

## Viewing the Application

1. Select the running `employee-directory-app` instance by checking its box.
2. On the **Details** tab, copy the **Public IPv4 address**.
3. Open a new browser window and paste the copied IP address. Ensure you are using **HTTP** (not HTTPS).

4. You should see an **Employee Directory** placeholder page. The application is not yet connected to a database, so no interactions are available.

![image](https://github.com/user-attachments/assets/c538aa61-7645-4340-94bd-d866f162cb7c)
![image](https://github.com/user-attachments/assets/70a9f8b4-877b-40e6-b48d-58e0ddd9eed3)

## Stopping the Instance

To avoid incurring additional costs, it's best to stop the EC2 instance.

1. Return to the **AWS Management Console**.
2. Select the `employee-directory-app` instance.
3. Choose **Instance state**, then **Stop instance**, and confirm by choosing **Stop**.
4. Wait for the **Instance state** to change to **Stopped**.

---

## Resources

- [AWS SAA Prep Course Exercise](https://aws.amazon.com/training/)
- [AWS Free Tier](https://aws.amazon.com/free/)
- [Amazon EC2 Pricing](https://aws.amazon.com/ec2/pricing/)
- [Custom VPC Setup Guide](https://github.com/DaudCloud-sudo/AWS-Networking-and-Content-Delivery-Labs/tree/main)

---

**Note:** This guide is part of the AWS SAA preparation course lab. Ensure you terminate all resources to avoid incurring costs.

---

## Lab 2: Load Balancing and Auto Scaling on AWS to EC2

## Introduction 

This second lab demonstrates how to set up a load-balanced, scalable environment for an application on AWS. The steps involve launching a new EC2 instance, configuring an Application Load Balancer (ALB), creating a launch template, setting up an Auto Scaling group, and testing the scalability of the setup.

## Prerequisites

- Access to AWS Management Console with Admin privileges.
- An existing VPC (`app-vpc`) with at least two public subnets. Otherwise, Follow the instructions in the [Custom VPC Setup Guide](https://github.com/DaudCloud-sudo/AWS-Networking-and-Content-Delivery-Labs/tree/main).
- Basic understanding of AWS EC2, ALB, and Auto Scaling concepts.

## Task 1: Again Launching an EC2 Instance

1. **Log in to the AWS Management Console** as your Admin user.
2. **Navigate to EC2 Dashboard**: Search for and open EC2 in the AWS Management Console.
3. **Select an Existing Instance**: In the navigation pane, choose **Instances**. Select the check box for `employee-directory-app` (Stopped state).
4. **Launch a New Instance**:
   - Choose **Actions** > **Image and templates** > **Launch more like this**.
   - Update **Name** field by appending `elb`.
   - Choose **Key pair name**: `app-key-pair`.
   - Enable **Auto-assign Public IP**.
   - Click **Launch instance** and choose **View all instances**.
5. **Verify Instance Launch**:
   - Wait until the instance state changes to **Running** and the status check shows **2/2 checks passed**.
   - Copy the **Public IPv4 address** from the **Details** tab and open it in a browser using `http://<Public_IP>`.
  
   ![image](https://github.com/user-attachments/assets/e32c7033-e929-4c81-92fb-59513b0e891b)

## Task 2: Creating the Application Load Balancer

1. **Navigate to Load Balancers**: In the EC2 console navigation pane, under **Load Balancing**, choose **Load Balancers**.
2. **Create Load Balancer**:
   - Click **Create Load Balancer** and select **Application Load Balancer**.
   - Configure the settings:
     - **Load balancer name**: `app-alb`
     - **VPC**: `app-vpc`
     - **Mappings**: Select both availability zones (e.g., `us-eat-2a` and `us-east-2b`).
     - **Security groups**: Remove default and **Create new security group**:
       - **Security group name**: `load-balancer-sg`
       - **Description**: HTTP and HTTPS access
       - **Inbound rules**: Allow **HTTP** from **Anywhere-IPv4**
3. **Create Target Group**:
   - Choose **Create target group** and configure:
     - **Target group name**: `app-target-group`
     - **Health checks**:
       - Healthy threshold: 2
       - Unhealthy threshold: 5
       - Timeout: 30
       - Interval: 40
   - Register the `employee-directory-app-elb` instance.
   
   ![image](https://github.com/user-attachments/assets/02e245b9-8e08-41bd-bbbf-4d262a3ff3cc)

4. **Finalize Load Balancer**: Click **Create load balancer** and wait for it to become **Active**. 

![image](https://github.com/user-attachments/assets/69a1b403-c202-41a1-a1ce-37d23c729f82)

5. Copy the DNS name to a text editor and prepend `http://`.

![image](https://github.com/user-attachments/assets/b7fd3ed6-5002-496a-b202-6d6ebe655999)
![image](https://github.com/user-attachments/assets/869494a0-1523-47ab-8c1e-adccc07d6347)

## Task 3: Creating the Launch Template

1. **Navigate to Launch Templates**: In the EC2 console, choose **Launch Templates**.
2. **Create a New Launch Template**:
   - **Launch template name**: `app-launch-template`
   - **Template version description**: A web server for the employee directory application
   - **Instance type**: `t2.micro`
   - **Key pair name**: `app-key-pair`
   - **Security groups**: `web-security-group`
3. **Advanced Details**:
   - **IAM instance profile**: `S3DynamoDBFullAccessRole`
   - **User data**:
     ```bash
     #!/bin/bash -ex
     wget https://aws-tc-largeobjects.s3-us-west-2.amazonaws.com/DEV-AWS-MO-GCNv2/FlaskApp.zip
     unzip FlaskApp.zip
     cd FlaskApp/
     yum -y install python3-pip
     pip install -r requirements.txt
     yum -y install stress
     export PHOTOS_BUCKET=employee-photo-bucket-daud-42
     export AWS_DEFAULT_REGION=us-east-1
     FLASK_APP=application.py /usr/local/bin/flask run --host=0.0.0.0 --port=80
     ```
   - Replace placeholders with your bucket and region details.
4. **Create Template**: Click **Create launch template**.

![image](https://github.com/user-attachments/assets/0399391e-df2d-4903-a57c-7c86aef0e817)

## Task 4: Creating the Auto Scaling Group

1. **Navigate to Auto Scaling Groups**: In the EC2 console, choose **Auto Scaling Groups**.
2. **Create Auto Scaling Group**:
   - **Auto Scaling group name**: `app-asg`
   - **Launch template**: `app-launch-template`
   - **VPC**: `app-vpc`
   - **Availability Zones and subnets**: Select the appropriate Public subnets.
3. **Configure Scaling Policies**:
   - **Desired capacity**: 2
   - **Minimum capacity**: 2
   - **Maximum capacity**: 4
   - **Scaling policies**: Target tracking scaling policy with a target value of 60.
4. **Add Notifications**:
   - Create an SNS topic `app-sns-topic` and subscribe with your email.
5. **Finalize**: Click **Create Auto Scaling group**.

![image](https://github.com/user-attachments/assets/0e1ad51e-a690-403d-93a1-799d40c03ab7)

## Task 5: Testing the Application

1. **Navigate to Target Groups**: In the EC2 console, choose **Target Groups** and select `app-target-group`.
2. **Verify Instance Health**: Ensure all instances are **Healthy**.
3. **Access the Application**:
   - Copy the DNS name of the load balancer, prepend `http://`, and open it in a browser.
   - Append `/info` to see which instance is handling the request.
4. **Stress Test**:
   - Use a stress tool to simulate load for 10 minutes and observe scaling behavior.
  
![image](https://github.com/user-attachments/assets/cab733ec-4e6e-4783-9859-cc8201ebcee9)

## Task 6: Deleting the Resources

1. **Delete Auto Scaling Group**:
   - Navigate to **Auto Scaling Groups**, select `app-asg`, and delete.
2. **Delete Load Balancer**:
   - Navigate to **Load Balancers**, select `app-alb`, and delete.
3. **Delete Target Group**:
   - Navigate to **Target Groups**, select `app-target-group`, and delete.
4. **Terminate EC2 Instances**:
   - Navigate to **Instances**, select all instances, and terminate.
5. **Delete DynamoDB Table**:
   - Navigate to DynamoDB, select the table `Employees`, and delete.
6. **Delete S3 Bucket**:
   - Empty and delete the S3 bucket used for the application.
7. **Delete VPC Resources**:
   - Remove associated subnets, route tables, and internet gateways before deleting the VPC.
8. **Delete Security Groups**:
   - Navigate to **Security Groups** and delete the security groups created for this lab.
9. **Delete IAM Role**:
   - Navigate to IAM, select `S3DynamoDBFullAccessRole`, and delete.
10. **Delete SNS Topic**:
    - Navigate to SNS, select the topic `app-sns-topic`, and delete.

By following this lab, you have successfully set up a scalable, load-balanced application environment on AWS. You learned how to create an EC2 instance, configure a load balancer, set up a launch template, create an Auto Scaling group, and delete resources after testing.
