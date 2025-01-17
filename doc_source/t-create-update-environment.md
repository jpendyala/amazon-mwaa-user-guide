# Troubleshooting: Creating and updating an Amazon MWAA environment<a name="t-create-update-environment"></a>

The topics on this page contains errors you may encounter when creating and updating an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment and how to resolve these errors\.

**Contents**
+ [Updating requirements\.txt](#troubleshooting-reqs)
  + [I specified a new version of my `requirements.txt` and it's taking more than 20 minutes to update my environment](#t-requirements)
+ [Create bucket](#troubleshooting-create-bucket)
  + [I can't select the option for S3 Block Public Access settings](#t-create-bucket)
+ [Create environment](#troubleshooting-create-environment)
  + [I tried creating an environment and it's stuck in the "Creating" state](#t-stuck-failure)
  + [I tried to create an environment but it shows the status as "Create failed"](#t-create-environ-failed)
  + [I tried to select a VPC and received a "Network Failure" error](#t-network-failure)
  + [I received a service, partition, or resource "must be passed" error](#t-service-partition)
  + [I tried to create an environment and it shows the status as "Available" but when I try to access the Airflow UI an "Empty Reply from Server" or "502 Bad Gateway" error is shown](#t-create-environ-empty-reply)
+ [Update environment](#troubleshooting-update-environment)
  + [I tried changing the environment class but the update failed](#t-rollback-billing-failure)
+ [Access environment](#troubleshooting-access-environment)
  + [I can't access the Apache Airflow UI](#t-no-access-airflow-ui)

## Updating requirements\.txt<a name="troubleshooting-reqs"></a>

The following topic describes the errors you may receive when updating your `requirements.txt`\.

### I specified a new version of my `requirements.txt` and it's taking more than 20 minutes to update my environment<a name="t-requirements"></a>

If it takes more than twenty minutes for your environment to install a new version of a `requirements.txt` file, the environment update failed and Amazon MWAA is rolling back to the last stable version of the container image\.

1. Check package versions\. We recommend always specifying either a specific version \(`==`\) or a maximum version \(`>=`\) for the Python dependencies in your `requirements.txt`\.

1. Check Apache Airflow logs\. If you enabled Apache Airflow logs, verify your log groups were created successfully on the [Logs groups page](https://console.aws.amazon.com/cloudwatch/home#logsV2:log-groups) on the CloudWatch console\. If you see blank logs, the most common reason is due to missing permissions in your execution role for CloudWatch or Amazon S3 where logs are written\. To learn more, see [Execution role](mwaa-create-role.md)\.

1. Check Apache Airflow configuration options\. If you're using Secrets Manager, verify that the key\-value pairs you specified as an Apache Airflow configuration option were configured correctly\. To learn more, see [Configuring an Apache Airflow connection using a Secrets Manager secret key](connections-secrets-manager.md)\.

1. Check VPC network configuration\. To learn more, see [I tried creating an environment and it's stuck in the "Creating" state](#t-stuck-failure)\.

1. Check execution role permissions\. An execution role is an AWS Identity and Access Management \(IAM\) role with a permissions policy that grants Amazon MWAA permission to invoke the resources of other AWS services \(such as Amazon S3, CloudWatch, Amazon SQS, Amazon ECR\) on your behalf\. Your [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk) or [AWS owned CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk) also needs to be permitted access\. To learn more, see [Execution role](mwaa-create-role.md)\.

1. To run a troubleshooting script that checks the Amazon VPC network setup and configuration for your Amazon MWAA environment, see the [Verify Environment](https://github.com/awslabs/aws-support-tools/tree/master/MWAA) script in AWS Support Tools on GitHub\.

## Create bucket<a name="troubleshooting-create-bucket"></a>

The following topic describes the errors you may receive when creating an Amazon S3 bucket\.

### I can't select the option for S3 Block Public Access settings<a name="t-create-bucket"></a>

The [execution role](mwaa-create-role.md) for your Amazon MWAA environment needs permission to the `GetBucketPublicAccessBlock` action on the Amazon S3 bucket to verify the bucket blocked public access\. We recommend the following steps:

1. Follow the steps to [Attach a JSON policy to your execution role](mwaa-create-role.md)\. 

1. Attach the following JSON policy:

   ```
   {
      "Effect":"Allow",
      "Action":[
         "s3:GetObject*",
         "s3:GetBucket*",
         "s3:List*"
      ],
      "Resource":[
         "arn:aws:s3:::YOUR_S3_BUCKET_NAME",
         "arn:aws:s3:::YOUR_S3_BUCKET_NAME/*"
      ]
   }
   ```

   Substitute the sample placeholders in *YOUR\_S3\_BUCKET\_NAME* with your Amazon S3 bucket name, such as *my\-mwaa\-unique\-s3\-bucket\-name*\.

1. To run a troubleshooting script that checks the Amazon VPC network setup and configuration for your Amazon MWAA environment, see the [Verify Environment](https://github.com/awslabs/aws-support-tools/tree/master/MWAA) script in AWS Support Tools on GitHub\.

## Create environment<a name="troubleshooting-create-environment"></a>

The following topic describes the errors you may receive when creating an environment\.

### I tried creating an environment and it's stuck in the "Creating" state<a name="t-stuck-failure"></a>

We recommend the following steps:

1. Check VPC network with *public routing*\. If you're using an Amazon VPC *with Internet access*, verify the following:

   1. That the VPC security group allows all traffic in the self\-referencing rule, or *optionally* specifies the port range for HTTPS port range 443 and a TCP port range 5432\.

   1. That the two public subnets route to a NAT gateway \(or NAT instance\) with an Elastic IPv4 Address \(EIP\), and have a route table that directs internet\-bound traffic to an Internet gateway\.

   1. That the two private subnets have a route table to a NAT gateway \(or NAT instance\), and **do not** route to an Internet gateway\.

   1. To learn more, see [About networking on Amazon MWAA](networking-about.md)\.

1. Check VPC network with *private routing*\. If you're using an Amazon VPC *without Internet access*, verify the following:

   1. That the VPC security group allows all traffic in the self\-referencing rule, or *optionally* specifies the port range for HTTPS port range 443 and a TCP port range 5432\.

   1. That the two private subnets **do not** have a route table to a NAT gateway \(or NAT instance\), *nor* an Internet gateway\.

   1. That the two private subnets have a route table to the VPC endpoints for AWS services used by your environment, and VPC endpoints for Apache Airflow\.

   1. That the VPC endpoint policies allow "all access"\.

   1. That the local route table enables instances in your VPC to communicate with your own network\.

   1. To learn more, see [About networking on Amazon MWAA](networking-about.md)\.

1. To run a troubleshooting script that checks the Amazon VPC network setup and configuration for your Amazon MWAA environment, see the [Verify Environment](https://github.com/awslabs/aws-support-tools/tree/master/MWAA) script in AWS Support Tools on GitHub\.

### I tried to create an environment but it shows the status as "Create failed"<a name="t-create-environ-failed"></a>

We recommend the following steps:

1. Check VPC network configuration\. To learn more, see [I tried creating an environment and it's stuck in the "Creating" state](#t-stuck-failure)\.

1. Check user permissions\. Amazon MWAA performs a dry run against a user's credentials before creating an environment\. Your AWS account may not have permission in AWS Identity and Access Management \(IAM\) to create some of the resources for an environment\. For example, if you chose the **Private network** Apache Airflow access mode, your AWS account must have been granted access by your administrator to the [AmazonMWAAFullConsoleAccess](access-policies.md#console-full-access) access control policy for your environment, which allows your account to create VPC endpoints\.

1. Check execution role permissions\. An execution role is an AWS Identity and Access Management \(IAM\) role with a permissions policy that grants Amazon MWAA permission to invoke the resources of other AWS services \(such as Amazon S3, CloudWatch, Amazon SQS, Amazon ECR\) on your behalf\. Your [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk) or [AWS owned CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk) also needs to be permitted access\. To learn more, see [Execution role](mwaa-create-role.md)\.

1. Check Apache Airflow logs\. If you enabled Apache Airflow logs, verify your log groups were created successfully on the [Logs groups page](https://console.aws.amazon.com/cloudwatch/home#logsV2:log-groups) on the CloudWatch console\. If you see blank logs, the most common reason is due to missing permissions in your execution role for CloudWatch or Amazon S3 where logs are written\. To learn more, see [Execution role](mwaa-create-role.md)\.

1. To run a troubleshooting script that checks the Amazon VPC network setup and configuration for your Amazon MWAA environment, see the [Verify Environment](https://github.com/awslabs/aws-support-tools/tree/master/MWAA) script in AWS Support Tools on GitHub\.

### I tried to select a VPC and received a "Network Failure" error<a name="t-network-failure"></a>

We recommend the following steps:
+ If you see a "Network Failure" error when you try to select a VPC when creating your environment, turn off any in\-browser proxies that are running, and then try again\.

### I received a service, partition, or resource "must be passed" error<a name="t-service-partition"></a>

We recommend the following steps:
+ This may be because the S3 URI you entered for the S3 bucket for your environment includes a '/' at the end of the URI\. To resolve this issue, remove the '/' in the S3 bucket path\. The value should be an S3 URI in the following format:

  ```
  s3://your-bucket-name
  ```

### I tried to create an environment and it shows the status as "Available" but when I try to access the Airflow UI an "Empty Reply from Server" or "502 Bad Gateway" error is shown<a name="t-create-environ-empty-reply"></a>

We recommend the following steps:

1. Check VPC security group configuration\. To learn more, see [I tried creating an environment and it's stuck in the "Creating" state](#t-stuck-failure)\.

1. To run a troubleshooting script that checks the Amazon VPC network setup and configuration for your Amazon MWAA environment, see the [Verify Environment](https://github.com/awslabs/aws-support-tools/tree/master/MWAA) script in AWS Support Tools on GitHub\.

## Update environment<a name="troubleshooting-update-environment"></a>

The following topic describes the errors you may receive when updating an environment\.

### I tried changing the environment class but the update failed<a name="t-rollback-billing-failure"></a>

If you update your environment to a different environment class \(such as changing an `mw1.medium` to an `mw1.small`\), and the request to update your environment failed, the environment status goes into an `UPDATE_FAILED` state and the environment is rolled back to, and is billed according to, the previous stable version of an environment\.

We recommend the following steps:

1. Test your Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

1. To run a troubleshooting script that checks the Amazon VPC network setup and configuration for your Amazon MWAA environment, see the [Verify Environment](https://github.com/awslabs/aws-support-tools/tree/master/MWAA) script in AWS Support Tools on GitHub\.

## Access environment<a name="troubleshooting-access-environment"></a>

The following topic describes the errors you may receive when accessing an environment\.

### I can't access the Apache Airflow UI<a name="t-no-access-airflow-ui"></a>

We recommend the following steps:

1. Check user permissions\. You may not have been granted access to a permissions policy that allows you to view the Apache Airflow UI\. To learn more, see [Accessing an Amazon MWAA environment](access-policies.md)\.

1. Check network access\. This may be because you selected the **Private network** access mode\. If the URL of your Apache Airflow UI is in the following format `387fbcn-8dh4-9hfj-0dnd-834jhdfb-vpce.c10.us-west-2.airflow.amazonaws.com`, it means that you're using *private routing* for your Apache Airflow *Web server*\. You can either update the Apache Airflow access mode to the **Public network** access mode, or create a mechanism to access the VPC endpoint for your Apache Airflow *Web server*\. To learn more, see [Managing access to VPC endpoints on Amazon MWAA](vpc-vpe-access.md)\.