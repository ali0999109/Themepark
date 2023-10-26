# Cloud 9 Setup
- Go to the AWS Management Console, Select Services then select Cloud9  under Developer Tools. From the top-right of the Console, select an available region for this workshop. Once you have selected a region for Cloud9, use the same region for the entirety of this workshop.

- Select Create environment.
- Enter theme-park-development into Name and optionally provide a Description.
- For Instance type, choose t3.small. Keep the defaults in the other sections.
- Review the environment settings and select Create. It will take a few minutes for your Cloud9 environment to be provisioned and prepared.
- Verify that your user is logged in by running the command: aws sts get-caller-identity  C
- copy and paste the command into the Cloud9 terminal window.
- You'll see output indicating your account and user information:
  
  ![image](https://github.com/ali0999109/Themepark/assets/145396907/31b64a00-8706-495a-98bd-ad07cde38cb9)
  
- Check the current AWS Region to make sure you are running the workshop in a supported Region.
- Run these commands in the Cloud9 terminal window:
- Copy and paste: AWS_REGION=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone | sed 's/\(.*\)[a-z]/\1/')
SUPPORTED_REGIONS=("us-west-2" "us-east-1" "us-east-2" "eu-central-1" "eu-west-1" "ap-southeast-2" )
if [[ ! " ${SUPPORTED_REGIONS[@]} " =~ " ${AWS_REGION} " ]]; then
    /bin/echo -e "\e[1;31m'$AWS_REGION' is not a supported AWS Region, delete this Cloud9 instance and restart the workshop in a supported AWS Region.\e[0m"
else
    /bin/echo -e "\e[1;32m'$AWS_REGION' is a supported AWS Region\e[0m"
fi


- You should get this response
  
  ![image](https://github.com/ali0999109/Themepark/assets/145396907/d0ba7803-2fce-4156-a67f-b9d394f34992)

- Clone the repo which will download a local copy of the instructions and code you will use to build the backend portion of the workshop.
- cd ~/environment/
- git clone https://github.com/aws-samples/aws-serverless-workshop-innovator-island ./theme-park-backend
- Next, install JQ to provide formatting for JSON in the console:
- sudo yum install jq -y


# Deploy The Frontend Infrastructure
- Go to the AWS Management Console , click Services then select CodeCommit  under Developer Tools
- Select Create Repository.
- Set the Repository name to theme-park-frontend.
- Create
- Go back to your browser tab with Cloud9 running. If you need to re-launch Cloud9, from the AWS Management Console, select Services then select Cloud9  under Developer Tools.
- The AWS Cloud9 development environment comes with AWS managed temporary credentials that are associated with your IAM user. You use these credentials with the AWS CLI credential 
  helper. Enable the credential helper by running the following two commands in the terminal of your Cloud9 environment.
- git config --global credential.helper '!aws codecommit credential-helper $@'
- git config --global credential.UseHttpPath true
- Now run the following command to download the frontend code from GitHub into a separate local subdirectory from the backend instructions and code:
- mkdir ~/environment/theme-park-frontend
- cd ~/environment/theme-park-frontend
- wget https://innovator-island.s3-us-west-2.amazonaws.com/front-end/theme-park-frontend-202310.zip
- Unzip the code:
- unzip theme-park-frontend-202310.zip
- Set up the local Git repository and the commit:
- cd ~/environment/theme-park-frontend/
- git init -b main
- git add .
- git commit -am "First commit"
- Push the downloaded code to populate your recently created CodeCommit repository:
- AWS_REGION=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone | sed 's/\(.*\)[a-z]/\1/')
- git push --set-upstream https://git-codecommit.$AWS_REGION.amazonaws.com/v1/repos/theme-park-frontend main


# Deploy The Site With AWS Amplify
- Launch Amplify Console .
- Scroll down to the Get started section, then choose Get Started in the Amplify Hosting panel.
- Under Host your web app, select AWS CodeCommit and select Continue.
- Select theme-park-frontend under Recently updated repositories, then choose main under Branch. Select Next.
- On the Configure Build settings page, leave the defaults, scroll down and select Next.
- On the Review page, verify the settings and select Save and deploy.
- The deployment process will take a few minutes to complete. Once the build has a completed the Deploy stage, select the Amplify provided link for your app

  ![image](https://github.com/ali0999109/Themepark/assets/145396907/2db7ac2a-0ac3-42d5-83ab-8711445ccf86)


# The Serverless Backend
- A DynamoDB table which you will populate with information about all the rides and attractions throughout the park
- A Lambda function which performs a table scan on the DynamoDB to return all the items.
- An API Gateway API creates a public http endpoint for the front-end application to query. This invokes the Lambda function to return a list of rides and attractions.


# Step By Step Instructions 
- Go back to your browser tab with Cloud9 running. If you need to re-launch Cloud9, from the AWS Management Console, select Services then select Cloud9  under Developer Tools. Make 
 sure your region is correct.
- Create a deployment bucket in S3 with a unique name. SAM will upload its code to the bucket to deploy your application services. You will also store this bucket name as an 
  environment variable s3_deploy_bucket which will make it easier to type future deployment commands. In the terminal, run the following commands which pulls your accountID from the 
  Cloud9 Instance metadata and then creates and displays a unique S3 bucket name:

- accountId=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | jq -r .accountId)

  s3_deploy_bucket="theme-park-sam-deploys-${accountId}"

  echo $s3_deploy_bucket


- In the terminal, run the following commands to create the bucket: aws s3 mb s3://$s3_deploy_bucket
- Change directory: cd ~/environment/theme-park-backend/1-app-deploy/ride-controller/
- Use SAM CLI to deploy the first part of the infrastructure by running the following commands:
- sam package --output-template-file packaged.yaml --s3-bucket $s3_deploy_bucket
- sam deploy --template-file packaged.yaml --stack-name theme-park-ride-times --capabilities CAPABILITY_IAM
- This will take a few minutes to deploy. You can see the deployment progress in the console. Wait until you see the Successfully created/updated stack - theme-park-ride-times 
  confirmation message in the console before continuing.
- Now, change directory: cd ~/environment/theme-park-backend/1-app-deploy/sam-app/
- Use SAM CLI to deploy the second part of the infrastructure by running the following commands:
- sam build
- sam package --output-template-file packaged.yaml --s3-bucket $s3_deploy_bucket
- sam deploy --template-file packaged.yaml --stack-name theme-park-backend --capabilities CAPABILITY_IAM
- This will take a few minutes to deploy. You can see the deployment progress in the console. Wait until you see the Successfully created/updated stack - theme-park-backend 
  confirmation message in the console before continuing.
- Configure environment variables.
- Set a number of environment variables to represent the custom names of resources deployed in your account. These commands use the AWS CLI to retrieve the CloudFormation resource 
  names and then construct the environment variables using Linux string manipulation commands grep and cut. This makes it easier to type deployment commands in later modules. In the 
  terminal, run:
- AWS_REGION=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone | sed 's/\(.*\)[a-z]/\1/')
 FINAL_BUCKET=$(aws cloudformation describe-stack-resource --stack-name theme-park-backend --logical-resource-id FinalBucket --query "StackResourceDetail.PhysicalResourceId" --output 
 text)
 PROCESSING_BUCKET=$(aws cloudformation describe-stack-resource --stack-name theme-park-backend --logical-resource-id ProcessingBucket --query "StackResourceDetail.PhysicalResourceId"  output text)
 UPLOAD_BUCKET=$(aws cloudformation describe-stack-resource --stack-name theme-park-backend --logical-resource-id UploadBucket --query "StackResourceDetail.PhysicalResourceId" --output 
 text)
DDB_TABLE=$(aws cloudformation describe-stack-resource --stack-name theme-park-backend --logical-resource-id DynamoDBTable --query "StackResourceDetail.PhysicalResourceId" --output text)
echo $FINAL_BUCKET
echo $PROCESSING_BUCKET
echo $UPLOAD_BUCKET
echo $DDB_TABLE


- The terminal now looks like this, echoing back all the set environment variables:
  ![image](https://github.com/ali0999109/Themepark/assets/145396907/e7420841-3b54-4358-a69e-2c0a181cadb3)


# Populate The Dynamo Db Table
- From the Cloud9 console, navigate to the local-app directory in 1-app-deploy: cd ~/environment/theme-park-backend/1-app-deploy/local-app/
- Install the NPM packages needed: npm install
- Run the import script: node ./importData.js $AWS_REGION $DDB_TABLE


# Test The Config
- Confirm that the data is now in the DynamoDB table by running the following command: aws dynamodb scan --table-name $DDB_TABLE
- Call the API Gateway endpoint URL which SAM has created. First, run the following command in the console to show the endpoint URL: aws cloudformation describe-stacks --stack-name 
  theme-park-backend --query "Stacks[0].Outputs[?OutputKey=='InitStateApi'].OutputValue" --output text
- Once you have the endpoint URL, select the URL link in the Cloud9 terminal and select Open:
  ![image](https://github.com/ali0999109/Themepark/assets/145396907/cf20fcfe-0c58-4482-9131-3e5c60b743b9)
- This opens another browser tab and returns all the raw ride and attraction data from the DynamoDB table via API Gateway and Lambda. You have now created a public API that your 
  frontend application can use to populate the map with points of interest.

# Update The Frontend
- In this section, you will add the API endpoint you have created to the frontend configuration. This allows the frontend application to get the list of rides and attractions via the 
 APIGateway URL which pulls the information from DynamoDB.

- In the Cloud9 terminal, in the left directory panel navigate to theme-park-frontend/src.
- Locate the config.js file and double-click to open in the editor.
- In the MODULE 1 section at the beginning of the file, update the initStateAPI attribute of the API by pasting the API Endpoint URL from the previous section between the two '.
- Save the file.
  ![image](https://github.com/ali0999109/Themepark/assets/145396907/ebe4f308-3f87-4785-8346-dad9ac8b5faa)
# Push To CodeCommit And Deploy Via Amplify Consol

- In the Cloud9 terminal, change to the front-end directory with the following command: cd ~/environment/theme-park-frontend/
- Commit to CodeCommit by executing the following commands:
- git commit -am "Module 1"
- git push
- After the commit is completed, go to the Amplify Console . Make sure you are in the correct region.
- In the All apps section, click theme-park-frontend. If you are going back to a previously open browser tab, you may need to refresh.
 You will see a new build has automatically started as a result of the new commit in the underlying code repo. This build will take a few minutes. Once complete

# Module Review
- Connected your backend application with the Flow & Traffic Controller's SNS topic.
- Created a Lambda function that was invoked by the SNS topic whenever a new message is published.
- Published the message to the IoT topic for the front end.
- Updated the front end with the configuration information so it can listen to new messages on the IoT topic.





























