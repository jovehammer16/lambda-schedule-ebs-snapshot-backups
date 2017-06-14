# AWS auto Snapshot
This project contains all the steps and files to do the followings:
1. Create scheduled Snapshot on AWS
1. Delete outdated Snapshot on AWS

## Requirement
You are suggested to have an IAM User account with IAM Full Access and CloudWatch Access
You will need the permissions to create new role/policy, list role/policy, create CloudWatch Rules, etc..
So it would be great to have Full Access for IAM and CloudWatch

## Create New Role and Policy for ebs-backup-worker
You can create new role with either [Management Console](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html#roles-creatingrole-service-console) or the [AWS CLI](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html#roles-creatingrole-service-cli). But the policy are the same.

#### Using the Management Console
1. Go to "IAM" in Management Console on AWS Website
1. Go to "Roles"
1. Click "Create new role"
1. Select "AWS Lambda" in role type
1. Press "Next" to skip "Attach Policy" section, we will come back to set it later
1. Name to role as "ebs-backup-worker"
1. Click "Create Role"
1. On the menu on the left hand side, click "Policies".
1. Click "Create Policy"
1. Select "Create Your Own Policy"
1. Set "Policy Name" as "TakeSnapshots"
1. Copy *snapshot-policy.json* files to "Policy Document"
1. Click "Create Policy"
1. Go back to the "Roles" section, select "ebs-backup-worker", click "Attach Policy" 
1. Select "TakeSnapshots"

#### Using AWS CLI
1. Install AWS CLI and login as your IAM User 
1. Clone this repo to your local computer
1. cd into the directory
1. Run the following command on terminal to create a new role
``
aws iam create-role --role-name ebs-backup-worker \
    --assume-role-policy-document file://snapshot-trust.json
``
1. Run the following command on terminal to attach the policy to the newly created role
``
aws iam put-role-policy --role-name ebs-backup-worker \
    --policy-name TakeSnapshots \
    --policy-document file://snapshot-policy.json
``

## Create Lambda functions
We will create 2 Lambda functions on Management Console
1. Go to Lambda function on AWS Management Console Website
1. Click "Create a Lambda function"
1. Select "Blank Function"
1. Click "Next" on "Configure trggers" page
1. Name the function as "createSnapshot"
1. Select "Python 2.7" as "Runtime"
1. Copy the code from the file *schedule-ebs-snapshot-backups.py*
1. Edit line 32 in *schedule-ebs-snapshot-backups.py* if you want to change expiry date (default 14 days)
1. In "Lambda function handler and role" session, confirm "Handler" is "lambda_function.lambda_handler", "Role" = "Choose an existing role", select "ebs-backup-worker" for "Existing role"
1. click "Next" and then "Create function"
1. Go back to AWS Lambda 
1. Click "Create a Lambda function"
1. Select "Blank Function"
1. Click "Next" on "Configure trggers" page
1. Name the function as "deleteSnapshot"
1. Select "Python 2.7" as "Runtime"
1. Copy the code from the file *ebs-snapshot-janitor.py*
1. In "Lambda function handler and role" session, confirm "Handler" is "lambda_function.lambda_handler", "Role" = "Choose an existing role", select "ebs-backup-worker" for "Existing role"
1. click "Next" and then "Create function"

## Create CloudWatch Rules
We will need to create CloudWatch Rules to trigger our functions
1. Go to CloudWatch, select "Rules" on the left menu
1. Click "Create rule"
1. select "Schedule"
1. You may set the rate in "Fixed rate of X Days", in my exmaple, I set "Cron expression" as "0 0 ? * 1 *" which means "UTC 00:00 every sunday" for creating snapshot
1. Click "Add targets" on the right hand side
1. Select "Lambda function"
1. select "createSnapshot" for "Function"
1. Click "Configure Details"
1. Name the rule as "createSnapshotRule"
1. Click "Create Rule"
#### Next, we need to repeat again to create rule for "deleteSnapshot"
1. Once again, Click "Create rule" on the "Rules" session
1. select "Schedule"
1. Set "fixed rate of 1 day"
1. Click "Add targets" on the right hand side
1. Select "Lambda function"
1. select "deleteSnapshot" for "Function"
1. Click "Configure Details"
1. Name the rule as "deleteSnapshotRule"
1. Click "Create Rule"

## Select target EC2 Instances to Backup
The last thing we need to do is to indicate the target instances. <br />
This will be established by adding tag to the target instances on EC2 console.
1. Go to EC2 console
1. Select "Instances" on the left menu
1. Select the target instances
1. In the session located on the bottom of the screen, select "Tags"
1. Click "Add/Edit Tags"
1. Add a key "Backup" with value "True"
1. Save

We are done with all setup at this point, you can go back to Lambda and test running createSnapshot and check if everythings' working!











