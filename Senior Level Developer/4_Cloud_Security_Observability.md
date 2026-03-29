# Module 4: Observability, Cloud & Security

This module covers the operational side of senior engineering: how to see into your systems, how to leverage cloud infrastructure efficiently at the highest scales, and how to build a resilient, "Zero Trust" security posture.

---

## Observability & Monitoring

### 41. How do you transition a company from reactive alerts to proactive SLOs/Error Budgets?
**Context**: Traditional monitoring relies on static thresholds (e.g., "Alert if CPU > 90%"). This causes alert fatigue and wakes engineers up for issues that don't actually impact the customer.

**Deep Dive**:
- **SLIs (Service Level Indicators)**: Measure what the user experiences. Typically: Latency (e.g., p99 response time) and Error Rate (e.g., 5xx responses).
- **SLOs (Service Level Objectives)**: Establish a target for the SLI. (e.g., "99.9% of requests will respond in under 200ms over a 30-day window").
- **Error Budgets (The Key Concept)**: If the SLO is 99.9%, you are allowed a 0.1% failure rate. This is your "Error Budget."
- **The Cultural Shift**: Instead of alerting on CPU spikes, you alert on *Error Budget Burn Rate*. If a deployment burns 20% of your 30-day budget in one hour, trigger a critical page. If your budget drops to zero, feature development stops, and the team must exclusively focus on reliability/tech debt until the 30-day rolling window recovers the budget.
- **Trade-offs**: Requires massive organizational buy-in from Product Managers, who must agree to halt product development when the budget is spent.

### 42. What is OpenTelemetry (OTel), and why is it replacing native APMs?
**Context**: Vendor lock-in with monitoring tools (Datadog, New Relic) is extremely expensive. Swapping them out traditionally required rewriting thousands of logging lines in the codebase.

**Deep Dive**:
- **The Concept**: OTel is an open-source observability framework (APIs, SDKs, and a central Collector agent). It provides a *vendor-agnostic* standard for generating, collecting, and exporting telemetry data (Metrics, Logs, Traces - MELT).
- **The Architecture**: 
    1. *Instrumentation*: Your code imports the OTel SDK, not the Datadog SDK.
    2. *The Collector*: A sidecar or daemonset process. Your app sends data to the Collector.
    3. *Exporters*: The Collector pipeline transforms the data and exports it. You configure the collector to send traces to Jaeger, metrics to Prometheus, and logs to Datadog.
- **Real-World Applicability**: Allows a company to change their SaaS monitoring vendor seamlessly without ever touching the application code, leveraging incredible negotiating power over vendor contracts.

### 43. How do you implement Distributed Tracing context propagation?
*(Expanded from Module 3 for the Observability context).*
**Context**: Without distributed tracing, debugging a microservices outage is like reading 10 separate books simultaneously to piece together a single story.

**Deep Dive**:
- A **Trace** represents the unbroken lifecycle of a request. A **Span** represents bounds of work within a single service (e.g., a database query).
- **Context Injection**: Use the `W3C Trace Context` specification. The API Gateway creates a `traceparent` header (e.g., `00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01`).
    - The first part is the format version.
    - The second part is a globally unique `Trace-ID`.
    - The third part is the `Span-ID` of the caller.
    - The fourth part is the trace flags (e.g., is this trace being sampled/recorded?).
- **Context Extraction**: Service B receives the header, extracts the `Trace-ID`, and uses the incoming `Span-ID` as the *Parent* of its own newly generated Span.
- **Trade-offs**: High computational and network overhead. You must implement *Tail-based Sampling* (e.g., the Collector only forwards 1% of successful traces to storage, but 100% of traces containing an error or high latency).

---

## Cloud Architecture

### 44. What is the difference between ALB, NLB, and API Gateway in AWS?
**Context**: A senior engineer must choose the correct entry point based on strict protocol, performance, and scaling requirements.

**Deep Dive**:
- **ALB (Application Load Balancer / Layer 7)**:
    - *When to use*: General-purpose web traffic (HTTP/HTTPS/gRPC).
    - *Capabilities*: Evaluates URL paths, Headers, and Cookies to route traffic (e.g., `/api/v1/*` goes to Target Group A). Native integration with WAF, OIDC/Cognito authentication.
    - *Cons*: Higher latency than L4; IP addresses change (dynamic scaling).
- **NLB (Network Load Balancer / Layer 4)**:
    - *When to use*: Extreme ultra-high performance, low latency, UDP traffic (e.g., gaming servers, real-time video), or when you need a *Static IP*.
    - *Capabilities*: Bypasses HTTP inspection completely. Just forwards TCP/UDP packets. Can handle millions of connections per second almost instantly.
- **API Gateway**:
    - *When to use*: Serverless architectures (Lambda), REST API management.
    - *Capabilities*: Deep integration with IAM, built-in rate limiting, payload validation (JSON schemas), protocol transformation (e.g., mapping a REST HTTP call into a direct DynamoDB SQL query without a backend server).
    - *Cons*: Extremely expensive at massive scale compared to ALB.

### 45. How do you implement Cell-based architecture for maximum isolation in the cloud?
**Context**: Standard Multi-AZ architecture protects against hardware failures but fails during global software bugs or "poison pill" deployments. E.g., a bad configuration pushes to the entire `us-east-1` region and takes down 100% of customers.

**Deep Dive**:
- **The Concept**: Coined by AWS, a Cell is an entirely isolated, fully functional stack of your services, databases, and network. Instead of one massive auto-scaling group serving 1,000,000 customers, you build 10 "Cells", each restricted to 100,000 customers.
- **Cell Routing Layer**: A specialized lightweight router maps a `customer_id` / `tenant_id` to a specific Cell.
- **Blast Radius Mitigation**: If a developer pushes a memory-leak bug, it is deployed to Cell 1 first. Cell 1 crashes, affecting 10% of customers. The deployment halts. Cells 2-10 are completely untouched and unaware of the failure.
- **Trade-offs**: Extreme operational overhead. Managing schema migrations, DNS routing, and global aggregate reporting across 10 disjointed databases is profoundly complex.

---

## Security Architecture

### 46. Explain the mitigation strategies for a complex Layer 7 DDoS attack versus a Layer 4 attack.
**Context**: DDoS attacks target different layers. Volumetric attacks try to overwhelm pipes; Application attacks try to overwhelm CPUs.

**Deep Dive**:
- **Layer 3/4 Attacks (Volumetric/Protocol)**: e.g., UDP Reflection, SYN Floods. The goal is to saturate the bandwidth of the data center.
    - *Mitigation*: These attacks must be stopped at the Edge, not at your servers. Use Cloudflare or AWS Shield. They use massive global bandwidth and BGP Anycast to absorb to traffic and specialized hardware to drop spoofed packets before they ever reach your VPC.
- **Layer 7 Attacks (Application Layer)**: e.g., HTTP Floods, Slowloris. The goal is to establish legitimate TCP connections and request heavy database operations (e.g., hitting the `/search?q=expensive_query` endpoint 50,000 times a second).
    - *Mitigation*: Layer 4 protections cannot see HTTP data securely (TLS termination). You must use a **WAF (Web Application Firewall)**.
    - Implement IP reputation filtering (block known Tor/Botnet IPs).
    - Configure aggressive Rate Limiting strictly on computationally expensive endpoints.
    - Implement intelligent Captchas (e.g., Turnstile) during high-load events to prove human interactivity.

### 47. How do you design a robust Key Management Strategy (Envelope Encryption)?
**Context**: Storing a 256-bit encryption key in plaintext on a server, or in environment variables, is incredibly vulnerable. If memory is dumped or the disk is stolen, all PII (Personally Identifiable Information) is compromised.

**Deep Dive**:
- **Envelope Encryption Concept**: You use a "Master Key" (KMS - Key Management Service) to encrypt your "Data Keys."
    1. The app requests a new Data Key from AWS KMS to encrypt a user's SSN.
    2. KMS returns two things: The *Plaintext Data Key* and an *Encrypted Data Key* (encrypted by the KMS Master Key, which never leaves AWS's secure hardware modules).
    3. The application uses the *Plaintext Data Key* in memory to quickly encrypt the SSN using AES-GCM.
    4. Crucially, the application instantly deletes the *Plaintext Data Key* from RAM. It then stores the *Encrypted Data Key* alongside the encrypted SSN in the database.
- **Decryption**: To read the SSN later, the app fetches the Database record. It sends the *Encrypted Data Key* to AWS KMS. KMS decrypts it (using IAM permissions to verify access) and returns the Plaintext Data Key to the app, which decrypts the SSN.
- **Trade-offs**: Highly secure, allows granular IAM access to decrypt specific records, and allows crypto-shredding (deleting the Master Key instantly makes all Data Keys unreadable, achieving GDPR compliance). However, it adds latency to read/write operations and incurs KMS API costs.

### 48. What is Zero Trust Architecture?
**Context**: The traditional "Castle and Moat" security model assumes anyone inside the corporate VPN is trusted. Once breached, attackers move laterally without restriction.

**Deep Dive**:
- **Core Principle**: "Never trust, always verify." Location (inside the VPC or VPN) implies absolutely zero inherent trust.
- **Micro-segmentation**: Firewalls exist between every single microservice. Service A cannot talk to Service B unless explicitly whitelisted in Security Groups or a Service Mesh policy.
- **Identity-Aware Proxy (IAP)**: All requests, even internal admin portals, must be authenticated via an Identity Provider (SSO with MFA), verifying both the user and the health/security posture of the device they are using.
- **Mutual TLS (mTLS)**: Every network call between internal microservices is encrypted using TLS certificates. Both the client and the server authenticate each other's certificates (preventing Man-in-the-Middle attacks even inside the private subnets).

### 49. Secure Authentication: OAuth 2.0 PKCE and Token Invalidation.
**Context**: Session cookies are going away. JWTs have immense vulnerabilities if not implemented specifically for Single Page Applications (SPAs).

**Deep Dive**:
- **OAuth 2.0 PKCE for SPAs**: The Implicit Flow is deprecated due to token interception vulnerabilities in the browser. You must use the "Authorization Code Flow with Proof Key for Code Exchange (PKCE)". The SPA dynamically generates a cryptographically random string (Code Verifier) and hashes it (Code Challenge) to prove to the Authorization Server that the app requesting the token is the exact same app that initiated the login.
- **Token Invalidation Challenges**: A standard JWT is stateless. A server cannot realistically "invalidate" it until its expiration time.
    - *Strategy 1 (Short-lived tokens)*: JWT access tokens expire in 5 minutes. Use long-lived, opaque "Refresh Tokens" stored in `HttpOnly, Secure` cookies. To revoke access, delete the Refresh Token in the database.
    - *Strategy 2 (Denylist)*: For extreme security situations, maintain a fast Redis cache of "revoked" JWT IDs (`jti` claim). Every API request must check if the token exists in the Redis Denylist before proceeding.

### 50. Secret Management at Enterprise Scale.
**Context**: `dotenv` files in production or hardcoded API keys in Git are massive compliance failures.

**Deep Dive**:
- **Dynamic Secrets**: Instead of giving an application a static Database Password that lives forever, use tools like HashiCorp Vault. Vault dynamically generates a unique, temporary database username/password that is valid for exactly 1 hour. If the app is compromised, the secret rotates and dies automatically.
- **Workload Identity Federation**: Instead of a CI/CD pipeline (like GitHub Actions) storing long-lived AWS IAM Access Keys to deploy code, configure an OIDC trust relationship. GitHub assumes a short-lived IAM role dynamically during the build, meaning there are zero static credentials to steal.
