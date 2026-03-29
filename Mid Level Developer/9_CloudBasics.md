# Cloud Basics (AWS) – Mid-Level Developer Interview Reference

---

## ☁️ Section 1: AWS Core Services & Serverless Architecture

---

### 1. What is "Serverless" and how does AWS Lambda work?

**Definition:** Serverless is a cloud computing model where the cloud provider (AWS) manages the infrastructure. You only provide the code (the "Lambda function").

**Internal Mechanics:**
1.  **Event-Driven:** Lambda is triggered by an event (HTTP request, S3 upload, DynamoDB change).
2.  **Stateless:** Every execution starts fresh. You cannot save state to the local disk.
3.  **Automatic Scaling:** If 1,000 requests come at once, AWS spawns 1,000 "micro-containers" to handle them.
4.  **Pay-per-Execution:** You only pay for the milliseconds your code is running.

---

### 2. What are "Cold Starts" and how do you minimize them?

**Problem:** When a Lambda hasn't been used for a while, AWS shuts down its container. The next request triggers a "Cold Start" where AWS must pull your code, start a new container, and boot the runtime. This can take 1–5 seconds.

**Solution:**
1.  **Memory Increase:** Larger memory allocation proportionally increases CPU power, speeding up the boot.
2.  **Provisioned Concurrency:** You pay a small fee to keep X containers "warm" at all times.
3.  **Lambda Layers:** Keep your main function package small by moving heavy dependencies (like `sharp` or `node_modules`) to a Layer.
4.  **Language Choice:** Go/Rust have faster cold starts than Java/Node.

---

### 3. SQS (Simple Queue Service) vs SNS (Simple Notification Service)

-   **SNS (Fan-out):** One Message → Many Subscribers. When a "New User" is created, SNS can tell the Email service, the Analytics service, and the Welcome-SMS service all at once.
-   **SQS (One-to-One):** One Message → One Consumer. Used for **Decoupling**. If the "Thumbnail Generator" is slow, SQS holds the messages until the generator catches up.

---

### 4. IAM (Identity and Access Management): Roles vs Policies

**Least Privilege Principle:** Every user or service should have the **absolute minimum** permissions required to do its job.

1.  **Policy:** A JSON document that defines "Who" can do "What" (e.g., `Allow S3:Read on BucketX`).
2.  **Role:** An "Identity" that can be assumed by a service (like a Lambda). You attach the policy to the Role, and then tell the Lambda to use that Role.

---

### 5. S3 Storage Classes: Optimizing for Cost

1.  **Standard:** High speed, high cost. Best for Frequently Accessed images/videos.
2.  **Standard-IA (Infrequent Access):** Lower storage cost, but you pay a fee per retrieval. Best for backups or reports from last month.
3.  **Glacier / Glacier Deep Archive:** Extremely cheap but retrieval takes 1 minute to 15 hours. Best for compliance logs that you hope to never read.

---

### 6. VPC (Virtual Private Cloud) & Subnets

**Public Subnet:** Has an internet gateway. Used for Load Balancers or public-facing servers.
**Private Subnet:** Not directly accessible from the internet. Used for **Databases** and **Private APIs**.

**NAT Gateway:** Allows a server in a private subnet (like your DB) to "talk out" to the internet (e.g., to download an OS update) while still preventing the internet from talking directly to it.

---

### 7. What is a "Dead Letter Queue" (DLQ)?

**Problem:** A "Poisoned Message" enters your SQS queue. Your Lambda keeps trying to process it, fails, and the message goes back to the queue (looping endlessly).

**The Solution:**
Set a "Redrive Policy". If a message fails 3 times, move it to a **DLQ (Dead Letter Queue)**. This allows your main system to continue running while you manually inspect the failed message in the DLQ later.

---

### 8. Cloudfront: Edge Locations vs Regional Cache

**Edge Location:** The "entry point" closet to the user (hundreds globally). It caches static assets (JS/CSS/Images).
**Regional Edge Cache:** A larger cache that sits between the Edge Location and your server. If an asset isn't in Tokyo's edge, it checks the Asia Regional Cache before hitting your server in Oregon.

---

### 9. Secrets Manager vs Parameter Store

-   **Parameter Store:** Great for non-sensitive config like `API_URL` or `FEATURE_FLAG`. (Free in most cases).
-   **Secrets Manager:** Built specifically for passwords/keys.
    -   *Killer Feature:* **Automatic Rotation**. It can automatically change your RDS database password every 30 days and update your Lambda config without any downtime.

---

### 10. API Gateway: REST vs HTTP APIs

-   **HTTP APIs (Recommended for Lambda):** 70% cheaper and 50% faster. Best for simple Lambda proxying.
-   **REST APIs:** More features (Edge-optimized, caching, Request/Response mapping). Use only if you need advanced features.

---

### 11. Monitoring: CloudWatch Dashboards & Alarms

**Mid-Level Goal:** Don't wait for users to complain.
1.  **Metrics:** Monitor "4XX and 5XX Error Rates".
2.  **Alarms:** Set an alarm to send a Slack message if the 5XX rate goes above 1% for 5 minutes.
3.  **Traces (AWS X-Ray):** Visualize exactly which service (DB vs external API) is making a request slow.

---

### 12. AWS Cognito: User Pools vs Identity Pools

-   **User Pools:** A "User Directory". It handles Registration, Login, and Password recovery. It provides a JWT token to the app. (Who you are).
-   **Identity Pools:** Allows users (from User Pools or Social logins) to get **AWS Credentials**. 
    -   *Use Case:* Allowing a mobile user to upload a photo **directly** to S3 using a temporary IAM permission. (What you can do in AWS).

---

### 13. S3: Pre-signed URLs for Secure Access

**Scenario:** You have a private bucket of "Paid Invoices". You don't want to make it public.
-   **Solution:** Your API generates a "Pre-signed URL" that is valid for only 15 minutes. 
-   The user clicks the link, downloads the file directly from S3, and after 15 minutes, the link dies.
-   **Benefit:** Zero traffic/bandwidth hits your Node server. S3 handles the heavy lifting safely.

---

### 14. CloudFront: Signed URLs vs Signed Cookies

-   **Signed URLs:** Access control for a **Single File**. Best for a single private video or photo.
-   **Signed Cookies:** Access control for **Multiple Files** (e.g., an entire folder of premium courses). The user gets a cookie once, and then they can see all files in that directory seamlessly.

---

### 15. Lambda Layers

**Problem:** You have 50 different Lambda functions that all use the same `database-client` and `error-logger` code.
**Solution:** Put that code into a **Lambda Layer**. 
1.  It reduces the "Deployment Package Size" of your functions.
2.  AWS caches the layer internally, leading to slightly faster Cold Starts.
3.  Makes code updates easier as you only update the Layer, not 50 ZIP files.

---

### 16. Scaling DBs: RDS Aurora Serverless v2

**Why choose it?**
Unlike standard RDS where you pick a fixed CPU/RAM (e.g., `db.t3.medium`), Aurora Serverless **instantly** scales CPU and RAM up/down based on demand.
-   *Benefit:* During a "Black Friday" traffic spike, it scales from 2GB to 64GB of RAM in seconds without any downtime, and scales back down when the rush is over.

---

### 17. Security: AWS WAF (Web Application Firewall)

**Where it sits:** In front of your CloudFront or ALB (Load Balancer).
**What it does:** It blocks "Layer 7" attacks like:
1.  **SQL Injection.**
2.  **Cross-Site Scripting (XSS).**
3.  **Geo-Blocking:** Blocking traffic from certain countries.
4.  **Bot Protection:** Blocking scrapers that hit your API 500 times per second.

---

### 18. Elasticache (Redis) vs DynamoDB (DAX)

-   **Elasticache:** A dedicated Redis cluster. Best for caching SQL queries or Session data. You manage the "Nodes".
-   **DAX (DynamoDB Accelerator):** A "Write-through" cache specifically for DynamoDB. 
    -   *Benefit:* It's 100% transparent. Your code just queries DynamoDB, and DAX automatically caches the result in memory for 1ms response times.

---

### 19. Cost Optimization: Spot Instances vs Savings Plans

-   **Spot Instances:** Up to 90% discount! 
    -   *The catch:* AWS can terminate them with 2 minutes notice if they need the capacity back. 
    -   *Best for:* Background workers, video processing, or Dev environments. Never use for a production API.
-   **Savings Plans:** 30–50% discount if you commit to spending `$X per hour` for 1-3 years. Best for stable Production workloads.

---

### 20. Disaster Recovery: RTO vs RPO

-   **RTO (Recovery Time Objective):** "How long can we stay down?" (e.g., "We must be up within 1 hour").
-   **RPO (Recovery Point Objective):** "How much data can we lose?" (e.g., "We can lose up to 15 minutes of data").
-   **Mid-Level Goal:** You design your architecture (Backups, Multi-region) to meet these two numbers provided by the business.

---

*Expansion Complete*
