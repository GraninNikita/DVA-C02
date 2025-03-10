### EC2

* EC2 User data script - only run once at the instance first start, runs with the root user
* Instance types
  * General Purpose
  * Compute Optimized - compute-intensive tasks that require high performance processors
  * Memory Optimized - process large data sets in memory
  * Storage Optimized - high, sequential read and write access to large data sets on local storage
* Security Groups - only allow rules

### EBS

* An EBS (Elastic Block Store) - network drive, there might be latency, can be detacthed and attached to another EC2
* Have capacity (GB size and IOPS)
* There is also Delete on Termination attribute - you can protect EBS from removal when EC2 is terminated

### Auto Scaling Group
* There is a cooldown period - 300 seconds by default, there will be no launch/terminate in ASG to stabilize metrics

### STS—Security Token Service

Limited and temporary access to AWS (up to 1 hour)

- AssumeRole — assume roles within your account
- AssumeRoleWithSAML — returns credentials for users logged with SAML
- GetSessionToken - MFA
- GetFederationToken — temporary creds for a federated user
- GetCallerIDentity - returns details about the IAM user or role in API call
- DecodeAuthorizationMessage — decodes error message when AWS API is denied.

### STS to assume role:

* Define an IAM role, 15 mins to 1 hour
* support cross-account access with STS with MFA - GetSessionToken from STS

### Advanced IAM

default - DENY
IAM policies & S3 Bucket Policies - IAM + S3 = total policy (Union)

### Dynamic Policies with IAM

e.g. Need to assign each user a /home/<user> folder in S3 bucket

* Option 1: Create IAM policies allowing for each user his/her folder, e.g. /home/sarah, /home/brian, etc.
  One Policy per user does not scale
* Option 2: Use Dynamic policy. `policy variable  home/${aws:username}/`

### Inline vs. Managed policies

* Managed - only AWS can change. Good for admins
* Custom managed — you can change them, reusable, can be applied to many principals
* Inline — Strict one-to-one relationship between policy and principal. **Policy deleted when IAM principal removed.**
    * Restricted in size, cannot define a lot of resources.

### Passing a role to a service

iam:PassRole permission to pass to another service
iam:GetRole to view the role being passed
**Roles can only be passed to what was trusted**

### Microsoft AD

* AWS Managed Microsoft AD - Users are located in AD and in AWS. **Supports MFA**
* AD Connector — like a proxy. Limitation up to 5000 users
* Simple AD — without integration with Windows. Hosted in AWS.

### AWS other services

#### SES: Simple Email Service. Send + receive emails.

#### OpenSearch — use with DB to search + analitic queries

* Managed cluster + serverless cluster
* Does not support SQL
* CloudWatch logs + OpenSearch

#### Athena

* Analyze data by SQL language
* load to S3 and use Athena from S3
* Supports CSV
* Price: $5 per TB
* Used with Quicksight for reports
* **Use cases:** analytics, reports, query logs
* **Exam** : analyze data in serverless -> Athena
* **Exam**: Performance improvement: use columnar data: Apache Parquet, ORC, Use Glue to convert data to ORC, Parquet
    * Compress data
    * Partition datasets in S3
    * Use larger files (> 128 mb)
* Federated query

#### Amazon Managed Streaming for Apache Kafka

* alternative to Kinesis
  Kinesis | Amazon Kafka
  message limit

#### Certificate Manager (ACM)

* Manage SSL/TLS Certificates
* Private certificates: X.509, cannot be deployed on the internet, used internally.

#### Amazon Macie

* Checks sensitive data via ML, alerts.

#### App Config

* Configure, validate, deploy dynamic configurations, rollback

#### Cloudwatch Evidently

* validate feature to provide it to a specified % of your users
* **Use cases:** Launches, feature flags. Expiriments, compare versions.
* Overrides - pre-define a feature for a specific user

## AWS Security

* In-flight encryption: TLS/SSL - TLS certificates used(HTTPS).
* Server-side encryption at rest: encrypted after recieving, decrypted before sending. Stored in an encrypted form.
* Client-side encryption - Envelope Encryption

### KMS - Key management service

AWS manages encryption - **default**

* Able to audit KMS key via CloudTrail
* KMS through API calls, store encrypted keys in env/code

#### KMS Keys Types

* Customer Master Key
* Symmetric(AES-256) — used by AWS services
* Asymmetric — (RSA, ECC key pairs): public + private

#### Types of KMS keys

* AWS owned(free): SSE-S3, SSE-SQS, SSE-DDB (default)
* AWS managed (free): aws/service-name, e.g., aws/rds, aws/ens
* Customer managed keys _created_ in KMS: $1/month
* Customer managed keys _imported_ in KMS: $1/month
* pay for API call to KMS
* Key rotation: every 1 year
* Customer managed KMS key: automatic, on demand rotation
  Saved **per region**. Migrate: create EBS snapshot, use ReEncrypt with KMS key B
* KMS Key policies:
    * default — everyone can access an account
    * Custom key policy — define who can access the Key for cross-account

#### EXAM Security

- if on-premises needs to have access - create a programmatic access user
- `GenerateDataKeyWithoutPlainText` - generates not immediately DEK
- CloudHSM - hardware security modules
- Throttling: use DEK caching or contact AWS to increase, or through API
- `GenerateDataKey` - API command to generate a data key for each file to encrypt them

# AWS Global Infrastructure

- AWS region: Geographical locations where AWS hosts its data centers
- Availability Zone: small groups of data centers that are logically and physically separated
  by a distance that falls within 100 kilometers (km) (60 miles) of each other.

# Security

- AssumeRole API method - it is possible to set a time limitt on the role session using DurationSeconds argument
  max: 12 hours

## KMS Encryption

- KMS keys (CMK) - only encrypt data up to 4KB
- plain text data key - is downloaded together with an encrypted data key. Plain text DK should not be stored.
- x-amz-server-side-encryption - when AWS managed key is used,
    - for custom managed keys (SSE-C), it is required to specify algorithm and custom key

# DynamoDB

- Global secondary index.
    - **Can be created at any time**.
    - It Can be considered as a different partition key + sort key
    - No limitations of the size
    - Only eventual consistency is supported.
    - **has its own provisioned throughput settings for read and write activity that are separate from those of the table.**
- Local secondary index.
    - Contains the same partition key but different sort key
    - Limitation up to 10 GB per partition
    - It can be only created on table creation
    - Maximum 5 LSI per table
    - **shares provisioned throughput settings for read and write activity with the table it is indexing**

- WCU and RCU
    - Read Capacity Units (RCU):
        - 4KB, 1 Strongly Consistent (SC) read
        - 4KB, 1/2 Eventually Consistent (EC) read
        - 4KB, 2 Transactional read
    - Write Capacity Units (WCU):
        - 1KB, 1 write
        - 1KB, 2 Transactional writes
- `ReturnConsumedCapacity`:
    - `INDEXES` - The response includes the aggregate ConsumedCapacity for the operation, together with ConsumedCapacity
      for each table and secondary index that was accessed.
    - `TOTAL` - The response includes only the aggregate ConsumedCapacity for the operation.
    - `NONE` - No ConsumedCapacity details are included in the response.
- Condition keys:
    - `dynamodb:Attributes` - Filters access by attribute (field or column) names of the table
    - `dynamodb:LeadingKeys` - Filters access by the partition key of the table
    - `dynamodb:ReturnValues` - Filters access by the ReturnValues parameter of request. Contains one of the
      following: "ALL_OLD", "UPDATED_OLD", "ALL_NEW", "UPDATED_NEW", or "NONE"
    - `dynamodb:Select` - Filters access by the Select parameter of a Query or Scan request
- DAX - solves **hot key** problem. In-memory cache for DynamoDB. Does not require any changes in application. 5 min TTL
  by default. SECURE.
    - vs. Elasticache: DAX used for individual objects cache, query + scan. Elasticache: store aggregation result, e.g.
      SUM, TOTAL, etc.
- StreamViewType:
    - `KEYS_ONLY` - Only the key attributes of the modified item are written to the stream.
    - `NEW_IMAGE` - The entire item, as it appears after it was modified, is written to the stream.
    - `OLD_IMAGE` - The entire item, as it appeared before it was modified, is written to the stream.
    - `NEW_AND_OLD_IMAGES` - Both the new and the old item images of the item are written to the stream.
- DynamoDB Streams is subject to a 24-hour lifetime

# Kinesis

## Kinesis Data streams

- By default, data is stored for 24 hours. It can be increased up to 365 days by **IncreaseStreamRetentionPeriod**
- Shards
    - Writes capacity: 1MB/sec, 1000 records/sec, per shard
    - Read capacity: 2MB/sec, 1000 records/sec, per shard. Shared fanout: all consumers share a shard 2mb/sec.
        - Enhanced fanout type: 2MB for each consumer.
- ![img.png](img/img.png)
- Data is stored as records, max 1MB.
- PutRecords, PutRecord tu put data into stream
- Capacity modes:
    - On-demand - 4MB/sec, 4000 records/sec. It Can scale up to 200MB/sec, 200.000 records/sec
    - Provisioned - you need to specify the number of shards, you can increase(split)/decrease(merge) shards.

# Lambda

- Lambda authorizers
    - token-based, e.g., JWT or OAuth token
    - parameter-based - receive the caller's identity via headers, query strings, etc.
- Max timeout is 15 minutes
- the **unreserved concurrency** pool at a minimum of 100 concurrent executions so that functions that do not have
  specific limits set can still process requests. So, in practice, if your total account limit is 1000, you are limited
  to
  allocating 900 to individual functions.
- `FunctionUrlAuthType` - you can control access to Lambda Function Url:
    - `AWS_IAM` - Lambda uses AWS Identity and Access Management (IAM) to authenticate and authorize requests based on
      the IAM principal's identity policy and the function's resource-based policy. **Choose this option if you want
      only
      authenticated users and roles to invoke your function via the function URL.**
    - `NONE` - Lambda doesn't perform any authentication before invoking your function. However, your function's
      resource-based policy is always in effect and must grant public access before your function URL can receive
      requests. **Choose this option to allow public, unauthenticated access to your function URL.**
- `Unable to import module` error - Install the missing modules locally to your application’s folder. Package the folder into a ZIP file and upload it to AWS Lambda.

# X-Ray

- To properly instrument your application hosted in an EC2 instance, you have to install the X-Ray daemon by using a
  user data script. To use the daemon on Amazon EC2, create a new instance profile role or add the managed policy to an
  existing one.
- **Annotations**: key-value pairs to add user-specific custom attributes to your trace data. **They are indexed** and *
  *can be used for queries** using filter expressions. You can use annotations to provide more context to your searches
  and
  create groups. By adding these custom attributes as annotations, you can use filter expressions to limit the returned
  results based on these attributes. You can use them to group traces within the console or when you call the
  GetTraceSummaries API.
- **Metadata**: key-value pairs that are **not indexed**. The values can be of any type and comprise objects and lists.
  You can use metadata to record additional data to store in your traces but **do not require it for searches**.
- **Segments**:Segments provide detailed information about the requests made in your traces, including host IP, start
  and end times, duration, and request methods such as POST and GET requests. Segments also allow you to identify any
  issue such as errors, faults and exceptions. In the following screenshot, you can see a typical segment detail for a
  trace
- **SubSegments**:more granular details of segments and details of any downstream calls the application makes to serve
  those requests. These can contain details such as calls to an AWS service, databases, or APIs. Some services, such as
  DynamoDB, don’t send their own segments. Here, subsegments can be used to generate inferred segments for such services

- Lambda environment variables for X-RAY:
    - _`X_AMZN_TRACE_ID`: Contains the tracing header, which includes the sampling decision, trace ID, and parent
      segment ID. If Lambda receives a tracing header when your function is invoked, that header will be used to
      populate the _X_AMZN_TRACE_ID_ environment variable. If a tracing header was not received, Lambda will generate one for you.
    - `AWS_XRAY_CONTEXT_MISSING`: The X-Ray SDK uses this variable to determine its behavior in the event that your
      function tries to record X-Ray data, but a tracing header is not available. Lambda sets this value to LOG_ERROR by
      default.
    - `AWS_XRAY_DAEMON_ADDRESS`: This environment variable exposes the X-Ray daemon’s address in the following format:
      IP_ADDRESS:PORT. You can use the X-Ray daemon’s address to send trace data to the X-Ray daemon directly without
      using the X-Ray SDK.
- To enable X-Ray for app deployed in Elastic Beanstalk, include xray-daemon.config configuration file in the .ebextensions
- `GetTraceSummaries` - Retrieves IDs and annotations for traces available
- `BatchGetTraces` - Retrieves a list of traces specified by ID
- If a load balancer or other intermediary forwards a request to your application, X-Ray takes the client IP from the `X-Forwarded-For` header in the request instead of
  from the source IP in the IP packet. The client IP that is recorded for a forwarded request can be forged, so it should not be trusted.
- `AWSXRayDaemonWriteAccess` role Elastic Beanstalk uses it for the X-Ray daemon to upload data to X-Ray
- listens for traffic on **UDP port 2000**
- The `X-Forwarded-Port` header is used to represent the port number used by the client for the request.

# Athena

- It Is a serverless query service which enables fast analysis of data stored in Amazon S3 using standard SQL.

# Cognito

- Cognito Sync — makes it possible to sync application-related user data across devices.
  You Can synchronize user profile data across mobile devices and the web without using your own backend

# AWS AppSync

- Let you create a flexible API to securely access, manipulate, and combine data from one or more data sources.
  AppSync is a managed service that uses GraphQL to make it easy for applications to get exactly the data they need.
  E.g. `app preferences, game stats to be synchronized across devices`

# API Gateway

- Integration types:
    - `AWS` - Lets an API expose AWS service actions. You must configure
      both the integration request and integration response and set up necessary data mappings from the method request
      to the integration request, and from the integration response to the method response.
    - `AWS_PROXY` - Lets an API method be integrated with the Lambda function invocation action with a flexible,
      versatile, and streamlined integration setup. This integration relies on direct interactions between the client
      and the integrated Lambda function. Also known as the **Lambda proxy integration**, you do not set the integration
      request or the integration response. API Gateway passes the incoming request from the client as the input to the backend Lambda function.
    - `HTTP` - Lets an API expose HTTP endpoints in the backend. Also known as the HTTP
      custom integration, you must configure both the integration request and integration response.
    - `HTTP_PROXY` - Allows a client to access the backend HTTP endpoints with a streamlined
      integration setup on single API method. You do not set the integration request or the integration response.
    - `MOCK` - Lets API Gateway return a response without sending the request further to the backend
- Multi-value headers:
    - Additional `multiValueQueryStringParameters` and `multiValueHeaders`
    - E.g.:
      `"multiValueQueryStringParameters": { "myKey": ["val1", "val2"] },`
    - You have multiple _cookie_ headers:
      `    
                 "cookie": "name1=value1",
                  "cookie": "name2=value2",
          `\
      As a result:
    ```json 
     "multiValueHeaders": {
     "cookie": ["name1=value1", "name2=value2"],
     ...
     },
  ```

# CloudFormation

- When you have a few-lines code to be deployed into Lambda in CloudFormation, use `ZipFile` parameter
- helper scripts:
    - `cfn-init` – Use to retrieve and interpret resource metadata, install packages, create files, and start services.
    - `cfn-signal` – Use to signal with a CreationPolicy or WaitCondition, so you can synchronize other resources in the
      stack when the prerequisite resource or application is ready.
    - `cfn-get-metadata` – Use to retrieve metadata for a resource or path to a specific key.
    - `cfn-hup` – Use to check for updates to metadata and execute custom hooks when changes are detected.
- drift detection - allows to detect the changes on resources comparing to the initial configuration of the stack

# SAM

* Types:
    * API — Creates a collection of Amazon API Gateway resources
    * Application — Embeds a serverless application from the AWS Serverless Application Repository or from an Amazon S3 bucket as a nested application.
    * Connector — Configures permissions between two resources.
    * Function — Creates an AWS Lambda function, an AWS Identity and Access Management (IAM) execution role, and event source mappings that trigger the function.
    * GraphQLApi - create and configure an AWS AppSync GraphQL API for your serverless application.
    * HttpApi - Creates an Amazon API Gateway HTTP API, which enables you to create RESTful APIs with lower latency and lower costs than REST API
    * LayerVersion — Creates a Lambda LayerVersion that contains library or runtime code needed by a Lambda Function
    * SimpleTable — Creates a DynamoDB table with a single attribute primary key. It is useful when data only needs to be accessed via a primary key.
    * StateMachine - Creates an AWS Step Functions state machine, which you can use to orchestrate AWS Lambda functions and other AWS resources to form complex and robust
      workflows.
* Local testing strategy: SAM CLI. Supports local invocation and testing
    * `sam package` - creates a deployment package and uploads it to S3
    * `sam deploy` - deploys template into CF stack
* Using DeploymentPreference to enable gradual Lambda deployments.
    *   ```
      Alarms: List
      Enabled: Boolean
      Hooks: Hooks
      PassthroughCondition: Boolean
      Role: String
      TriggerConfigurations: List
      Type: String
      ```
    * Alarms — A list of CloudWatch alarms that you want to be triggered by any errors raised by the deployment.
    * Hooks — Validation Lambda functions that are run before and after traffic shifting

# CDK

- How to test what will be created locally?: `cdk init app --language typescript` + `cdk synth --no-staging`

# BeanStalk

- Deployment strategies:
    - All-at-once - Performs in place deployment on all instances.
    - Rolling — Splits the instances into batches and deploys to one batch at a time.
    - Rolling with additional batch - Splits the deployments into batches but for the first batch creates new EC2
      instances instead of deploying on the existing EC2 instances.
    - Immutable — If you need to deploy with a new instance instead of using an existing instance. (Auto scalling
      groups)
    - Traffic splitting — Performs immutable deployment and then forwards percentage of traffic to the new instances for
      a pre-determined duration of time. If the instances stay healthy, then forward all traffic to new instances and shut
      down old instances.
- Config files:
    - `.ebextensions` folder at the root folder

# Troubleshooting and Optimization

## API Gateway + Lambda -> returns 504 error code.

* INTEGRATION_TIMEOUT of API Gateway — because there is a timeout maximum value for all integration types: **29 seconds**. P.s. AWS now supports increasing
  it: https://aws.amazon.com/about-aws/whats-new/2024/06/amazon-api-gateway-integration-timeout-limit-29-seconds/
* INTEGRATION_FAILURE — if integration does not work

## How to capture information about the IP traffic going to and from network interfaces in your VPC

* Use flow log in your VPC.

# Cloudwatch

* It does not track memory utilization by default. It is recommended to create a custom metric to do it.
* When you create an alarm, you specify three settings to enable CloudWatch to evaluate when to change the alarm state:

* **Period** is the length of time to use to evaluate the metric or expression to create each individual data point for an alarm. It is expressed in seconds.

* **Evaluation Periods** is the number of the most recent periods, or data points, to evaluate when determining alarm state.

* **Datapoints to Alarm** is the number of data points within the Evaluation Periods that must be breaching to cause the alarm to go to the ALARM state. The breaching
  data
  points don't have to be consecutive, but they must all be within the last number of data points equal to Evaluation Period.

# S3

* aws s3api list-objects
    * `--page-size` - pagination size
    * `--max-items` - how many items to output
* Encryption
  * SSE-S3 - must set `x-amz-server-side-encryption": "AES256`
  * SSE-KMS - `x-amz-server-side-encryption": "aws:kms`

# Step functions

* `waitForTaskToken` - Wait for Callback

# Tricky questions

- You got an encoded error message, how to decode it?
    - `aws sts decode-authorization-message --encoded-message
- AWS WAF - service to protect against common web exploits and bots that can affect availability, compromise security, or consume excessive resources.
- Amazon ElastiCache
    - for Memcached - **multithread support**
    - for Redis - usually **single-threaded**

# Deployment section

### Cloud computing deployment models

* On-premises - hosting hardware in companies own data centers
* Cloud - on-demand delivery of IT resources over the internet with primarily pay-as-you-go pricing
* Hybrid is a way to connect infrastructure and applications between cloud-based resources and existing resources that are not located in the cloud.
*

ECS stopping container and deregistering - When a container instance is terminated in the stopped state, the container instance is not automatically deregistered from the
cluster.