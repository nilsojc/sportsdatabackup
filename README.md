<p align="center">
  <img src="assets/diagram.png" 
</p>
  
## ☁️ 30 Days DevOps Challenge - Automating Sports Data Highlights  ☁️

This is part of the final project of the Multicloud, Devops and AI Challenge!

In this project I automated and deployed Sports highlights which will be stored in S3 and DynamoDB and runs on a automated schedule using ECS Fargate and EventBridge. It processes videos, and uses template JSON files with environemnt variables for easy configuration and production release. 




<h2>Environments and Technologies Used</h2>

  - Amazon Web Services (AWS)
  - RapidAPI
  - DynamoDB
  - ECS Fargate
  - Amazon S3


  
<h2>Key Features</h2>  

✅ Automated Sports Highlights Extraction: Automatically processes and extracts highlights from sports videos, saving time and effort.
✅ Runs on a fully automated schedule using Amazon EventBridge, ensuring timely processing without manual intervention.
✅ Simplified deployment process with environment variables and reusable templates, making it easy to maintain and update.

<h2>Step by Step Instructions</h2>

***1. Repo and API configuration***

We will begin by setting up the environment and code that we will be utilizing. In this instance, we will use gitpod to create a new workspace and do the commands from there. We will be setting up an account with RapidAPI for our Premier League Standings data.

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

And finally, we will also be installing `gettext` which is a command-line utility is used for environment variable substituition in shell scripts and text files.

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

***3.  ***



***4. Deploy Cloudmart backend and update Kubernetes - Final Result***

In this final step, we will create our `cloudmart-backend.yaml` file to execute the deployment:



<h2>Conclusion</h2>

The successful completion of these integrations and deployments established CloudMart as a production-ready platform with robust analytics capabilities and scalable infrastructure.
