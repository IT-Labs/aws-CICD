
# Step 1 - Build the project on the cloud 

1.	Go to the AWS Management Console, click Services then select Cloud9 under Developer Tools.

2.	Click Create environment.

3.	Enter MyDevEnvironment into Name and optionally provide a Description.

4.	Click Next step.

5.	You may leave Environment settings at their defaults of launching a new t2.micro EC2 instance which will be paused after 30 minutes of inactivity.

6.	Click Next step.

7.	Review the environment settings and click Create environment. It will take several minutes for your environment to be provisioned and prepared.

8.	Use `aws sts get-caller-identity` to verify that your user is logged in.


## Create an AWS CodeCommit Repository

1.	Open the AWS CodeCommit console.

2.	On the Welcome page, choose Get Started Now. (If a Dashboard page appears instead, choose Create repository.)

3.	On the Create repository page, in the Repository name box, type WebAppRepo.

4.	In the Description box, type My demonstration repository.

5.	Choose Create repository to create an empty AWS CodeCommit repository named WebAppRepo.

## Clone the Repo

1.	From CodeCommit Console, you can get the https clone url link for your repo.

2.	Save the link, we will need it when cloning the repository 

3.	Go to Cloud9 IDE terminal prompt

4.	Run git clone to pull down a copy of the repository into the local repo:
`git clone [your repository https link]`

## Commit changes to Remote Repo

1.	Download the Sample Web App Archive from the sample app folder in the repository

2.	Go to cloud 9 click File, then upload Local files and upload the downloaded Zip file

3.	Unarchive and copy all the contents of the unarchived folder to your local repo folder

```unzip Web-App-Archive.zip```

```mv -v Web-App-Archive/* WebAppRepo/```

4. Change the directory to your local repo folder, commit and push the files 

```cd WebAppRepo```

```git add *  ```

```git commit -m "Initial Commit  ```

```git config credential.helper store  ```

```git push -u origin master  ```

## Prepare Build Service 

1.	First, let us create the necessary roles required to finish labs. Run the CloudFormation stack to create service roles. Ensure you are launching it in the same region as your AWS CodeCommit repo.

2.	Download the `01-aws-devops-workshop-roles.template` located in templates folder

3.  Go to CloudFormation service

4.	Click create stack with new resources 

5.	Select upload template file and upload the 01-aws-devops-workshop-roles.template that we downloaded 

6.	Click next 

7.	Enter DevopsWorkshop-roles as stack name 

8.	Click next 

9.	At the bottom under capabilities check I acknowledge that AWS CloudFormation might create IAM resources. 

10.	Then click create stack 

11. After the stack is created,go to outputs and save all the arns that are there

12.	Let us create CodeBuild project. To create the build project using AWS CLI, we need JSON-formatted input. Create a json file named 'create-project.json' under 'MyDevEnvironment'.

13.	Open this file located in files folder ,Copy the content to create-project.json. (Replace the placeholders marked with <<>> with values for BuildRole ARN, S3 Output Bucket and region from the previous step.).

14.	Save the file 

15.	Switch to the directory that contains the file you just saved, and run the `create-project` command:

```aws codebuild create-project --cli-input-json file://create-project.json```


## Build the code on cloud

1.Create a file namely, buildspec.yml under WebAppRepo folder. Copy the content from buildspec.yml file located under files directory.

2.Commit & push the build specification file to repository

```git add buildspec.yml ```

```git commit -m "adding buildspec.yml" ```

```git push -u origin master ```

3. Run the `start-build` command

```aws codebuild start-build --project-name devops-webapp-project ```


# Automate deployment for testing

## Prepare environment for Testing

1.	Go to Cloudformation 

2.	Click create a stack with new resources 

3.  Download the 02-aws-devops-workshop-environment-setup.template located under templates directory

4.	Check upload a template file and upload the template

5.	Click next

6.	Enter `DevopsWorkshop-Env` as stack name and click next

7.	Leave as default everything and click next 

9.	Check and acknowledge that AWS CloudFormation might create IAM resources.

10.	Click create stack 

## Create CodeDeploy Application and Deployment group

1.	Create an application for CodeDeploy.

```aws deploy create-application --application-name DevOps-WebApp ``` 

2. Create a deployment group and associate it with the specified application 

```
aws deploy create-deployment-group --application-name DevOps-WebApp \
--deployment-config-name CodeDeployDefault.OneAtATime \
--deployment-group-name DevOps-WebApp-BetaGroup \
--ec2-tag-filters Key=Name,Value=DevWebApp01,Type=KEY_AND_VALUE \
--service-role-arn <<REPLACE-WITH-YOUR-CODEDEPLOY-ROLE-ARN>>
 ```
 
 ## Prepare application for deployment 
 
1.	Create an appspec.yml file under WebAppRepo 

2. Copy the content from the coresponding file under files directory

3.	Modify buildspec.yml file and under artifacts files add the scripts and the appspec.yml file
 
4.	Save the changes to buildspec.yml file 

5.	Commit & push the build specification file to repository

```git add buildspec.yml ```

```git add appspec.yml ```

```git commit -m "changes to build and app spec ```

```git push -u origin master ```


## Deploy an application revision 

1.	Run the `start-build` command
```aws codebuild start-build --project-name devops-webapp-project ```

2.  Get the eTag for the object WebAppOutputArtifact.zip uploaded to S3 bucket.  

3. Create a deployment 

```
aws deploy create-deployment --application-name DevOps-WebApp \
--deployment-group-name DevOps-WebApp-BetaGroup \
--description "My very first deployment" \
--s3-location bucket=<<REPLACE-YOUR-S3-OUTPUT-BUCKET-NAME>>,key=WebAppOutputArtifact.zip,bundleType=zip,eTag=<<REPLACE-YOUR-ETAG-VALUE>>
 ``` 
 
 # Step 3 - Setup CI/CD using AWS CodePipeline
 
 ## Create a Pipeline 
 
1.	Open the AWS CodePipeline console.

2.	On the CodePipeline Home page, choose Create pipeline.

3.	On the Step 1: Choose pipeline settings page, in the Pipeline name box, type the name for your pipeline like WebAppPipeline

4.	For Service role, Select Existing service role and choose the Role name from drop down starting with DevopsWorkshop

5.	For Artifact store, Select Custom location and choose the Bucket from drop down starting with cicd-workshop, and then choose Next step.

6.	On the Step 2: Source page, in the Source provider drop-down list, choose the type of repository where your source code is stored and specify its required options:

•	AWS CodeCommit: In Repository name, choose the name of the AWS CodeCommit repository you created in Lab 1 to use as the source location for your pipeline. In Branch name, from the drop-down list, choose the master branch.

•	In Change Detection Mode leave the default selection of Amazon CloudWatch Events selection. Choose Next step.

7.	On the Step 3: Build page, do the following

•	Choose AWS CodeBuild, and then Select an existing build project.

•	Then choose Next step.

8.	On the Step 4: Deploy page, do the following, and then choose Next step:

•	Choose the following default providers from the Deployment provider drop-down list:

•	AWS CodeDeploy: Type or choose the name of an existing AWS CodeDeploy application in Application name and the name of a deployment group for that application in Deployment group we created and then choose Next step.

9.	On the Step 5: Review page, review your pipeline configuration, and then choose Create pipeline to create the pipeline.

 
 
 




