


❌ **Skipping Billing Alerts**
👉 Risk of accidental charges. Always set up a cost budget with email alerts.

❌ **Using Large EC2 or RDS Instances**
👉 Stick to **t2.micro** or **free tier eligible types** only.

❌ **Deploying from Root User**
👉 Create an **IAM user** for daily activities with limited permissions.

❌ **Forgetting to Stop/Terminate Resources**
👉 EC2, RDS, and even EBS volumes can incur charges if left running.

❌ **Storing Large Files in S3 Without Cleanup**
👉 Free tier includes **5GB** — monitor S3 usage regularly.

❌ **Missing Activation Email**
👉 Your account isn’t ready until you receive confirmation from AWS.

❌ **Choosing Paid Support Plan**
👉 Select **Basic Support (Free)** during sign-up.

❌ **Not Monitoring Usage**
👉 Use the **Billing Dashboard** to track monthly usage.


# S3 -

- **Bucket:** Like a folder. Top-level container for your files (objects).
- **Object:** Any file you upload (with metadata and key).
- **Key:** Unique name for the object in a bucket.
- **Region:** AWS region where your bucket resides.
- **Storage Classes:** Different cost/durability options like Standard, Intelligent-Tiering 
- **Public Access:** Controls who can access the bucket or files.

connect s3 to spring boot -

we need aws SDK. S3 client is class provided by aws sdk in order to perform operation on s3.

IAM user banke policies attached kr lete hai. phir us user ko s3 se connect krte hai. (s3 ko root user se bhi kr sakte hai). Ab es IAM user ke behalf pe aws ke services ko access krne ke liye access key and secret key chahiye hota hai.

**S3 Presigned URL** is a temporary, secure link that gives someone access to a specific file in an Amazon S3 bucket, even if that file or bucket is completely private.

S3Client -- upload the file
S3Presigner -- get pre-sign url

