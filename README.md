# Lambda Basic Auth

An AWS Lambda@Edge function that adds HTTP Basic Authentication to a CloudFront distribution backed by an S3 bucket.

## Architecture

```
Browser → CloudFront → Lambda@Edge (basic-auth) → S3 Bucket
```

- CloudFront uses Origin Access Control (OAC) to access the S3 bucket
- Lambda@Edge intercepts viewer requests and enforces Basic Auth

## Contents

| File | Description |
|------|-------------|
| `basic-auth.mjs` | Lambda@Edge handler (ES module) |
| `public/` | Static assets served from S3 |

## Deployment

1. Create an S3 bucket and set up a CloudFront distribution with OAC
2. Deploy the Lambda function to `us-east-1` (required for Lambda@Edge)
3. Configure the Lambda execution role to be assumable by both `lambda.amazonaws.com` and `edgelambda.amazonaws.com`
4. Publish a numbered version of the Lambda function
5. Attach the versioned Lambda ARN as a CloudFront viewer-request trigger

## S3 Bucket Policy (OAC)

```json
{
  "Version": "2012-10-17",
  "Statement": {
    "Sid": "AllowCloudFrontServicePrincipalReadOnly",
    "Effect": "Allow",
    "Principal": { "Service": "cloudfront.amazonaws.com" },
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::your-bucket/*",
    "Condition": {
      "StringEquals": {
        "AWS:SourceArn": "arn:aws:cloudfront::<account-id>:distribution/<dist-id>"
      }
    }
  }
}
```

## IAM Trust Policy for Lambda@Edge Role

The execution role must trust both Lambda and Lambda@Edge principals:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Allow", "Principal": { "Service": "lambda.amazonaws.com" }, "Action": "sts:AssumeRole" },
    { "Effect": "Allow", "Principal": { "Service": "edgelambda.amazonaws.com" }, "Action": "sts:AssumeRole" }
  ]
}
```
