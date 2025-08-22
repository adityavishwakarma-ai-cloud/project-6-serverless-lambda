## 1️⃣ Folder Structure

```
project-6-serverless-lambda/
│-- README.md
│-- lambda/
│   ├── process-file.js         # Lambda function
│   └── package.json            # Node.js dependencies (if needed)
│-- templates/
│   └── s3-trigger.yaml         # CloudFormation template for S3 trigger
│-- .gitignore
```

---

## 2️⃣ Lambda Function – `lambda/process-file.js`

```javascript
const AWS = require('aws-sdk');
const s3 = new AWS.S3();

exports.handler = async (event) => {
    console.log('Received event:', JSON.stringify(event, null, 2));

    const bucket = event.Records[0].s3.bucket.name;
    const key = decodeURIComponent(event.Records[0].s3.object.key.replace(/\+/g, " "));

    try {
        const params = { Bucket: bucket, Key: key };
        const data = await s3.getObject(params).promise();
        
        // Example processing: convert text to uppercase
        const processedData = data.Body.toString('utf-8').toUpperCase();

        // Save processed data to another bucket
        const destParams = {
            Bucket: 'processed-files', // Ensure this bucket exists
            Key: key,
            Body: processedData
        };
        await s3.putObject(destParams).promise();

        console.log(`Processed file saved to processed-files/${key}`);
        return { status: 'Success' };
    } catch (err) {
        console.error(err);
        throw new Error('Error processing file');
    }
};
```

---

## 3️⃣ Optional Node.js `package.json` (only if external packages needed)

```json
{
  "name": "serverless-lambda",
  "version": "1.0.0",
  "description": "Lambda function for S3 file processing",
  "main": "process-file.js",
  "dependencies": {
    "aws-sdk": "^2.1450.0"
  }
}
```

> Note: For basic AWS SDK usage, Lambda runtime already includes it, so this is optional unless you need other npm packages.

---

## 4️⃣ CloudFormation Template – `templates/s3-trigger.yaml`

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Serverless Lambda with S3 trigger

Resources:

  IncomingBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: incoming-files

  ProcessedBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: processed-files

  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: LambdaS3Role
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
        - arn:aws:iam::aws:policy/AmazonS3FullAccess

  ProcessFileLambda:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: ProcessFile
      Handler: process-file.handler
      Role: !GetAtt LambdaExecutionRole.Arn
      Runtime: nodejs18.x
      Code:
        S3Bucket: <your-s3-bucket-for-lambda-code>
        S3Key: process-file.zip
      MemorySize: 128
      Timeout: 10

  S3EventTrigger:
    Type: AWS::Lambda::Permission
    Properties:
      FunctionName: !Ref ProcessFileLambda
      Action: lambda:InvokeFunction
      Principal: s3.amazonaws.com
      SourceArn: !GetAtt IncomingBucket.Arn

Outputs:
  IncomingBucketName:
    Value: !Ref IncomingBucket
  ProcessedBucketName:
    Value: !Ref ProcessedBucket
  LambdaFunctionName:
    Value: !Ref ProcessFileLambda
```

> **Note:**
>
> * Replace `<your-s3-bucket-for-lambda-code>` with the bucket where you’ll upload `process-file.zip`.
> * Zip the `process-file.js` (and optionally `package.json`) before deploying.

---

## 5️⃣ How to Deploy

### a) Zip Lambda code

```bash
cd lambda
zip -r process-file.zip process-file.js package.json
```

### b) Upload to S3

```bash
aws s3 cp process-file.zip s3://<your-s3-bucket-for-lambda-code>/
```

### c) Deploy CloudFormation stack

```bash
aws cloudformation create-stack \
    --stack-name serverless-lambda-stack \
    --template-body file://templates/s3-trigger.yaml \
    --capabilities CAPABILITY_NAMED_IAM
```

### d) Test

* Upload a file to `incoming-files` bucket.
* Lambda triggers automatically and saves the processed file to `processed-files` bucket.
* Check **CloudWatch logs** for execution details.

---

✅ Project 6 is now **fully packaged and deployable**.
It demonstrates **serverless architecture, automation, event-driven computing, and cloud-native design**.
