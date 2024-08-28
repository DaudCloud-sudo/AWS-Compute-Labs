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

### Step 1: Launching an EC2 instance that uses a role

1. Log in to the **AWS Management Console** as the IAM Admin user.
2. In the **Services** search bar, search for **EC2**, and open the service.

    ![EC2 Service](path/to/your/image1.png)

3. In the navigation pane, under **Instances**, choose **Instances**.
4. Choose **Launch instances**.
5. For **Name**, use `employee-directory-app`.

    ![Instance Configuration](path/to/your/image2.png)

### Step 2: Creating a Key Pair

6. Under **Application and OS Images (Amazon Machine Image)**, choose the default **Amazon Linux 2023**.
7. Under **Instance type**, select `t2.micro`.
8. Under **Key pair (login)**, choose **Create a new key pair**.
9. For **Key pair name**, enter `app-key-pair`. Choose **Create key pair**. The required `.pem` file should automatically download.

    ![Key Pair Creation](path/to/your/image3.png)

### Step 3: Configuring Network Settings

10. Under **Network settings**, choose **Edit**.
    - Keep the default VPC selection.
    - **Subnet**: Choose the first subnet in the dropdown list.
    - **Auto-assign Public IP**: Enable.

    ![Network Settings](path/to/your/image4.png)

### Step 4: Setting Up Security Groups

11. Under **Firewall (security groups)**, choose **Create security group**.
12. Use `app-sg` for the **Security group name** and **Description**.
13. Under **Inbound security groups rules**, choose **Remove** above the SSH rule.
14. Choose **Add security group rule**. For **Type**, choose **HTTP**. Under **Source type**, choose **Anywhere**.

    ![Security Group Configuration](path/to/your/image5.png)

### Step 5: Setting Up User Data

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

    ![User Data Script](path/to/your/image6.png)

17. Replace `<INSERT REGION HERE>` with your AWS region, e.g., `us-west-2`.

18. Choose **Launch instance**.

19. Choose **View all instances**.

20. Wait for the **Instance state** to change to **Running** and the **Status check** to change to **2/2 checks passed**.

    ![Instance Running](path/to/your/image7.png)

## Viewing the Application

1. Select the instance by checking its box.
2. On the **Details** tab, copy the **Public IPv4 address**.

    ![Instance Details](path/to/your/image8.png)

3. Open a new browser window, paste the IP address, and ensure you are using **HTTP** (not HTTPS).
4. You should see an **Employee Directory** placeholder. The application isn't connected to a database yet.

    ![Employee Directory App](path/to/your/image9.png)

## Cleaning Up

1. Go back to the **AWS Management Console**.
2. Select the **employee-directory-app** instance.
3. Choose **Instance state**, select **Stop instance**, and choose **Stop**.
4. Once the instance is in the **Stopped** state, choose **Instance state** again, select **Terminate instance**, and choose **Terminate**.

    ![Terminate Instance](path/to/your/image10.png)

## Resources

- AWS SAA Prep Course Lab
- [AWS Free Tier](https://aws.amazon.com/free/)
- [Amazon EC2 Pricing](https://aws.amazon.com/ec2/pricing/)

---

**Note:** This guide is based on the AWS SAA preparation course lab. Ensure you terminate all resources to avoid incurring costs.
