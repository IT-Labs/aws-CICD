# aws CI/CD workshop 

## An introductory Workshop on AWS CI/CD practises

In this workshop we will show how to effectively leverage various AWS services that can help us make the software release process fast,automated, and reliable.
Our goal will be to embrace some of the best practises around continuous integration & delivery using AWS Developer tools including, AWS CodeCommit
(a managed source control service), AWS CodeBuild (a fully managed build service), CodePipeline (a fully managed continuous delivery service), 
and CodeDeploy (an automated application deployment service).

See the diagram below for the architecture. 

![alt text](https://github.com/IT-Labs/aws-CICD/blob/main/img/CICD_DevOps_Demo_onlyDev.png)

## Prerequisites

- **Aws account**
- **IAM user:**: An IAM user with programatic and console access 
- **IAM Permissions:**: Ensure your user has sufficient privileges. Give the user administrator access or ensure it has the following permissions

	AWS Identity and Access Management
  
	Amazon Simple Storage Service
  
	AWS CodeCommit
  
	AWS CodeBuild
  
	AWS CloudFormation
  
	AWS CodeDeploy
  
	AWS CodePipeline
  
	AWS Cloud9
  
	Amazon EC2
  
	Amazon SNS
  
	Amazon CloudWatch Events



## Clean up the resources

1.	Visit CodePipeline console, select the created pipeline. Select the Edit and click Delete.

2.	Visit CodeDeploy console, select the created application. In the next page, click Delete Application.

3.	Visit CodeBuild console, select the created project. Select the Action and click Delete.

4.	Visit CodeCommit console, select the created repository. Go to setting and click Delete repository.

5.	Visit Cloudformation console, select the created stacks. Select the Action and click Delete Stack.

6.	Visit Cloud9 console, select the created Environment. Select the Action and click Delete.

7.	Visit s3 console, select the created buckets and click Delete.
