# AWS Serverless DynamoDB Backup to S3

A hands-on AWS project where I built a simple serverless workflow to fetch user data from a public API, store it in DynamoDB, and automatically back it up to Amazon S3.

This project was completed as part of my **BongoDev DevOps Bootcamp** while practicing AWS Lambda, Python, Boto3, IAM, and EventBridge Scheduler.

## Project Overview

The goal was to automate a basic data backup process without running or managing an EC2 instance.

The project uses two AWS Lambda functions:

- **Import Lambda:** Fetches sample user data from the JSONPlaceholder API and saves it to DynamoDB.
- **Backup Lambda:** Reads the records from DynamoDB and exports them to S3 as timestamped JSON files.

Amazon EventBridge Scheduler triggers the backup Lambda every two minutes.

## Architecture

```text
        JSONPlaceholder REST API
                  |
                  v
          AWS Lambda (Import)
                  |
                  v
           Amazon DynamoDB
         jsonplaceholder-users
                  |
                  v
          AWS Lambda (Backup) <---- EventBridge Scheduler
                  |                  (Every 2 minutes)
                  v
              Amazon S3
           JSON Backup Files
```

## Technologies Used

| Technology | Purpose |
|---|---|
| AWS Lambda | Run Python functions without managing servers |
| Amazon DynamoDB | Store user records |
| Amazon S3 | Store JSON backup files |
| EventBridge Scheduler | Run automatic backups every two minutes |
| AWS IAM | Manage permissions between AWS services |
| Amazon CloudWatch | Monitor Lambda executions and logs |
| Python | Write Lambda functions |
| Boto3 | Interact with AWS services using Python |

## What I Implemented

### 1. DynamoDB Setup

Created a DynamoDB table named `jsonplaceholder-users`.

Configuration:

- Partition key: `id`
- Data type: Number
- Region: `us-east-1`

### 2. S3 Bucket Setup

Created a private Amazon S3 bucket to store the JSON backup files.

Configuration:

- Region: `us-east-1`
- Block Public Access: Enabled
- Default encryption: SSE-S3
- Bucket versioning: Disabled for this lab

### 3. Import Users with AWS Lambda

Deployed a Python Lambda function named `import-jsonplaceholder-users`.

The function connects to the JSONPlaceholder API, retrieves 10 sample users, and writes the records to DynamoDB using Boto3.

I configured the `TABLE_NAME` environment variable and attached an IAM policy allowing `dynamodb:PutItem` on the specific table.

The Lambda test completed successfully with HTTP-style status code `200` and `saved_count` of `10`.

I also verified the records directly in DynamoDB.

### 4. Export DynamoDB Data to S3

Created a second Lambda function named `export-dynamodb-to-s3`.

This function scans the DynamoDB table, converts the records into JSON, and uploads them to S3.

I configured three environment variables:

```text
TABLE_NAME=jsonplaceholder-users
BUCKET_NAME=<your-s3-bucket>
BACKUP_PREFIX=backups
```

The Lambda execution role was given the necessary `dynamodb:Scan` and `s3:PutObject` permissions.

The first manual test exported all 10 users successfully.

### 5. Verify Manual Backups

I checked the S3 bucket to confirm that the exported JSON file was created.

I then ran the backup function multiple times to verify that each execution produced a separate timestamped file rather than overwriting the previous backup.

### 6. Automate the Backup with EventBridge Scheduler

After verifying the manual backups, I created an EventBridge schedule named `dynamodb-backup-every-2min`.

Configuration:

- Schedule type: Recurring
- Rate expression: `rate(2 minutes)`
- Target: `export-dynamodb-to-s3`
- Flexible time window: Off
- Payload: `{}`
- Scheduler execution role: Permission to invoke the backup Lambda

### 7. Verify Automatic Backups

I checked the S3 bucket again and confirmed that new backup files were created after enabling the schedule.

The number of backup objects increased from four to six, with new exports approximately two minutes apart.

### 8. Disable the Schedule

After verifying the automated backup workflow, I disabled the EventBridge schedule to prevent unnecessary executions and AWS charges.

The schedule configuration was kept so it could be enabled again if needed.

## Project Structure

```text
aws-homework/
├── README.md
├── lambda/
│   ├── import_users.py
│   └── backup_to_s3.py
├── iam-policies/
│   ├── lambda-import-putitem-policy.json
│   └── lambda-backup-export-policy.json
└── screenshots/
    ├── 01-dynamodb-table.png
    ├── 02-s3-bucket.png
    ├── 03-lambda-import.png
    ├── 04-dynamodb-users.png
    ├── 05-lambda-backup.png
    ├── 06-s3-backups.png
    ├── 07-s3-multiple-backups.png
    ├── 08-eventbridge-schedule.png
    ├── 09-automatic-s3-backups.png
    └── 10-eventbridge-disabled.png
```

## IAM and Security

I used separate execution roles for the import and backup functions.

The import function was permitted to write records to the DynamoDB table, while the backup function was permitted to scan the table and write JSON objects under the S3 `backups/` prefix.

The EventBridge Scheduler used its own execution role to invoke the backup function.

S3 public access was blocked. The IAM policy examples in this repository use placeholders for account-specific values.

## What I Learned

This lab helped me understand how multiple AWS services work together in a serverless application.

My main takeaways were:

- How Lambda execution roles control access to AWS resources.
- How environment variables keep configuration separate from application code.
- How Boto3 connects Python code with DynamoDB and S3.
- How EventBridge Scheduler can automate recurring workloads.
- Why verifying the actual data in DynamoDB and S3 matters, rather than relying only on a successful Lambda response.
- Why AWS resources and recurring schedules should be reviewed after finishing a lab.

## Limitations and Future Improvements

This was an educational project, not a production backup system.

The Lambda creates JSON exports of DynamoDB records. It does not provide a transactionally consistent snapshot or DynamoDB point-in-time recovery.

Possible improvements include adding backup retention policies, failure alerts, automated deployment with Terraform, GitHub Actions, and testing the restore process.

## Acknowledgment

This project was completed as part of the **BongoDev DevOps Bootcamp**. The instructor provided the starter Lambda code and IAM policy templates.

I configured the AWS resources, deployed the functions, tested the workflow, verified the backup files, and documented the results as part of my hands-on practice.
