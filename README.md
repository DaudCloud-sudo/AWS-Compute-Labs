# AWS Compute LABS

## Deploying Employee Directory Application to EC2

This lab demonstrates how to deploy a simple Employee Directory application to an Amazon EC2 instance using a user data script. The steps are part of an AWS SAA preparation course lab.

## Prerequisites

- AWS account with necessary permissions.
- Familiarity with Amazon EC2 and AWS Management Console.
- Basic understanding of IAM, VPC, and Security Groups.

## Table of Contents

1. [Launching an EC2 Instance](#launching-an-ec2-instance)
   - [Configuring Network Settings](#configuring-network-settings)
   - [Setting Up User Data](#setting-up-user-data)
2. [Viewing the Application](#viewing-the-application)
3. [Stopping the Instance](#stopping-the-instance)
4. [Resources](#resources)

---

## Launching an EC2 Instance

### Step-by-Step Guide

1. **Log in** to the **AWS Management Console** as the IAM Admin user.
2. In the **Services** search bar, type **EC2** and open the **Amazon EC2 console**.
3. In the navigation pane, choose **Instances**, then select **Launch instances**.

4. For **Name**, enter `employee-directory-app`.

   ![Launch Instances](https://github.com/user-attachments/assets/68b24572-73bd-4cd9-a5bb-34799955ee53)

5. Under **Application and OS Images (Amazon Machine Image)**, select the default **Amazon Linux 2023**.
6. For **Instance type**, choose `t2.micro`.
7. Under **Key pair (login)**, choose the `app-key-pair` that was created in exercise-3.

   ![Choose Key Pair](https://github.com/user-attachments/assets/809a2c03-e108-40df-9b1f-c3283bbebda7)

### Configuring Network Settings

8. Under **Network settings**, choose **Edit** and configure the following settings:

   - **VPC**: Select `app-vpc`.
   - **Subnet**: Choose **Public Subnet 1**.
   - **Auto-assign Public IP**: Select **Enable**.

   ![Network Settings](https://github.com/user-attachments/assets/4af63a48-4094-41b9-8db1-0fdfa0312edf)

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

   ![Instance Running](https://github.com/user-attachments/assets/0c72dd06-6a6d-4b06-bf19-e0938d4bcf9a)

## Viewing the Application

1. Select the running `employee-directory-app` instance by checking its box.
2. On the **Details** tab, copy the **Public IPv4 address**.

   ![Public IP](https://github.com/user-attachments/assets/8a67e45a-e85e-4b42-b378-622c963aa8d6)

3. Open a new browser window and paste the copied IP address. Ensure you are using **HTTP** (not HTTPS).

4. You should see an **Employee Directory** placeholder page. The application is not yet connected to a database, so no interactions are available.

   ![Employee Directory Placeholder](https://github.com/user-attachments/assets/abb683af-d53d-4669-9aca-4b24650fdd81)

## Stopping the Instance

To avoid incurring additional costs, it's best to stop the EC2 instance.

1. Return to the **AWS Management Console**.
2. Select the `employee-directory-app` instance.
3. Choose **Instance state**, then **Stop instance**, and confirm by choosing **Stop**.

   ![Stop Instance](https://github.com/user-attachments/assets/b1c6b6d6-d3d2-4f4c-b8a2-c9057ff1d003)

4. Wait for the **Instance state** to change to **Stopped**.

---

## Resources

- [AWS SAA Prep Course Exercise](https://aws.amazon.com/training/)
- [AWS Free Tier](https://aws.amazon.com/free/)
- [Amazon EC2 Pricing](https://aws.amazon.com/ec2/pricing/)

---

**Note:** This guide is part of the AWS SAA preparation course lab. Ensure you terminate all resources to avoid incurring costs.
