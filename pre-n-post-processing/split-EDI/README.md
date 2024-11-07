# EDI File Splitter

This solution provides an AWS Lambda function that automatically splits large EDI (Electronic Data Interchange) files into individual transactions. When an EDI file is uploaded to an S3 bucket, the Lambda function processes it and creates separate files for each transaction while maintaining the proper EDI structure (ISA/GS/ST segments).

## Features

- Automatically splits EDI files into individual transactions
- Maintains EDI envelope structure in each split file
- Handles different types of segment terminators and suffixes
- Supports files up to 5GB through Lambda's ephemeral storage
- Processes files asynchronously via S3 events

## Prerequisites

- AWS Account
- AWS CLI installed and configured
- S3 bucket for uploading EDI files

## Deployment Steps

### Option 1: AWS Console Deployment

1. Sign in to the AWS Management Console

2. Navigate to CloudFormation:
   - Go to Services > CloudFormation
   - Click "Create stack" > "With new resources (standard)"

3. Upload the template:
   - Select "Upload a template file"
   - Click "Choose file" and select the `edi-split.yaml` file
   - Click "Next"

4. Configure stack:
   - Enter a Stack name (e.g., "edi-splitter")
   - Under Parameters:
     - S3BucketName: Enter your bucket name
     - S3OutputPrefix: Enter the prefix for split files (e.g., "split/")
   - Click "Next"

5. Configure stack options:
   - Leave default settings
   - Click "Next"

6. Review:
   - Check the acknowledgment box for IAM resource creation
   - Click "Create stack"

7. Wait for stack creation to complete (status: CREATE_COMPLETE)

### Option 2: AWS CLI Deployment

1. Clone or download this repository

2. Deploy the CloudFormation stack using AWS CLI:
```bash
aws cloudformation create-stack \
  --stack-name edi-splitter \
  --template-body file://edi-split.yaml \
  --capabilities CAPABILITY_IAM \
  --parameters \
    ParameterKey=S3BucketName,ParameterValue=your-bucket-name \
    ParameterKey=S3OutputPrefix,ParameterValue=split/
```

Replace `your-bucket-name` with your S3 bucket name and adjust the output prefix as needed.

## S3 Bucket Configuration

1. Create an S3 bucket (if you haven't already):
```bash
aws s3 mb s3://your-bucket-name
```

2. Add a bucket policy to allow Lambda to access the bucket. Create a file named `bucket-policy.json`:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowLambdaAccess",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::YOUR_ACCOUNT_ID:role/STACK_NAME-LambdaExecutionRole-XXXX"
            },
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::your-bucket-name/*",
                "arn:aws:s3:::your-bucket-name"
            ]
        }
    ]
}
```

Replace:
- `YOUR_ACCOUNT_ID` with your AWS account ID
- `STACK_NAME` with your CloudFormation stack name
- `your-bucket-name` with your bucket name
- `LambdaExecutionRole-XXXX` with the actual lambda execution role ID

Apply the bucket policy:
```bash
aws s3api put-bucket-policy --bucket your-bucket-name --policy file://bucket-policy.json
```

3. Configure S3 event notification to trigger the Lambda function:

```bash
aws s3api put-bucket-notification-configuration \
  --bucket your-bucket-name \
  --notification-configuration '{
    "LambdaFunctionConfigurations": [{
        "LambdaFunctionArn": "arn:aws:lambda:REGION:ACCOUNT_ID:function:AwsLambdaEdiSplit",
        "Events": ["s3:ObjectCreated:*"],
        "Filter": {
            "Key": {
                "FilterRules": [{
                    "Name": "suffix",
                    "Value": ".edi"
                }]
            }
        }
    }]
}'
```

Replace:
- `REGION` with your AWS region
- `ACCOUNT_ID` with your AWS account ID

Note: replace the edi suffix `.edi` with your suffix or remove the filter if you have a dedicated S3 bucket.

## Usage

1. Upload an EDI file to your S3 bucket:
```bash
aws s3 cp your-edi-file.edi s3://your-bucket-name/
```

2. The Lambda function will automatically process the file and create individual transaction files in the specified output prefix location:
```
s3://your-bucket-name/split/your-edi-file_transaction_1.x12
s3://your-bucket-name/split/your-edi-file_transaction_2.x12
...
```

## Monitoring and Troubleshooting

- Monitor Lambda function execution in CloudWatch Logs
- Check CloudWatch Metrics for Lambda execution statistics
- View S3 event notifications in CloudTrail

## Limitations

- Maximum file size: 5GB (half of the Lambda ephemeral storage limit)
- Processing timeout: 15 minutes (Lambda maximum timeout)
- Memory allocation: 4GB

## Security Considerations

- The Lambda function uses IAM roles with least privilege access
- S3 bucket policy should be configured to restrict access
- Consider enabling S3 bucket encryption for sensitive EDI data
- Enable CloudTrail logging for audit purposes

## Cost Considerations

This solution uses the following AWS services that may incur costs:
- AWS Lambda invocations and compute time
- S3 storage and requests
- CloudWatch Logs storage

Monitor your AWS billing dashboard to track costs associated with this solution.
