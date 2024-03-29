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

# On-ride photo processing
- This module shows how to use a Lambda function to perform a chroma key operation on user-generated images. It explains how the work is processed asynchronously and shows how to alert 
  the frontend when the image is ready.

# How It Works
- The front-end calls an API endpoint to get a presigned URL to upload the photo to S3. This enables the front-end application to upload directly to S3 without a webserver. This 
  results in a new JPG object in the S3 Upload bucket.
- When a new object is written to the Upload bucket, this invokes the first Lambda function in the sequence, which uses a process called Chromakey to remove the green screen background 
  from the image. The resulting image is written to the Processing bucket.
- When a new object is written to the Processing bucket, this invokes the next Lambda function which composites the image with a new background and theme park logo. This resulting 
  image is written into the Final bucket.
- Lastly, when a new object is written to the Final bucket, this invokes the final Lambda function which sends a notification to IoT core that the file is ready. This notifies the 
  front-end application via the IoT topic notifications.

- The serverless backend
  ![image](https://github.com/ali0999109/Themepark/assets/145396907/570f5e28-bd9e-4717-aef2-1572a8d7030a)

- The image is uploaded by the front-end into the Upload Bucket.
- A chromakey process removes the background and saves the object into the Processing Bucket.
- A compositing process creates a final image that is saved into the Final Bucket.
- A message is sent to IoT Core to notify the front-end that the file is now ready.

# Set up environment variablesHeader anchor link
- Run the following commands in the Cloud9 terminal to set environment variables used in this workshop:
- AWS_REGION=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone | sed 's/\(.*\)[a-z]/\1/')
FINAL_BUCKET=$(aws cloudformation describe-stack-resource --stack-name theme-park-backend --logical-resource-id FinalBucket --query "StackResourceDetail.PhysicalResourceId" --output text)
UPLOAD_BUCKET=$(aws cloudformation describe-stack-resource --stack-name theme-park-backend --logical-resource-id UploadBucket --query "StackResourceDetail.PhysicalResourceId" --output text)
accountId=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | jq -r .accountId)
s3_deploy_bucket="theme-park-sam-deploys-${accountId}"




  # Creating the OpenCV Lambda 
- Go back to your browser tab with Cloud9 running. If you need to re-launch Cloud9, from the AWS Management Console, select Services then select Cloud9  under Developer Tools. Make 
  sure your region is correct.
- In the terminal enter the following command to download the code for the layer:
- mkdir ~/environment/lambda-layer
- cd ~/environment/lambda-layer
- wget https://innovator-island.s3-us-west-2.amazonaws.com/opencv-python-311.zip
- Upload the zipped code package to your S3 deployment bucket: aws s3 cp opencv-python-311.zip s3://$s3_deploy_bucket
- Create the Lambda layer: aws lambda publish-layer-version --layer-name python-opencv-311 --description "OpenCV for Python 3.11" --content S3Bucket=$s3_deploy_bucket,S3Key=opencv- 
  python-311.zip --compatible-runtimes python3.11
- After a few seconds, the JSON response in the terminal confirms the LayerArn and Version of the new layer.
  ![image](https://github.com/ali0999109/Themepark/assets/145396907/99978c7b-af76-45c0-97a4-c9a93d92e301)


# Creating the Chromakey Lambda function
- Go to the Lambda console - from the AWS Management Console, select Services then select Lambda  under Compute. Make sure your region is correct.
- Select Create function:
- Enter theme-park-photos-chromakey for Function name.
- Ensure Python 3.11 is selected under Runtime.
- For Architecture, select x86_64.
- Open the Change default execution role section:
- Select the Use an existing role radio button.
- Click the Existing role drop-down, and enter ThemeParkLambdaRole until the filter matches a single available role beginning with theme-park-backend-ThemeParkLambdaRole*.
- Select this role.
- Choose Create function.
  ![image](https://github.com/ali0999109/Themepark/assets/145396907/738e2d87-7f98-473f-bb96-356ff5680b10)

- In the Function overview panel, select + Add Trigger:
- In the Trigger configuration dropdown, select S3.
- In the Bucket dropdown, select the bucket name beginning with theme-park-backend-uploadbucket.
- For Event Type select All object create events from the dropdown.
- Read the Recursive invocation warning, and select the checkbox to confirm that you have read and understood the warning.
- Choose Add.

  ![image](https://github.com/ali0999109/Themepark/assets/145396907/3cff2fd4-8723-4b4a-bf1c-0f65fbccb15f)


- Back on the Lambda function page, select the Code tab. Scroll down to the Layers card. Select Add a layer.
 ![image](https://github.com/ali0999109/Themepark/assets/145396907/41ae7272-1bf6-492e-a099-46ab1fa5600f)

- On the Add layer page:
- Select the Custom layers radio button.
- In the Custom layers dropdown, choose python-opencv-311.
- In the Version dropdown, choose 1.
- Select Add.
- Back on the Lambda function page, select the Code tab to view the Code source card
  ![image](https://github.com/ali0999109/Themepark/assets/145396907/b4524960-0bad-4d55-8b8c-730ed6c323d3)

- Copy the code from the file in Cloud9 by navigating to 3-photos/1-chromakey/app.py onto the clipboard and paste into the lambda_function.py tab in the Lambda function:
  
  ![image](https://github.com/ali0999109/Themepark/assets/145396907/4f6c3a68-ec88-493c-98d3-c274d5808fc5)

- Select Deploy in the Code source card to save the changes.

# Adding environment variables

- This function uses three environment variables:

- OUTPUT_BUCKET_NAME: the name of the bucket where the output object is stored.
- HSV_LOWER: A tuple representing lower HSV value for the green screen chroma key matching process.
- HSV_UPPER: A tuple representing upper HSV value for the green screen chroma key matching process.
- In this section, you will retrieve and configure these Environment Variables for the function.

- Go back to your browser tab with Cloud9 running. If you need to re-launch Cloud9, from the AWS Management Console, select Services then select Cloud9  under Developer Tools. Make 
 sure your region is correct.

- In the terminal enter the following command to retrieve the value for OUTPUT_BUCKET_NAME:

- aws s3 ls | grep theme-park-backend-processingbucket

- Go back to the browser tab with the theme-park-photos-chromakey Lambda function open. Select the Configuration tab, then choose the Environment variables menu item on the left. 
  Choose Edit.

- Enter the three environment variables with the three values, as follows:

- OUTPUT_BUCKET_NAME: the value from step 2 above.
- HSV_LOWER: [36, 100, 100]
- HSV_UPPER: [75 ,255, 255]

  ![image](https://github.com/ali0999109/Themepark/assets/145396907/f88772b5-0d29-45f1-900d-e2912a04af2b)


- In the browser tab with the theme-park-photos-chromakey Lambda function open, select the Configuration tab. Choose the General configuration option in the menu on the left.
  
  ![image](https://github.com/ali0999109/Themepark/assets/145396907/43969613-e091-4b7d-bfa2-1271124e501a)

- Choose Edit. Change the Memory (MB) value to 3008 MB. Change the Timeout values to 0 min and 10 sec.

- The chroma key process uses memory-intensive libraries to complete the graphics processing. By allocating the more memory, in this case 3008 MB or 3 GB, this function will complete 
  processing more quickly.

- Select Save to update the function.



# Creating the photo compositing Lambda function
- Go to your browser tab with Cloud9 running. If you need to re-launch Cloud9, from the AWS Management Console, select Services then select Cloud9  under Developer Tools. Make sure 
  your region is correct.

- In the Cloud9 file explorer panel, navigate to and open theme-park-backend/3-photos/2-compositing/template.yaml to review its contents.

- SAM will read this file and convert this YAML into infrastructure. Some of the important sections for today include:

- Parameters: the function needs to know the name of the final bucket, so you can provide this as a parameter to the template.
 Globals: any settings here will apply across the entire template.
 Resources: defines the AWS resources to create.
 Within the Resources section:

- The template defines a single Lambda function called CompositeFunction.
 It specifies the runtime, memory size and where the code can be located (in the lambdaFunction directory).
 It defines an environment variable, which uses the FinalBucketName parameter as an input.
 It provides the IAM policy to enable access for the function to S3.


# Create the Lambda function using SAM
- Change directory: cd ~/environment/theme-park-backend/3-photos/2-compositing
- You can see the name of the final S3 bucket using the following command which has been already stored as an environment variable $FINAL_BUCKET. SAM will be configured to use this S3 
  bucket name to set the environment variable within the Lambda function.
- aws s3 ls | grep finalbucket
- In the terminal, execute the following SAM CLI commands which will build the SAM application, package the code with the same SAM S3 deployment bucket previously used and then deploy 
 the application specifying the S3 Final Bucket name for the Lambda function to use:
- sam build

- sam package --output-template-file packaged.yaml --s3-bucket $s3_deploy_bucket

- sam deploy --template-file packaged.yaml --stack-name theme-park-photos --capabilities CAPABILITY_IAM --parameter-overrides "FinalBucketName"=$FINAL_BUCKET

- This will take a few minutes to deploy - wait for the confirmation message in the console before continuing.

- This will take a few minutes to deploy - wait for the confirmation message in the console before continuing.

# Adding the S3 trigger

- Now you have created the Lambda function, you need to configure how it is invoked. This compositing function needs to execute when a new object is put into the processingbucket. In 
  this section, you will create this trigger.


- Go to the Lambda console - from the AWS Management Console, select Services then select Lambda  under Compute. Make sure your region is correct.

- Select the function with the name theme-park-photos-CompositeFunction-XXXXXXXXX.

- Select + Add Trigger:

- In the Trigger configuration dropdown, Select S3.
- In the Bucket dropdown, select the bucket name beginning with theme-park-backend-processingbucket.
- For Event Type select All object create events from the dropdown.
- Check the Recursive invocation acknowledgement, and select Add.

# Creating the post-processing Lambda function
- Go to the Lambda console - from the AWS Management Console, select Services then select Lambda  under Compute. Make sure your region is correct.

- Select Create function:

- Ensure Author from scratch is selected.
- Enter theme-park-photos-postprocess for Function name.
- Ensure Node.js 18.x is selected under Runtime.
- For Architecture, choose arm64.

- Open the Change default execution role section:
  Select the Use an existing role radio button.
  Click the Existing role drop-down, and enter ThemeParkLambdaRole until the filter matches a single available role beginning with theme-park-backend-ThemeParkLambdaRole*.
  Select this role.
  Select Create function.


  ![image](https://github.com/ali0999109/Themepark/assets/145396907/7d51cdbf-14c3-47da-bbe6-b50664d5d32c)



- Select + Add trigger:
- In the Trigger configuration dropdown, select S3.
- In the Bucket dropdown, select the bucket name beginning with theme-park-backend-finalbucket.
- For Event Type select All object create events from the dropdown.
- Check the Recursive invocation acknowledgement, and select Add.
- Back in the Lambda function page, select the Code tab to view the Code source card.
- The console creates a Lambda function with a single source file named index.js or index.mjs. Make sure yours is named index.js by renaming it. For more information, check out 
  Building Lambda functions with Node.js .]
  ![image](https://github.com/ali0999109/Themepark/assets/145396907/e473f996-ef9c-4cb9-ad68-0320ed512654)


- Go back to your browser tab with Cloud9 running. If you need to re-launch Cloud9, from the AWS Management Console, select Services then select Cloud9  under Developer Tools. Make sure your region is correct.

- Copy the code from 3-postprocess/app.js onto the clipboard and paste into the index.js tab in the Lambda function, overwriting the existing content:

- Select Deploy in the Function Code panel to save the changes and deploy the function.

# Adding Environment Variables

- Go back to your browser tab with Cloud9 running. If you need to re-launch Cloud9, from the AWS Management Console, select Services then select Cloud9  under Developer Tools. Make sure your region is correct.

- In the terminal enter the following command to retrieve the value for IOT_DATA_ENDPOINT: aws iot describe-endpoint --endpoint-type iot:Data-ATS

- Next, enter the following command to retrieve the value for DDB_TABLE_NAME: aws dynamodb list-tables | grep backend

- Next, enter the following command to retrieve the value for WEB_APP_DOMAIN: aws cloudformation describe-stacks --stack-name theme-park-backend --query "Stacks[0].Outputs[?OutputKey=='WebAppDomain'].OutputValue" --output text

- Go back to the browser tab with the theme-park-photos-postprocess Lambda function open. Select the Configuration tab, then select Environment variables from the menu. Choose Edit.
Enter the two environment variables names along with the values you retrieved in Cloud9 (without quotes):

![image](https://github.com/ali0999109/Themepark/assets/145396907/1f3f0ea1-d7c7-4669-b236-360dce62592a)


# update the frontend
- In this section, you will update the frontend with the API endpoint where it can request photo uploads. You will then commit the change to CodeCommit, which will cause the frontend to be republished.

- Go back to your browser tab with Cloud9 running. If you need to re-launch Cloud9, from the AWS Management Console, select Services then select Cloud9 under Developer Tools. Make sure your region is correct.

- In the terminal, run this command to show the uploads API from when the backend was deployed in module 1: aws cloudformation describe-stacks --stack-name theme-park-backend --query "Stacks[0].Outputs[?OutputKey=='UploadApi'].OutputValue" --output text

- Copy the output URL to the clipboard:

- In the Cloud9 terminal, in the left directory panel navigate to theme-park-frontend/src.

- Locate the config.js file and double-click to open in the editor.

- This file contains a JSON configuration for the frontend. The file is separated into modules that correspond with the modules in this workshop.

- In the MODULE 3 section, between the single quote marks '', update the photoUploadURL with the API Endpoint URL in the clipboard.

- Save the file.

  ![image](https://github.com/ali0999109/Themepark/assets/145396907/7dd84e0b-88e5-4c77-9860-b8ffaabc8f92)


# Push to CodeCommit and deploy via Amplify

- In the Cloud9 terminal, change to the front-end directory with the following command: cd ~/environment/theme-park-frontend/

- Commit to CodeCommit by executing the following commands:
- git commit -am "Module 3 - Photo compositing"
- git push

- After the commit is completed, go to the Amplify Console .

- In the All apps section, click theme-park-frontend. You will see a new build has automatically started as a result of the new commit in the underlying code repo. This build will take 
  a few minutes until the Deploy stage is complete

- Open the published application URL in a browser.

 ![image](https://github.com/ali0999109/Themepark/assets/145396907/7e550fb9-2b3b-4f98-b800-c425d6784b88)


  








 





  

























 




























