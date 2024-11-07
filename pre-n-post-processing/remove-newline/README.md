# EDI File Line Break Removal

This CloudFormation template deploys a Lambda function that processes EDI files stored in S3 by removing carriage returns (CR) and line feeds (LF). This is particularly useful when dealing with EDI files that need to be processed as a single line.

## Overview

The solution consists of:
- A Lambda function that removes CR/LF characters from files
- IAM role and policies for Lambda execution
- S3 bucket configuration for file processing

## Prerequisites

- AWS CLI installed and configured
- Appropriate AWS permissions to create:
  - Lambda functions
  - IAM roles
  - S3 buckets
  - CloudFormation stacks

## Deployment Steps
### Deploy via AWS Cloudformation console

1. Sign in to the AWS Management Console and navigate to the CloudFormation service

2. Click "Create stack" and choose "With new resources (standard)"

3. In the "Specify template" section:
   - Select "Upload a template file"
   - Click "Choose file" and upload the `remove-newline.yaml` template
   - Click "Next"

4. In the "Specify stack details" section:
   - Enter a stack name (e.g., "edi-line-removal")
   - Fill in the required parameters:
     - S3BucketName: Enter the name of your S3 bucket
     - S3OutputPrefix: Specify the prefix for processed files (e.g., "processed/")
   - Click "Next"

5. In the "Configure stack options" section:
   - Leave the default settings or adjust as needed
   - Click "Next"

6. In the "Review" section:
   - Review all the settings
   - Check the acknowledgment box for IAM resource creation
   - Click "Create stack"

7. Wait for the stack creation to complete (this typically takes 2-3 minutes)

### Deploy via AWS CLI

1. Clone or download the CloudFormation template (`remove-newline.yaml`)

2. Deploy the stack using AWS CLI:
```bash
aws cloudformation create-stack \
  --stack-name edi-line-removal \
  --template-body file://remove-newline.yaml \
  --parameters \
    ParameterKey=S3BucketName,ParameterValue=your-bucket-name \
    ParameterKey=S3OutputPrefix,ParameterValue=processed/ \
  --capabilities CAPABILITY_IAM
```

Replace `your-bucket-name` with your S3 bucket name and adjust the output prefix as needed.

3. Wait for the stack creation to complete:
```bash
aws cloudformation wait stack-create-complete --stack-name edi-line-removal
```

## S3 Bucket Configuration

### 1. Create S3 Event Trigger

After deploying the stack, configure your S3 bucket to trigger the Lambda function:

1. Go to the S3 console and select your bucket
2. Navigate to the "Properties" tab
3. Find the "Event notifications" section and click "Create event notification"
4. Configure the event:
   - Event name: `EDIProcessing`
   - Event types: Select "All object create events"
   - Destination: Select "Lambda function"
   - Lambda function: Select the function created by CloudFormation (starts with "edi-line-removal-ProcessS3ObjectLambda")

### 2. Add Bucket Policy

Add the following bucket policy to allow the Lambda function to access your S3 bucket:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowLambdaAccess",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::YOUR_ACCOUNT_ID:role/edi-line-removal-LambdaExecutionRole"
            },
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:AbortMultipartUpload",
                "s3:ListMultipartUploadParts",
                "s3:ListBucketMultipartUploads"
            ],
            "Resource": [
                "arn:aws:s3:::your-bucket-name/*"
            ]
        }
    ]
}
```

Replace:
- `YOUR_ACCOUNT_ID` with your AWS account ID
- `your-bucket-name` with your S3 bucket name

To add this policy:
1. Go to the S3 console and select your bucket
2. Navigate to the "Permissions" tab
3. Find "Bucket policy" and click "Edit"
4. Paste the modified policy and click "Save changes"

## Testing the Solution

1. Upload an EDI file to your S3 bucket:
```bash
aws s3 cp your-edi-file.edi s3://your-bucket-name/
```

2. The Lambda function will automatically process the file and save it to the specified output prefix
3. Check the processed file:
```bash
aws s3 cp s3://your-bucket-name/processed/your-edi-file.edi ./processed-file.edi
```

## Additional Considerations

- The Lambda function has a timeout of 15 minutes and 1024MB of memory
- Files are processed using multipart upload with 8MB chunks
- The function preserves the original file name but prepends the specified output prefix
- Monitor CloudWatch Logs for any processing errors
- The solution can handle large files through streaming and multipart uploads
- Original files remain unchanged; processed files are created in the output prefix location

## Cleanup

To remove the deployed resources:
```bash
aws cloudformation delete-stack --stack-name edi-line-removal
```

Note: This will not delete your S3 bucket or its contents. You'll need to remove those separately if desired.
