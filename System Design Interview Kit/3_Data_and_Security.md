# Module 3: Data, Analytics & Security Systems

This module handles the architecture necessary to process massive volumes of telemetry data (Data Warehousing, Streaming Pipelines) and how to design the rigorous security perimeters (Authentication, DDoS, Secrets) required for top-tier E-commerce and SaaS platforms.

---

## E-Commerce & Event Systems

### 18. Design an E-commerce Platform (like Amazon core)
**Requirements**: Users browse millions of products, add to cart (never lose data), and checkout smoothly even during Black Friday spikes.
**Architecture**:
- **Product Catalog (Read-Heavy)**: 
    - *Storage*: Products have wildly varying attributes (a TV has resolution, a shirt has size). Document stores (MongoDB) or PostgreSQL with `JSONB` columns are ideal.
    - *Caching*: The catalog changes rarely but is read billions of times. An aggressive CDN and Redis caching layer must serve 99% of requests.
- **Shopping Cart (Write-Heavy/Resilient)**:
    - *The Rule*: You can never, ever lose a cart item. Even if the database cluster goes offline, the user adding an item must succeed.
    - *Storage*: DynamoDB or Cassandra. Carts are isolated by `User_ID` and written to heavily. 
    - *Conflict Resolution*: If a user adds an item on their phone, loses connection, and adds a different item on their laptop, use **Vector Clocks / CRDTs** to merge both carts seamlessly when they reconnect, rather than overwriting one with the other.

### 19. Design an Order Management & Payment System
**Requirements**: Process a high-frequency financial transaction. Guarantee exactly-once processing (Idempotency).
**Architecture**:
- **The Saga Pattern**: A "Checkout" involves deducting inventory, validating a coupon, and charging a credit card. If the credit card fails, you *must* restock the inventory.
    - Use an orchestrator (AWS Step Functions). It executes the inventory deduction API. If successful, it calls the Payment API. If the payment API returns `HTTP 402 Payment Required`, the orchestrator automatically triggers the inventory "Compensating Action" API to restock the item.
- **Database Architecture**: Financial data is the explicit domain of RDBMS (PostgreSQL/MySQL) due to strict ACID (Atomicity, Consistency, Isolation, Durability) guarantees. Never use eventual consistency for a ledger.
- **Idempotency**: See earlier scaling chapters. A unique `Idempotency-Key` (UUID) prevents double-charging during network retries.

### 20. Design an Inventory Management System
**Requirements**: Users across the globe competing to buy the last 5 limited-edition PS5s. No overselling allowed.
**Architecture**:
- **Database Locks**:
    - *Pessimistic Locking*: `SELECT * FROM inventory WHERE item_id = 123 FOR UPDATE`. This locks the row. The first user gets to process the checkout while everyone else waits in a queue. Completely accurate, but extremely slow at scale.
    - *Optimistic Locking*: Add a `version` column. When a user tries to checkout, the system runs: `UPDATE inventory SET stock = stock - 1, version = version + 1 WHERE item_id = 123 AND version = X`. If another user already bought it, `X` changed, and the update fails (returning a friendly "Sold Out" error to the second user). Highly performant, requires zero database locking.

---

## Large Scale Data & Analytics Systems

### 21. Design a Metrics & Log Collection Platform (like Datadog/Splunk)
**Requirements**: Agents collect data from 100,000 servers, query massive datasets in seconds.
**Architecture**:
- **Ingestion Pipeline (The Shock Absorber)**: 
    - 100,000 servers emitting 100 logs per second = 10 million logs/sec. No database accepts that natively.
    - *Kafka*: Agents push to distributed Kafka topics. Kafka stores these massive byte streams on disk sequentially, acting as a massive, indestructible buffer.
- **Stream Processing (Real-Time)**:
    - Apache Flink or Spark Streaming consumes from Kafka. They perform windowing (e.g., "Count `HTTP 500s` grouped by `Service Name` in a 1-minute tumbling window"). If the count hits a threshold, trigger an external HTTP alert to PagerDuty.
- **Time-Series Database (TSDB)**:
    - Flink writes the aggregated, compressed, 1-minute rollup data directly into a specialized database like InfluxDB or Prometheus (optimized for `(Timestamp, Metric Name, Value)`) to power Grafana dashboards.

### 22. Design a Data Warehouse ETL Architecture
**Requirements**: Extract data from 50 different microservice databases, transform it, and load it for Business Intelligence (BI) analysts to run massive `JOIN`s.
**Architecture**:
- **Data Lake (Raw Storage)**: Extract raw rows from PostgreSQL/MongoDB via Change Data Capture (Debezium) and dump the raw JSON/Parquet files continuously into Amazon S3 (Data Lake). 
- **The "T" in ETL (Transformation)**: 
    - Nightly Batch Jobs (Apache Spark or AWS Glue) scan S3. They clean data, mask PII (hashing SSNs), and denormalize highly relational data into massive, flat "Fact" and "Dimension" tables (Star Schema).
- **Data Warehouse (Fast Reads)**: 
    - Load the transformed, flat data into a Columnar Database (Snowflake or Amazon Redshift). Columnar databases are physically optimized to aggregate millions of numbers in milliseconds, perfect for a query like "Sum total revenue for 2024 grouped by state."

---

## Security & Authentication Architectures

### 23. Design an Authentication & SSO System
**Requirements**: Centralized identity provider (IdP) so users only log in once to access dozens of internal applications.
**Architecture**:
- **Protocols (SAML vs OIDC)**:
    - *SAML 2.0*: Legacy, XML-heavy protocol used primarily for enterprise corporate integration (Active Directory).
    - *OpenID Connect (OIDC)*: Modern, built on top of OAuth 2.0. Uses lightweight JWTs.
- **The SSO Flow**:
    1. User visits App A. App A redirects user to the central SSO portal (e.g., Okta).
    2. User enters credentials. Okta sets a long-lived session cookie for the `okta.com` domain.
    3. Okta redirects the user back to App A with a Short-Lived Authorization Code.
    4. App A's backend securely exchanges the code for an Access Token (JWT) and an ID Token (who the user is).
    5. When the user visits App B later, it redirects to Okta. Okta reads the existing session cookie, realizes the user is already authenticated remotely, and instantly skips the login screen, dropping the user directly into App B.

### 24. Design an API Security Gateway & DDOS Protection
**Requirements**: A global architecture capable of absorbing a massive botnet attack trying to bring down the origin servers.
**Architecture**:
- **Layer 3/4 Protection (Volumetric)**:
    - *The Issue*: Attackers blast 500Gbps of junk UDP traffic to fill your network pipes.
    - *The Solution*: Use a CDN with Anycast IP routing (Cloudflare/AWS Shield). Anycast ensures a botnet in Russia is absorbed by the CDN's PoP in Russia, while a botnet in Brazil hits the CDN's hardware in Brazil. The massive distributed bandwidth of the CDN easily swallows the 500Gbps before it ever reaches your private VPC.
- **Layer 7 Protection (Application WAF)**:
    - *The Issue*: Attackers open valid TLS connections and send legitimate-looking HTTP requests, but thousands of times a second (e.g., trying to brute-force a login).
    - *The Solution*: A Web Application Firewall sits at the Edge. It uses ML-driven behavioral heuristics to identify non-human traffic patterns and triggers an aggressive CAPTCHA challenge or hard-drops the connection based on IP reputation, keeping the load entirely off your application servers.
