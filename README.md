# ⚡ Project 6: Serverless Application with AWS Lambda & S3

## 📌 Project Overview

This project demonstrates a **serverless architecture** using **AWS Lambda**, **S3**, and **API Gateway**.
The main goal is to process data automatically without managing servers, using cloud-native serverless services.

Key highlights:

* Event-driven architecture using **S3 triggers**
* **Lambda functions** to process or transform data
* Optional **API Gateway** to expose serverless endpoints
* Fully **scalable, cost-efficient, and zero server maintenance**

---

## 🛠️ Tech Stack / Tools Required

* **AWS Lambda** (Serverless compute)
* **Amazon S3** (Storage & Event triggers)
* **Amazon API Gateway** (Optional – HTTP endpoints)
* **AWS CLI / SDK**
* **Node.js / Python** (Lambda function runtime)
* **IAM Roles** (Secure access between services)

---

## 📂 Functional Requirements

* Trigger a Lambda function whenever a file is uploaded to S3.
* Process uploaded files (e.g., rename, resize images, convert formats, or analyze data).
* Store processed results back in S3.
* Optional: expose Lambda function via API Gateway for external requests.

---

## 📂 Non-Functional Requirements

* Ensure **secure access** using IAM roles.
* Maintain **scalability** – Lambda scales automatically with events.
* Minimize **cost** – pay only for executed Lambda invocations.
* Provide **error handling** and logging using AWS CloudWatch.

---

## 🔄 Project Flow

1. **Upload file** to S3 bucket (e.g., `incoming-files`).
2. **S3 triggers Lambda function** automatically.
3. **Lambda function executes**:

   * Reads the file
   * Processes data (e.g., transformation, validation)
   * Writes processed result to another S3 bucket (e.g., `processed-files`)
4. **Optional API Gateway endpoint** allows external triggering of Lambda.
5. **CloudWatch logs** capture processing details and errors.

---

## 📝 Sample Lambda Function (Node.js)

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
            Bucket: 'processed-files',
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

## 📂 Folder Structure

```
project-6-serverless-lambda/
│-- README.md
│-- lambda/
│   └── process-file.js     # Lambda function
│-- templates/
│   └── s3-trigger.yaml     # Optional CloudFormation template
│-- .gitignore
```

---

## ⚡ Deployment Steps

1. **Create S3 buckets**:

```bash
aws s3 mb s3://incoming-files
aws s3 mb s3://processed-files
```

2. **Create IAM role** for Lambda with S3 read/write permissions.

3. **Deploy Lambda function** using AWS Console, CLI, or SAM/CloudFormation:

```bash
aws lambda create-function \
  --function-name ProcessFile \
  --runtime nodejs18.x \
  --role <IAM_ROLE_ARN> \
  --handler process-file.handler \
  --zip-file fileb://lambda.zip
```

4. **Add S3 trigger** to Lambda on `incoming-files` bucket.

5. **Test** by uploading a file to `incoming-files`.

6. **Monitor** Lambda execution via **CloudWatch logs**.

---

## 📝 Resume Highlights

**Objective**
Built a **serverless application** to process files automatically using AWS Lambda and S3.

**Key Achievements**

* Implemented **event-driven processing** with S3 triggers.
* Ensured **scalability and cost efficiency** using serverless architecture.
* Enabled **error handling and logging** via AWS CloudWatch.

**Impact**
✅ Reduced manual file processing by 100%.
✅ Fully automated serverless workflow with zero server management.
✅ Optimized cloud resource utilization and cost.

---

## 🔗 Repository Link

```md
https://github.com/<your-username>/project-6-serverless-lambda
```
