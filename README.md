# Cloud 9 setup
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










