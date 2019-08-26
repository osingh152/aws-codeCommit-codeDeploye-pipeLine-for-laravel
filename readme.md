Automate deployment for laravel application from CodeCommit to EC2 instance on AWS

Setup CI, CD on AWS with CodeCommit,CodeDeploye,Pipeline

Before setup CI/CD must have to install aws cli and aws Codedeploy agent

How to Install the AWS CLI on Linux
	Find out the documentation of install AWS cli https://docs.aws.amazon.com/cli/latest/userguide/install-linux.html
How to install AWS code deploye agent	
	1. sudo apt-get update
	2. sudo apt-get install ruby
	3. sudo apt-get install wget
	4. cd /home/ec2-user (Most of cases it is ubuntu)
	5. wget https://bucket-name.s3.region-identifier.amazonaws.com/latest/install
		bucket-name is the name of the Amazon S3 bucket that contains the CodeDeploy Resource Kit files for your region. 
		region-identifier is the identifier for your region. For example, for the US East (Ohio) Region, replace bucket-name with aws-codedeploy-us-east-2 and replace region-identifier with us-east-2
		Resource Kit Bucket Names by Regionhttps://docs.aws.amazon.com/codedeploy/latest/userguide/resource-kit.html#resource-kit-bucket-names 

Above all is the software dependency to run CI/CD in aws instance. Please install before setting up blow process
Now begin the process of setup CI/CD in AWS - 		
1. Create CodeCommit repository
 Make sure to create CodeCommit repository in specific region where ec2 instance running
 1. Create repository
 2. Clone repository in your local machine using command: git clone ssh://your-repository-url
 3. Create files in repository
	1. appspec.yml
		Specify youe setting and commands required to run the application
		Example: 
			version: 0.0
			os: linux
			files:
			  - source: /
				destination: /var/www/html/houseoftracks_api/
				overwrite: true
			hooks:
			   AfterInstall:
				- location: scripts/install_composer_dependencies
				  timeout: 600
				  runas: root    
 
	File Information:
	
		- source: Source to define what to be deploye on server
		 destination: Where files will be deployed
		 AWS codeDeploye follow the events to deploye the application called hooks like : BeforeInstall, AfterInstall, ApplicationStart, ApplicationStop
		  1. Write your script according to required hook. In my case I have to run commands after deployment
		  AfterInstall:
				- location: scripts/install_composer_dependencies -  This the file in which i have write the commands it supports shell script to write the code
				Samle code for laravel deployment(scripts/install_composer_dependencies): 
					#!/bin/bash
					cd /var/www/html/houseoftracks_api
					sudo composer install
					sudo php artisan key:generate
					sudo chmod -R 777 storage
					sudo php artisan migrate
					sudo composer dump-autoload
					sudo php artisan db:seed 
		
2. Create AmazonEC2RoleforAWSCodeDeploy for EC2 instance
   To create an instance role
	1. Open the IAM console at https://console.aws.amazon.com/iam/).
	2. From the console dashboard, choose Roles.
 	3. Choose Create role.
	4. Under Choose the service that will use this role, select EC2, and then choose Next: Permissions.
	5. Search for and select the policy named AmazonEC2RoleforAWSCodeDeploy, and then choose Next: Tags.
	6. Choose Next: Review. Enter a name for the role (for example, EC2InstanceRole), and then choose Create role.
3. Attach created AmazonEC2RoleforAWSCodeDeploy for already runnung insrance 
	Please follow the aws guide to attach role in exsting instance
		https://aws.amazon.com/blogs/security/easily-replace-or-attach-an-iam-role-to-an-existing-ec2-instance-by-using-the-ec2-console/	
4. Create an tag in running instance for identify the instance, We will use create tag in codedeplye application.
5. Create an Application in CodeDeploy
	1. To create code deploye service role:
		1. Open the IAM console at https://console.aws.amazon.com/iam/).
		2. From the console dashboard, choose Roles.
		3. Choose Create role.
		4. Under Choose the service that will use this role, choose CodeDeploy. Under Select your use case , choose CodeDeploy.
		5. Choose Next: Permissions, Next: Tags, and Next: Review.
		6. Enter a name for the role (for example, CodeDeployRole), and then choose Create role.
	2. To create an application in CodeDeploy	
		1. Open the CodeDeploy console at https://console.aws.amazon.com/codedeploy.
		2. If the Applications page does not appear, on the AWS CodeDeploy menu, choose Applications.
		3. Choose Create application.
		4. Leave Custom application selected. In Application name, enter YourApplicationName.
		5. In Compute Platform, choose EC2/On-premises.
		6. Choose Create application.
	3. To create a deployment group in CodeDeploy
		1. On the page that displays your application, choose Create deployment group.
		2. In Deployment group name, enter YourDeploymentGroupName.
		3. In Service Role, choose the service role you created earlier (for example, CodeDeployRole).
		4. Under Deployment type, choose In-place.
		5. Under Environment configuration, choose Amazon EC2 Instances. In the Key field, enter the tag key you used to tag the instance (for example, MyCodePipelineDemo).
		6. Under Deployment configuration, choose CodeDeployDefault.OneAtaTime.
		7. Under Load Balancer, clear Enable load balancing.
		8. Expand the Advanced section. Under Alarms, choose Ignore alarm configuration.
		9. Choose Create deployment group.
		
6. Create Your First Pipeline in CodePipeline
	You're now ready to create and run your first pipeline. In this step, you create a pipeline that runs automatically when code is pushed to your CodeCommit repository.		
	1. To create a CodePipeline pipeline
		1. Sign in to the AWS Management Console and open the CodePipeline console at http://console.aws.amazon.com/codesuite/codepipeline/home.
		2. Open the CodePipeline console at https://console.aws.amazon.com/codepipeline/.
		3. Choose Create pipeline.
		4. In Step 1: Choose pipeline settings, in Pipeline name, enter YourPipelineName.
		5. In Service role, choose New service role to allow CodePipeline to create a service role in IAM.
		6. In Artifact store, choose Default location, and then choose Next.
		7. In Step 2: Add source stage, in Source provider, choose AWS CodeCommit. In Repository name, choose the name of the CodeCommit repository you created in Step 1: Create a CodeCommit Repository. In Branch name, choose master, and then choose Next step.
		After you select the repository name and branch, a message displays the Amazon CloudWatch Events rule to be created for this pipeline.
		8. Under Change detection options, leave the defaults. This allows CodePipeline to use Amazon CloudWatch Events to detect changes in your source repository.
		9.Choose Next.
		10. In Step 3: Add build stage, choose Skip build stage, and then accept the warning message by choosing Skip again. Choose Next.
		Note: In this tutorial, you are deploying code that requires no build service, so you can skip this step. However, if your source code needs to be built before it is deployed to instances, you can configure CodeBuild in this step.
		11. In Step 4: Add deploy stage, in Deploy provider, choose AWS CodeDeploy. In Application name, choose yourCreatedApplication. In Deployment group, choose YourDeploymentGroup, and then choose Next step.
		12. Finally review the information, and then choose Create pipeline.

Sample code for deploye laravel application, Here is only aws deplyment file not laravel
	https://github.com/osingh152/aws-codeCommit-codeDeploye-pipeLine-for-laravel
	
Help Center:
	Install CodeDeploye: https://docs.aws.amazon.com/codedeploy/latest/userguide/codedeploy-agent-operations-install-ubuntu.html
	Troubleshooting-ec2: https://docs.aws.amazon.com/codedeploy/latest/userguide/troubleshooting-ec2-instances.html
	CodeDeploy resource: https://docs.aws.amazon.com/codedeploy/latest/userguide/resource-kit.html#resource-kit-bucket-names
	Error Codes for AWS CodeDeploy: https://docs.aws.amazon.com/codedeploy/latest/userguide/error-codes.html
