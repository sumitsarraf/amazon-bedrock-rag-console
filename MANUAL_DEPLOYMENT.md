# Manual Deployment (AWS Console)

Step-by-step instructions for deploying the AWS infrastructure through the console. If you prefer automated deployment, see the **CDK** option in the **README.md**.

Do these steps **in this order**:

---

## Step 1 — Create the Lambda execution role

1. IAM → **Roles** → **Create role**
2. **Trusted entity type:** AWS service → **Service or use case:** Lambda → **Next**
3. **Add permissions:** attach no policies → **Next**
4. **Role name:** `rag-kb-s3-vectors-lambda-role` → **Create role**
5. On the role → **Permissions** tab → **Add permissions** → **Create inline policy** → **JSON** tab
6. Delete the placeholder, paste the full contents of **`iam/lambda-inline-policy.json`** → **Next** → **Policy name:** `rag-kb-s3-vectors-lambda-bedrock` → **Create policy**

---

## Step 2 — S3 docs bucket

7. S3 → **Create bucket** (name it and note the name)

8. Upload the 11 documents from **`bedrock_rag_kb/assets/`** into that bucket.

---

## Step 3 — Create the Knowledge Base role

9. IAM → **Roles** → **Create role**

10. **Trusted entity type:** Custom trust policy → delete the placeholder, paste the full contents of **`iam/kb-trust-policy.json`** → **Next**

11. **Add permissions:** attach no policies → **Next**

12. **Role name:** `rag-kb-s3-vectors-kb-role` → **Create role**

13. On the role → **Permissions** tab → **Add permissions** → **Create inline policy** → **JSON** tab

14. Delete the placeholder, paste the full contents of **`iam/kb-inline-policy.json`**, then replace every **`YOUR-DOCS-BUCKET-NAME`** with the bucket name from Step 7 → **Next** → **Policy name:** `rag-kb-s3-vectors-kb-s3-bedrock` → **Create policy**

---

## Step 4 — Bedrock Knowledge Base

15. Bedrock → **Knowledge bases** → **Create knowledge base**

16. Choose **S3 Vectors** as storage (the console creates the vector bucket and index automatically); use **Titan Text Embeddings V2** as the embedding model; attach the S3 documents bucket and the role **`rag-kb-s3-vectors-kb-role`**; sync the data source.

17. Wait for the sync to complete and note the **Knowledge Base ID**.

---

## Step 5 — Lambda function

18. Lambda → **Create function** (Author from scratch). **Name** it (for example `rag-kb-chat`).

19. **Execution role:** choose **Use an existing role** → select **`rag-kb-s3-vectors-lambda-role`**.

If you don't see it, you did not complete Step 1.

20. **Configuration** → **General configuration** → **Edit** → set:

* **Timeout:** 30 seconds (Bedrock calls often exceed the default 3 seconds)
* *(Optional)* **Memory:** 512 MB (can reduce latency)

21. **Configuration** → **Environment variables** → **Edit** → **Add**:

* **Key:** `KNOWLEDGE_BASE_ID`

  * **Value:** the Knowledge Base ID from Step 17

* **Key:** `FOUNDATION_MODEL_ARN`

  * **Value:** `arn:aws:bedrock:us-east-1::foundation-model/amazon.nova-lite-v1:0`

  *(Change **`us-east-1`** only if your Lambda is deployed in another AWS Region.)*

22. **Code** → replace the default handler with the contents of **`lambda/lambda.py`** → **Deploy**

---

## Step 6 — Set the Knowledge Base generation model

23. Bedrock → **Knowledge bases** → open your Knowledge Base → **Edit**

24. Set **Generative AI model** to **Amazon Nova Lite**.

25. Click **Save**.

---

## Step 7 — API Gateway

26. API Gateway → **Create API** → **HTTP API** → add a route **POST /chat** integrated with your Lambda.

27. Ensure you have a stage with **Auto-deploy** enabled (required to get an Invoke URL):

* In the HTTP API console → **Stages**
* Open **`$default`** (it is often created automatically) and enable **Auto-deploy**
* If **`$default`** does not exist, create it and enable **Auto-deploy**
* *(Alternative)* create a stage such as **`prod`** with **Auto-deploy** (your Invoke URL will include `/prod`)

28. Enable CORS:

* In the HTTP API console → **CORS** tab → **Configure CORS**

* **Access-Control-Allow-Origin**

  * Use `*` for a demo, or specify a frontend origin:

    * Local development: `http://localhost:3000`
    * S3 website: `http://<your-frontend-bucket>.s3-website-us-east-1.amazonaws.com`

* **Access-Control-Allow-Methods**

  * POST
  * OPTIONS

* **Access-Control-Allow-Headers**

  * Content-Type

* Click **Save**

29. Note the **Invoke URL**.

Your final endpoint is:

* `$default` stage:

  * `<invoke-url>/chat`

* `prod` stage:

  * `<invoke-url>/prod/chat`

---

## Step 8 — Test the Lambda

30. In the Lambda console → **Test** tab → **Create new event**

31. Paste the following JSON:

```json
{
  "body": "{\"query\": \"What is AI Agent Insure?\"}"
}
```

32. Click **Save** → **Test**

33. You should receive **Status Code 200** with a JSON response containing:

* `generated_response`
* `s3_locations`

If you receive **502**, verify the following:

* The Lambda is using **`rag-kb-s3-vectors-lambda-role`**.
* The IAM policy allows **`bedrock:InvokeModel`** for **Amazon Nova Lite**.
* **Amazon Nova Lite** is enabled under **Amazon Bedrock → Model access**.
* The **`FOUNDATION_MODEL_ARN`** environment variable exactly matches:

```
arn:aws:bedrock:us-east-1::foundation-model/amazon.nova-lite-v1:0
```

* The **`KNOWLEDGE_BASE_ID`** is correct.

---

## Step 9 — S3 static website (Optional, for frontend)

34. Deploy the frontend to S3 (static website).

Ensure **`frontend/.env`** contains:

```text
VITE_API_URL=<your-api-gateway-endpoint>/chat

REGION=us-east-1

BUCKET=<your-frontend-bucket-name>
```

35. From the repository root, run:

```bash
cd 6_Bedrock_RAG_KB_S3_Vectors/frontend

npm install

./deploy.sh
```

This script:

* Builds the application
* Creates the bucket if it doesn't already exist
* Enables S3 static website hosting
* Configures SPA routing (`index.html`)
* Makes the bucket publicly readable
* Uploads the `dist/` folder

36. Open the generated website URL, which looks similar to:

```
http://<bucket>.s3-website-us-east-1.amazonaws.com
```

37. If you receive a browser CORS error, update your API Gateway HTTP API CORS configuration to allow your website origin (or use `*` for testing).

---

# Step 10 — (Optional) Manual Tear Down

Delete the AWS resources in the following order:

1. Bedrock Knowledge Base (including the data source)
2. S3 Vector index
3. S3 Vector bucket
4. S3 documents bucket (empty first, then delete)
5. Lambda function
6. API Gateway HTTP API
7. S3 frontend bucket (empty first, then delete)
8. IAM roles
