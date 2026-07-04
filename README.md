# Amazon Bedrock RAG Knowledge Base using S3 Vectors

Build a production-ready Retrieval-Augmented Generation (RAG) chatbot using **Amazon Bedrock Knowledge Bases**, **Amazon Nova Lite**, **Amazon Titan Text Embeddings V2**, **S3 Vectors**, **AWS Lambda**, **API Gateway**, and a **React frontend**.

This project demonstrates how to build a secure, serverless AI chatbot that answers questions using your own documents stored in Amazon S3.

---

## Architecture

```
                User
                  │
                  ▼
          React Frontend
                  │
                  ▼
        Amazon API Gateway
                  │
                  ▼
            AWS Lambda
                  │
                  ▼
Amazon Bedrock Knowledge Base
        │                 │
        │                 ▼
        │        Amazon Nova Lite
        │      (Answer Generation)
        │
        ▼
Titan Text Embeddings V2
        │
        ▼
      S3 Vectors
        │
        ▼
 S3 Documents Bucket
```

---

## Features

* Amazon Bedrock Knowledge Bases
* Amazon Nova Lite foundation model
* Amazon Titan Text Embeddings V2
* Amazon S3 Vectors
* AWS Lambda
* Amazon API Gateway (HTTP API)
* React frontend
* S3 Static Website Hosting
* Retrieval-Augmented Generation (RAG)
* Serverless architecture
* Source document citations
* Production-ready IAM policies
* Manual deployment guide

---

## Technologies

| Service                  | Purpose                                 |
| ------------------------ | --------------------------------------- |
| Amazon Bedrock           | AI foundation models and Knowledge Base |
| Amazon Nova Lite         | Response generation                     |
| Titan Text Embeddings V2 | Document embeddings                     |
| Amazon S3                | Store documents                         |
| Amazon S3 Vectors        | Vector storage                          |
| AWS Lambda               | Backend API                             |
| Amazon API Gateway       | HTTP endpoint                           |
| React                    | Frontend                                |
| Python                   | Lambda runtime                          |

---

## Repository Structure

```
.
├── frontend/
│   ├── src/
│   ├── public/
│   └── deploy.sh
│
├── lambda/
│   └── lambda.py
│
├── iam/
│   ├── kb-inline-policy.json
│   ├── kb-trust-policy.json
│   ├── lambda-inline-policy.json
│   └── lambda-trust-policy.json
│
├── bedrock_rag_kb/
│   └── assets/
│
├── MANUAL_DEPLOYMENT.md
└── README.md
```

---

## Prerequisites

Before deploying, ensure you have:

* AWS Account
* Amazon Bedrock enabled
* Access to Amazon Nova Lite
* Access to Titan Text Embeddings V2
* Python 3.12
* Node.js 18+
* AWS CLI configured

---

## Deployment

Follow the complete deployment guide:

**MANUAL_DEPLOYMENT.md**

The guide walks through:

1. IAM Roles
2. S3 Bucket
3. Bedrock Knowledge Base
4. Lambda
5. API Gateway
6. Frontend Deployment

---

## Lambda Environment Variables

| Variable             | Description                           |
| -------------------- | ------------------------------------- |
| KNOWLEDGE_BASE_ID    | Bedrock Knowledge Base ID             |
| FOUNDATION_MODEL_ARN | Amazon Nova Lite Foundation Model ARN |

Example:

```text
KNOWLEDGE_BASE_ID=YOUR_KNOWLEDGE_BASE_ID

FOUNDATION_MODEL_ARN=arn:aws:bedrock:us-east-1::foundation-model/amazon.nova-lite-v1:0
```

---

## Test Request

Lambda Test Event

```json
{
  "body": "{\"query\":\"What is AI Agent Insure?\"}"
}
```

API Request

```http
POST /chat
Content-Type: application/json

{
    "query":"What is AI Agent Insure?"
}
```

---

## Example Response

```json
{
    "query":"What is AI Agent Insure?",
    "generated_response":"AI Agent Insure is ...",
    "s3_locations":[
        "s3://your-bucket/document1.pdf",
        "s3://your-bucket/document2.pdf"
    ]
}
```

---

## IAM Policies Included

The repository includes production-ready IAM policies:

* lambda-inline-policy.json
* lambda-trust-policy.json
* kb-inline-policy.json
* kb-trust-policy.json

---

## Project Workflow

```
Upload Documents
        │
        ▼
Amazon S3 Bucket
        │
        ▼
Knowledge Base Sync
        │
        ▼
Titan Embeddings V2
        │
        ▼
S3 Vectors
        │
        ▼
User Question
        │
        ▼
Lambda
        │
        ▼
RetrieveAndGenerate
        │
        ▼
Amazon Nova Lite
        │
        ▼
Response + Citations
```

---

## Security

This project follows AWS best practices:

* Least-privilege IAM roles
* Serverless architecture
* Private document retrieval
* Bedrock Knowledge Base authorization
* CORS configuration
* Environment variables for configuration

---

## Cleanup

See **MANUAL_DEPLOYMENT.md** for resource cleanup instructions.

Resources removed include:

* Bedrock Knowledge Base
* S3 Vector Bucket
* S3 Documents Bucket
* Lambda
* API Gateway
* Frontend Bucket
* IAM Roles

---

## Contributing

Contributions are welcome.

Please open an issue before submitting a pull request for significant changes.

---

## License

This project is licensed under the MIT License.
