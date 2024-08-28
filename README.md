# AWS LABS

# Deploying Employee Directory Application to EC2

This project demonstrates how to deploy a simple Employee Directory application to an Amazon EC2 instance using a user data script. The steps are part of an AWS SAA preparation course lab.

## Prerequisites

- AWS account with necessary permissions.
- Familiarity with Amazon EC2 and AWS Management Console.
- Basic understanding of IAM, VPC, and Security Groups.

## Table of Contents

1. [Launching an EC2 Instance](#launching-an-ec2-instance)
   - [Creating a Key Pair](#creating-a-key-pair)
   - [Configuring Network Settings](#configuring-network-settings)
   - [Setting Up User Data](#setting-up-user-data)
2. [Viewing the Application](#viewing-the-application)
3. [Cleaning Up](#cleaning-up)
4. [Resources](#resources)

---

## Launching an EC2 Instance

### Launching an EC2 instance that uses a role

1. Log in to the **AWS Management Console** as the IAM Admin user.
2. In the **Services** search bar, search for **EC2**, and open the service.
3. In the navigation pane, under **Instances**, choose **Instances**.
4. Choose **Launch instances**.
5. For **Name**, use `employee-directory-app`.

    ![image](https://github.com/user-attachments/assets/68b24572-73bd-4cd9-a5bb-34799955ee53)

### Creating a Key Pair

6. Under **Application and OS Images (Amazon Machine Image)**, choose the default **Amazon Linux 2023**.
7. Under **Instance type**, select `t2.micro`.
8. Under **Key pair (login)**, choose **Create a new key pair**.
9. For **Key pair name**, enter `app-key-pair`. Choose **Create key pair**. The required `.pem` file should automatically download.

   ![image](https://github.com/user-attachments/assets/809a2c03-e108-40df-9b1f-c3283bbebda7)

### Configuring Network Settings

10. Under **Network settings**, choose **Edit**.
    - Keep the default VPC selection.
    - **Subnet**: Choose the first subnet in the dropdown list.
    - **Auto-assign Public IP**: Enable.

    ![image](https://github.com/user-attachments/assets/4af63a48-4094-41b9-8db1-0fdfa0312edf)

11. Under **Firewall (security groups)**, choose **Create security group**.
12. Use `app-sg` for the **Security group name** and **Description**.
13. Under **Inbound security groups rules**, choose **Remove** above the SSH rule.
14. Choose **Add security group rule**. For **Type**, choose **HTTP**. Under **Source type**, choose **Anywhere**.

   ![image](https://github.com/user-attachments/assets/a6f333be-095b-426a-ad75-d9e2c2b735fd)

### Setting Up User Data

15. Expand **Advanced details** and under **IAM instance profile**, choose `S3DynamoDBFullAccessRole`.
16. In the **User data** box, paste the following script:

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

   ![image](https://github.com/user-attachments/assets/b0502842-25d9-4e2e-b837-60b041fd62b0)

17. Replace `<INSERT REGION HERE>` with your AWS region, e.g., `us-west-2`.

18. Choose **Launch instance**.

19. Choose **View all instances**.

20. Wait for the **Instance state** to change to **Running** and the **Status check** to change to **2/2 checks passed**.

    ![image](https://github.com/user-attachments/assets/0c72dd06-6a6d-4b06-bf19-e0938d4bcf9a)

## Viewing the Application

1. Select the instance by checking its box.
2. On the **Details** tab, copy the **Public IPv4 address**.
3. Open a new browser window, paste the IP address, and ensure you are using **HTTP** (not HTTPS).
4. You should see an **Employee Directory** placeholder. The application isn't connected to a database yet.

    ![image](https://github.com/user-attachments/assets/abb683af-d53d-4669-9aca-4b24650fdd81)

## Cleaning Up

1. Go back to the **AWS Management Console**.
2. Select the **employee-directory-app** instance.
3. Choose **Instance state**, select **Stop instance**, and choose **Stop**.
4. Once the instance is in the **Stopped** state, choose **Instance state** again, select **Terminate instance**, and choose **Terminate**.

## Resources

- AWS SAA Prep Course Exercise
- [AWS Free Tier](https://aws.amazon.com/free/)
- [Amazon EC2 Pricing](https://aws.amazon.com/ec2/pricing/)

---

**Note:** This guide is based on the AWS SAA preparation course lab. Ensure you terminate all resources to avoid incurring costs.
