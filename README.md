# aws-codedeploy-event-logs
AWS CodeDeploy event logs stored in S3 and query-able by Athena

## Architecture

![enter image description here](https://d50daux61fgb.cloudfront.net/aws-codedeploy-event-logs/solution-architecture.png)

Two EventBridge event rules for the default event bus track each type of CodeDeploy event. Each event rule targets a specific Kinesis Firehose delivery stream that will put log files into a certain S3 folder corresponding to a Glue table. A Lambda function is used by each delivery stream to transform the CloudWatch event payload into the log file format. After a log file is placed in the bucket, a Lambda function is run that creates a partition for the corresponding Glue table, is one does not exist.

## Event Types

### CodeDeploy Deployment State Change

The state of a deployment has changed

#### Log Format

 - **timestamp** *(string)* - The ISO 8601 format of the timestamp of the event
 - **eventVersion** *(string)* - The version of the event format
 - **applicationArn** *(string)* - The ARN of the application
 - **application** *(string)* - The name of the application
 - **deploymentGroupArn** *(string)* - The ARN of the deployment group
 - **deploymentGroup** *(string)* - The name of the deployment group
 - **region** *(string)* - The AWS region of the deployment
 - **state** *(string)* - The deployment state
 - **deploymentId** *(string)* - The ID of the deployment
 - **instanceGroupId** *(string)* - The ID of the instance group


### CodeDeploy Instance State Change

The state of an instance that belongs to a deployment group has changed

#### Log Format

 - **timestamp** *(string)* - The ISO 8601 format of the timestamp of the event
 - **eventVersion** *(string)* - The version of the event format
 - **instanceArn** *(string)* - The ARN of the instance
 - **instance** *(string)* - The ID of the instance
 - **applicationArn** *(string)* - The ARN of the application
 - **application** *(string)* - The name of the application
 - **deploymentGroupArn** *(string)* - The ARN of the deployment group
 - **deploymentGroup** *(string)* - The name of the deployment group
 - **region** *(string)* - The AWS region of the deployment
 - **state** *(string)* - The deployment state
 - **deploymentId** *(string)* - The ID of the deployment
 - **instanceGroupId** *(string)* - The ID of the instance group


## Log Partitions

All Glue tables are partitioned by the date and hour that the log arrives in S3. It is highly recommended that Athena queries on Glue database filter based on these paritions, as it will greatly reduce quety execution time and the amount of data processed by the query.


## Build and Deploy

To deploy the application, use the `cloudformation package` command from the AWS CLI. 
 
#### Example:

`aws cloudformation package --template templates/root.yaml --s3-bucket $S3_BUCKET --s3-prefix $S3_PREFIX --output-template template-export.yaml`