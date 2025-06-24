
# 📝 Serverless Blog Generation with AWS Bedrock, Lambda, and API Gateway

This project demonstrates how to build a **serverless application** on AWS that leverages **Amazon Bedrock** to generate blog posts on any given topic. The application is exposed via an **API Gateway** endpoint, which triggers a **Lambda function** to invoke the **Llama 3 70B Instruct** model. The generated blog is then automatically saved to an **Amazon S3** bucket.

---

## 🧱 Project Architecture

The application follows a simple, event-driven, serverless architecture:

```
[Client (e.g., Postman)] 
    --> [API Gateway (HTTP API)] 
        --> [AWS Lambda Function] 
            --> [Amazon Bedrock (Llama 3)]
                |
                '--> [Amazon S3 (Storage)]
```

**Workflow:**
1. A user sends a POST request with a blog topic to an API Gateway endpoint.
2. API Gateway triggers the AWS Lambda function, passing the topic.
3. The Lambda function invokes the Amazon Bedrock service and sends a prompt to the `meta.llama3-70b-instruct-v1:0` model to generate a blog post.
4. Once generated, the Lambda function saves the content as a `.txt` file to a specified S3 bucket.
5. A success message is returned to the user.

---

## ✨ Features

- **Serverless**: No servers to manage; scales automatically.
- **Generative AI**: Uses Amazon Bedrock (Llama 3) for high-quality content generation.
- **API-driven**: Easily integrable with any frontend.
- **Scalable Storage**: Amazon S3 stores generated blogs reliably.

---

## ✅ Prerequisites

- AWS Account with access to IAM, Lambda, API Gateway, S3, and Amazon Bedrock
- Python 3.9+ installed locally
- AWS CLI configured
- API testing tool (e.g., Postman or curl)

---

## 🚀 Setup and Deployment

### 1. Enable Model Access in Amazon Bedrock
- Go to the Bedrock Console → *Model access*
- Request access to **Meta Llama 3**
- Wait for approval

### 2. Create IAM User for Programmatic Access
- Go to IAM → Users → Create User
- Name: `bedrock-app-user`
- Grant access using **AdministratorAccess** policy (for demo; restrict in production)
- Generate and download access keys

### 3. Configure AWS CLI
```bash
aws configure
# Provide Access Key, Secret, Region (e.g., us-east-1), and format (json)
```

### 4. Create the S3 Bucket
- Go to S3 Console → *Create bucket*
- Bucket name: `aws-bedrock-blog-output-yourname`
- Default settings are fine

### 5. Create Lambda Layer for Latest boto3
```bash
mkdir -p python/lib/python3.12/site-packages
pip install boto3 -t python/lib/python3.12/site-packages
zip -r boto3-layer.zip python
```
- Go to Lambda Console → Layers → *Create Layer*
- Name: `boto3_update_layer`
- Upload `boto3-layer.zip`
- Runtime: Python 3.12

### 6. Create Lambda Function
- Author from Scratch → Name: `blog-generator-function`
- Runtime: Python 3.12
- Let Lambda create a new role
- Paste the provided code in the editor

### 7. Attach Layer to Lambda
- In Lambda, scroll to Layers → *Add Layer*
- Select `boto3_update_layer`

### 8. Set Lambda Execution Permissions
- Go to Lambda → Configuration → Permissions → Role name
- Attach the following policies:
  - `AmazonS3FullAccess`
  - `AmazonBedrockFullAccess`

---

## 🌐 Create the API Gateway

1. Go to API Gateway Console → *Create API*
2. Choose HTTP API
3. Name: `BedrockBlogAPI`
4. Configure POST route `/blog-generation`
5. Integration type: Lambda → `blog-generator-function`
6. Deploy and note the **Invoke URL**

---

## 📦 Usage

- **Method**: POST  
- **URL**: `https://your-api-url.amazonaws.com/blog-generation`  
- **Body (raw JSON)**:
```json
{
  "blog_topic": "The future of machine learning and generative AI"
}
```

- **Response**:
```json
{
  "statusCode": 200,
  "body": "Blog Generation is completed"
}
```

---

## 📁 Verification

- Navigate to the S3 bucket you created
- Go to `blog-output/` folder
- A `.txt` file will be there named by timestamp (e.g., `184654.txt`)
- Download to read the generated blog post

---

## 🧠 Code Explanation

### `blog_generate_using_bedrock(blogtopic: str) -> str`
- Accepts the blog topic
- Constructs a Bedrock-compatible prompt
- Invokes `meta.llama3-70b-instruct-v1:0` model via `boto3`
- Extracts the generated text
- Returns blog content

### `save_blog_details_s3(s3_key, s3_bucket, generate_blog)`
- Uploads blog text to S3 bucket
- Uses `put_object` method from `boto3`

### `lambda_handler(event, context)`
- Parses blog topic from the API Gateway request
- Calls blog generation function
- Saves output to S3
- Returns HTTP 200 response

---

## 📦 requirements.txt

```
boto3
```

---

## 👤 Author

**Hithesh Alle**  
B.Tech CSE, VIT Vellore  
