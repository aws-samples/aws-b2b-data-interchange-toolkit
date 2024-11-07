# Line Wrapping Lambda Function

This CloudFormation template deploys an AWS Lambda function that processes files in an S3 bucket by wrapping lines to 80 characters. When a file is processed, the Lambda function creates a new file with wrapped lines in the specified output prefix location in the output bucket.

## Features

- Automatically wraps text files to 80 characters per line (can be adjusted to other numbers in the YAML file)
- Processes files from S3 bucket events
- Configurable output location via prefix
- High performance configuration with 4GB memory and 15-minute timeout
- 5GB ephemeral storage for processing large files

## Prerequisites

- An AWS account
- An existing S3 bucket or permissions to create one
- Access to AWS CloudFormation in your desired region

## Parameters

The template requires two parameters:

- `S3BucketName`: The name of the S3 bucket where files will be processed
- `S3OutputPrefix`: The prefix (folder path) where processed files will be stored (e.g., "wrapped/")

## Deployment Instructions

Follow these steps to deploy the solution via the AWS Console:

1. **Open CloudFormation**
   - Log into your AWS Console
   - Navigate to CloudFormation service
   - Click "Create stack" > "With new resources (standard)"

2. **Upload Template**
   - Select "Upload a template file"
   - Click "Choose file" and select the `wrap-lines.yaml` file
   - Click "Next"

3. **Specify Stack Details**
   - Enter a Stack name (e.g., "line-wrapper")
   - Under Parameters:
     - Enter your S3 bucket name in `S3BucketName`
     - Enter your desired output prefix in `S3OutputPrefix` (e.g., "wrapped/")
   - Click "Next"

4. **Configure Stack Options**
   - You can leave the default options
   - Click "Next"

5. **Review**
   - Review your configuration
   - Check the acknowledgment box for IAM resource creation
   - Click "Create stack"

6. **Wait for Completion**
   - Stack creation takes about 2-3 minutes
   - Status will change to "CREATE_COMPLETE" when finished

## How It Works

The deployed Lambda function:

1. Triggers when files are added to the specified S3 bucket
2. Downloads the source file to temporary storage
3. Processes the file by wrapping lines to 80 characters
4. Uploads the processed file to the specified output prefix
5. Cleans up temporary files

## Outputs

The stack provides two outputs:

- `LambdaFunctionName`: The name of the created Lambda function
- `S3BucketName`: The name of the S3 bucket being used

## Testing

To test the deployment:

1. Upload a text file to your S3 bucket
2. The Lambda function will automatically process it
3. Check the output prefix location in your bucket for the processed file

## IAM Permissions

The template creates an IAM role with:
- Basic Lambda execution permissions
- S3 permissions to read from and write to the specified bucket

## Limitations

- Maximum file size is limited by Lambda's temporary storage (5GB)
- Processing time must complete within 15 minutes
- Only processes text-based files effectively
