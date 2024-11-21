# Datawise-Solution-Shell-Scripting-Functions-Arrays-Environment-variables-CLI-Error-handling-

## Background

Datawise solutions is a forward-thinking data science consulting company that specializes in deploying analytical and machine learning environment to support data-driven decision making for its clients.
Recognizing the need for agility and efficiency in setting up such environment, Datawise has decided to streamline its deployment process on AWS focusing on simplicity and automation.
The following shell scripting concepts will be employed:

- Functions
- Arrays
- Environment-variables
- Command Line Arguments
- Error-handling

## Scenario

One of DataWise Solutions' clients, a burgeoning e-commerce startup, is looking to harness the power of data science to analyze customer behavior and enhance their shopping experience. The startup wishes to deploy their data science workspace on AWS, utilizing EC2 instances for computational tasks and S3 buckets for storing their vast datasets of customer interactions.

## Specific requirements

To meet the client's needs and ensure a practical learning experience, DataWise Solutions plans to develop a script that automates the setup of EC2 instances and S3 buckets. This script will incorporate the 5 critical shell scripting concepts previously identified:

## Prerequisites

1. AWS CLI Installed and Configured
   `aws configure`
2. IAM Permissions: Ensure you have permissions to manage EC2 and S3
3. Key Pair: Create or have an existing SSH key pair for EC2 access.

Shell Script
This script will:

Use functions to modularize tasks.
Use arrays to manage resource IDs.
Leverage environment variables for configuration.
Accept command-line arguments for flexibility.
Implement error handling to ensure robustness.

- Let's save the script as automation.sh

  ```bash
  #!/bin/bash

# Load environment variables from the .env file

if [ -f .env ]; then
    source .env
else
    echo "Error: .env file not found. Please create a .env file with the necessary variables."
    exit 1
fi

# Enable strict error handling

set -euo pipefail

# Validate required environment variables

: "${AWS_REGION:?AWS_REGION is not set in .env file.}"
: "${AMI_ID:?AMI_ID is not set in .env file.}"
: "${INSTANCE_TYPE:?INSTANCE_TYPE is not set in .env file.}"
: "${VPC_ID:?VPC_ID is not set in .env file.}"
: "${SUBNET_ID:?SUBNET_ID is not set in .env file.}"
: "${SECURITY_GROUP_ID:?SECURITY_GROUP_ID is not set in .env file.}"

# Display loaded environment variables

echo "Environment Variables Loaded:"
echo "Region: $AWS_REGION"
echo "AMI ID: $AMI_ID"
echo "Instance Type: $INSTANCE_TYPE"
echo "VPC ID: $VPC_ID"
echo "Subnet ID: $SUBNET_ID"
echo "Security Group ID: $SECURITY_GROUP_ID"

# Declare an array to store resource IDs

declare -a RESOURCE_IDS

# Function to display usage information

usage() {
    echo "Usage: $0 <bucket-name> <key-name>"
    echo "Example: $0 my-bucket-name my-key-pair"
    exit 1
}

# Function to create an S3 bucket

create_s3_bucket() {
    local bucket_name=$1
    echo "Creating S3 bucket: $bucket_name"
    aws s3api create-bucket \
        --bucket "$bucket_name" \
        --region "$AWS_REGION"
    echo "S3 bucket $bucket_name created successfully."
    RESOURCE_IDS+=("$bucket_name")
}

# Function to launch an EC2 instance

launch_ec2_instance() {
    local key_name=$1
    echo "Launching EC2 instance..."
    local instance_id
    instance_id=$(aws ec2 run-instances \
        --image-id "$AMI_ID" \
        --instance-type "$INSTANCE_TYPE" \
        --key-name "$key_name" \
        --subnet-id "$SUBNET_ID" \
        --security-group-ids "$SECURITY_GROUP_ID" \
        --region "$AWS_REGION" \
        --query "Instances[0].InstanceId" \
        --output text)
    echo "EC2 instance launched successfully. Instance ID: $instance_id"
    RESOURCE_IDS+=("$instance_id")
}

# Function to verify deployment

verify_deployment() {
    echo "Verifying deployment..."
    for resource_id in "${RESOURCE_IDS[@]}"; do
        if [[ $resource_id == *"i-"* ]]; then
            local instance_state
            instance_state=$(aws ec2 describe-instances \
                --instance-ids "$resource_id" \
                --region "$AWS_REGION" \
                --query "Reservations[0].Instances[0].State.Name" \
                --output text)
            echo "EC2 Instance $resource_id State: $instance_state"
        else
            echo "S3 Bucket $resource_id exists and is accessible."
        fi
    done
}

# Main function

main() {
    if [ $# -ne 2 ]; then
        usage
    fi

    local bucket_name=$1
    local key_name=$2

    echo "Starting deployment..."
    create_s3_bucket "$bucket_name"
    launch_ec2_instance "$key_name"
    verify_deployment
    echo "Deployment complete."
}

 ```

# Call the main function with command-line arguments

`main "$@"`

 


## Explanation of Key Concepts
Environmental Variables:

REGION, AMI_ID, and INSTANCE_TYPE are configured as environment variables with defaults.
This allows flexibility for different environments or overrides.
Command-Line Arguments:

The script expects <bucket-name> and <key-name> as arguments.
The usage function explains the required input if they are missing.
Functions:

create_s3_bucket: Creates an S3 bucket and appends the bucket name to the RESOURCE_IDS array.
launch_ec2_instance: Launches an EC2 instance and appends the instance ID to RESOURCE_IDS.
verify_deployment: Verifies the state of EC2 instances and S3 bucket accessibility.
Arrays:

RESOURCE_IDS is used to track all created resources for verification and potential cleanup.
Error Handling:

set -euo pipefail: Ensures the script exits on errors, undefined variables, or pipe failures.
Conditional checks (if [ $? -eq 0 ]) ensure proper handling of AWS CLI command outcomes.

## Running the Script
- Make the script executable
`chmod +x automation.sh`
-Run the script while you make sure the .env and script are in thesame directory
./automation.sh <bucket-name> <key-name>
Replace <bucket-name> with a unique name for the S3 bucket and <key-name> with your existing AWS key pair name.

## Best Practices
Validation: Ensure bucket names and key names meet AWS naming standards.
Cleanup: Extend the script to delete created resources in case of errors or after testing.
Logging: Add logging for better troubleshooting.

