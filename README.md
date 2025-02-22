<p align="center">
  <img src="assets/diagram.png" 
</p>
  
## ☁️ 30 Days DevOps Challenge - Automating Sports Data Highlights  ☁️

This is part of Week 3 for the 30-day DevOps Challenge!

In this project I automated and deployed Sports highlights which will be stored in S3 and DynamoDB and runs on a automated schedule using ECS Fargate and EventBridge. It processes videos, and uses template JSON files with environment variables for easy configuration and production release. 


<h2>Environments and Technologies Used</h2>

  - Amazon Web Services (AWS)
  - RapidAPI
  - DynamoDB
  - Elastic Container Services
  - Amazon S3
  - Docker
  - Cloudwatch


  
<h2>Key Features</h2>  

✅ Automated Sports Highlights Extraction: Automatically processes and extracts highlights from sports videos, saving time and effort.

✅ Runs on a fully automated schedule using Amazon EventBridge, ensuring timely processing without manual intervention.

✅ Simplified deployment process with environment variables and reusable templates, making it easy to maintain and update.

<h2>Step by Step Instructions</h2>

***1. Repo and API configuration***

We will begin by setting up the environment and code that we will be utilizing. In this instance, we will use gitpod to create a new workspace and do the commands from there. We will be setting up an account with RapidAPI.

I created a .yml script for gitpod where it will automatically install AWS CLI and set the AWS credentials with the environment variables defined in Gitpod. This makes sure that our future projects are automated and we can start right away.

To achieve this, we will go to Gitpod's settings and set our credentials with the variables `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` and `AWS_DEFAULT_REGION` Respectively.

![image](/assets/image1.png)

Finally, we will make sure our dependencies are installed properly.

```
pip install boto3
pip install python-dotenv
pip install requests
```

***Option 2: Local AWS CLI Setup***

NOTE: Keep in mind this is for a Linux environment, check the AWS documentation to install it in your supported Os.

   ```
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
We then do `AWS configure` and enter our access and secret key along with the region. Output format set to JSON. With this command we will double check that our credentials are put in place for CLI:

```
aws sts get-caller-identity
```

And finally, we will also be installing `gettext` which is a command-line utility is used for environment variable substitution in shell scripts and text files.

```
sudo apt-get update 
sudo apt-get install gettext
```

***2.  Configuring and Creating .env file***

In this step we will be configurating and setting up the .env file that we will be using for this project. 

We will need to replace the following in the `.env` file:

- Your AWS Account ID with `aws sts get-caller-identity --query "Account" --output text`
- Your-RAPIDAPI-Key
- Your-AWS-Access-Key
- Your-AWS-Secret-Access-key
- S3_BUCKET_NAME=your-alias
- Your-MediaConvert-Endpoint which we can get with command `aws mediaconvert describe-endpoints`
- SUBNET_ID=subnet-
- SECURITY_GROUP_ID=sg-

NOTE: For the `SUBNET_ID`and `SECURITY_GROUP_ID`  we will use the `vpc_setup.sh` file on the src folder for the creation of the VPC with subnets and security groups.

Finally, we will load the .env file:

```
set -a
source .env
set +a
```

***3. Generate JSON files with Templates***

In this step, we will be loading the variables and inject it into our JSON file and create the files necessary for the task to execute. 

We will be begin by generating the json with the ECS Task Definition. 

`envsubst < taskdef.template.json > taskdef.json`

Then, we will set the S3/DynamoDB Policy

`envsubst < s3_dynamodb_policy.template.json > s3_dynamodb_policy.json`

Now, setting the ECS Target

`envsubst < ecsTarget.template.json > ecsTarget.json`

And finally, setting the ECS Events role policy:

`envsubst < ecsEventsRole-policy.template.json > ecseventsrole-policy.json`

NOTE: Make sure that you install envsubst

***4. Build and Push Docker Image***

In this final step, we will create our docker image and build it pointing to our source folder.

First, we will be creating an ECR repo where we will be hosting our docker container in the cloud:

```bash
aws ecr create-repository --repository-name sports-backup
```

We will then log in to ECR

```bash
aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin accountid.dkr.ecr.us-east-1.amazonaws.com
```

Next, we will build our docker image

```bash
docker build -t sports-backup .
```

We will now tag the image

```bash
docker tag sports-backup:latest ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/sports-backup:latest
```
Lastly, we will proceed with pushing the image

```bash
docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/sports-backup:latest
```

***5. Create ECS Task Definition and Role configuration***

In this step, we will be creating a task definition to be set with Elastic Container Services, as well as set up Cloudwatch logs and the roles necessary to execute our project.

First, we create the task definition with our generated json file:

```
aws ecs register-task-definition --cli-input-json file://taskdef.json --region us-east-1
```

Then, we create the Cloudwatch log group

```
aws logs create-log-group --log-group-name "${AWS_LOGS_GROUP}" --region us-east-1
```

Next, we will attach the S3/DynamoDB Policy to the ECS Task Execution Role

```
aws iam put-role-policy \
  --role-name ecsTaskExecutionRole \
  --policy-name S3DynamoDBAccessPolicy \
  --policy-document file://s3_dynamodb_policy.json
```

And finally, we will create and attach the ECS role

```
aws iam create-role --role-name ecsEventsRole --assume-role-policy-document file://ecsEventsRole-trust.json
aws iam put-role-policy --role-name ecsEventsRole --policy-name ecsEventsPolicy --policy-document file://ecseventsrole-policy.json
```


***6. Setting up EventBridge and Testing ECS Task - Final Result***

In this step, we will be creating a rule for setting up an event with Eventbridge as well as run the ECS task created.

We first begin by creating the EventBridge rule:

```
aws events put-rule --name SportsBackupScheduleRule --schedule-expression "rate(1 day)" --region ${AWS_REGION}
```

Then, we add the target

```
aws events put-targets --rule SportsBackupScheduleRule --targets file://ecsTarget.json --region ${AWS_REGION}
```

And finally, we test the ECS task. 

```
aws ecs run-task \
  --cluster sports-backup-cluster \
  --launch-type Fargate \
  --task-definition ${TASK_FAMILY} \
  --network-configuration "awsvpcConfiguration={subnets=[\"${SUBNET_ID}\"],securityGroups=[\"${SECURITY_GROUP_ID}\"],assignPublicIp=\"ENABLED\"}" \
  --region ${AWS_REGION}
```


<h2>Conclusion</h2>

With this tool we are able to fetch Sports highlights with RapidAPI keys, store them in S3 and DynamoDB, and run an automated task by deploying docker containers of the application with an ECS task and Cloudwatch rules set up with EventsBridge.

