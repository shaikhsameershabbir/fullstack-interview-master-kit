# Module 2: Infrastructure & Search Systems

This module focuses on the heavy-lifting backbone of large-scale platforms. Interviewers here are testing your knowledge of intermediate layers (Gateways, Caches, Analytics) that sit between the raw compute and the data layer, as well as the specialized architecture required for Search and Discovery.

---

## Backend Infrastructure Systems

### 11. Design an API Gateway
**Requirements**: Single entry point for all clients, routing, rate limiting, authentication, logging, and protocol translation.
**Architecture**:
- **The Entry Point**: The Gateway sits behind a Layer 4 Load Balancer (NLB). It terminates SSL/TLS to offload decryption overhead from internal microservices.
- **Micro-Filters Pattern**: Architect the gateway using a chain of fast middleware filters (e.g., Spring Cloud Gateway or Envoy proxies).
    - *Filter 1 (Auth)*: Validates the JWT locally without hitting a database.
    - *Filter 2 (Rate Limiter)*: Checks a Redis cluster to decrement the user's Token Bucket.
    - *Filter 3 (Routing)*: Reads the URL path (`/api/v1/users`) and forwards traffic to the internal `User Service`.
- **Protocol Translation**: Mobile apps might send REST HTTP JSON. The Gateway translates this payload into highly compressed `gRPC` (Protobuf) before sending it to the internal network, saving massive bandwidth inside the VPC.
- **Trade-offs**: A Gateway is a systemic Single Point of Failure (SPOF) and a potential latency bottleneck. It must be highly available, over-provisioned, and functionally "dumb" (never place complex business logic in the gateway).

### 12. Design a Rate Limiting System Let's dive deeper than the basics.
**Requirements**: Limit users to 100 requests/minute to protect backend APIs. Track by IP or User ID.
**Architecture**:
- **Algorithms**:
    - *Token Bucket*: A bucket holds 100 tokens. A cron job adds 10 tokens per second. Requests cost 1 token. Good for allowing sudden "bursts" of traffic.
    - *Sliding Window Log*: Stores the exact timestamp of every request in Redis. Highly accurate but consumes immense memory at scale.
    - *Sliding Window Counter*: A hybrid approach. Tracks request counts per second in Redis and interpolates the rate over a rolling minute. Best balance of accuracy and memory.
- **Distributed State**: You have 50 API Gateways. Local rate limits (in memory) will allow abuses (a user could hit 100 on Gateway A, then 100 on Gateway B).
    - *Solution*: A centralized Redis cluster utilizing Lua scripts. Lua scripts execute atomically on Redis, preventing race conditions when two concurrent requests try to decrement the same bucket simultaneously.

### 13. Design a Distributed Logging & Monitoring System (like DataDog/ELK)
**Requirements**: Ingest logs from 10,000 servers, allow full-text search, graph error rates in real-time.
**Architecture**:
- **Ingestion Pipeline**: Servers cannot write directly to the logging database (it will crash under load).
    - *Log Forwarders*: Fluentd, Filebeat, or Logstash run as DaemonSets on the servers. They tail the `.log` files and push to a buffer.
    - *The Buffer*: Apache Kafka. This is the crucial shock-absorber. If Elasticsearch slows down during indexing, Kafka queues the terabytes of logs indefinitely until the DB catches up, preventing data loss.
- **Storage & Search Engine**: Elasticsearch (or AWS OpenSearch). It creates an "Inverted Index" of the text, allowing instantaneous searching of a specific `"NullPointerException"` across billions of logs.
- **Cold Storage**: Keeping 30 days of logs in Elasticsearch is incredibly expensive (requires massive RAM). Use a Lifecycle Policy: After 7 days, logs are automatically zipped and exported to AWS S3 (Cold Storage/Glacier) for compliance and deleted from active memory.

### 14. Design a Background Job / Task Scheduling System (Cron Service)
**Requirements**: Users schedule jobs "Run X every Tuesday at 9 AM" or "Run Y in 30 seconds." Billions of executions per day.
**Architecture**:
- **Storage**: A traditional database (PostgreSQL) is used to store the Job Definition (User ID, frequency, webhook URL).
- **The Engine (Time-Wheel or Priority Queue)**:
    - You cannot run `SELECT * FROM jobs WHERE execution_time = NOW()` across billions of rows every second.
    - *Redis ZSET (Sorted Set)*: Store the `Job_ID` with the Unix Timestamp as the "Score." A background worker polls the ZSET every second: `ZRANGEBYSCORE queue 0 <current_timestamp>`. If jobs exist, it pops them off the queue and sends them to workers.
- **Execution Workers**: A massive auto-scaling pool of workers (RabbitMQ/Celery) consumes the popped IDs, fetches the heavy payload from the DB, and executes the HTTP call.
- **Idempotency**: If a worker crashes midway, the job might be rerun. The downstream system being called *must* be idempotent, or the job system must track "Execution Locks" rigorously.

---

## Search & Discovery Systems

### 15. Design a Search Engine (like Google Search core crawler)
**Requirements**: Crawl the web, build an index, rank results.
**Architecture**:
- **1. The Crawler (Spider)**:
    - Starts with a list of seed URLs.
    - Downloads the HTML. A parser extracts all `<a href>` links and pushes them into an enormous **URL Frontier** queue (Redis or Kafka).
    - *Politeness*: DNS Caching is mandatory (DNS lookups are slow). The crawler must respect `robots.txt` and rate-limit itself so it doesn't accidentally run a DDoS attack on small websites.
- **2. The Indexer (MapReduce)**:
    - The downloaded HTML is sent to Hadoop/Spark clusters. They clean the text, remove stop words ("the", "and"), and run NLP (stemming: "running" -> "run").
    - Builds the *Inverted Index*: A massive hash map where the Key is the word (e.g., "React") and the Value is a list of Document IDs containing that word.
- **3. The Query Engine & Ranking**:
    - When a user types "React Tutorial", the system looks up both words in the Inverted Index, intersects the Document IDs, and sends the top 100 results to the Ranker.
    - PageRank (legacy) or Machine Learning Models calculate relevance based on keyword density, domain authority, and user click-through rates.

### 16. Design an Autocomplete Search System (Typeahead)
**Requirements**: User types "App", system suggests "Apple", "Application", "App Store" instantly. Must be sub-20ms.
**Architecture**:
- **Data Structure**: A **Trie** (Prefix Tree). Every node is a letter. The path `A -> p -> p` branches off to `-l-e` and `-l-i-c-a-t-i-o-n`.
- **Scaling the Trie**: A Trie of the entire English dictionary + popular phrases is too big for one server's RAM. You must shard the Trie (e.g., Server A holds prefixes a-m, Server B holds n-z).
- **Caching**: Put an aggressive Redis cache in front of the API. If 1,000 users type "App", the first one hits the Trie server, and the next 999 get the suggestions directly from Redis.
- **Analytics Pipeline**: How do we know "Apple" is more popular than "Application"? A Kafka stream tracks successful searches. A nightly cron job aggregates the counts and reconstructs the Trie with updated "weights" on the nodes so the most popular words bubble to the top.

### 17. Design a Recommendation Engine (like Netflix or Amazon)
**Requirements**: Personalize the homepage based on a user's past viewing/purchase history.
**Architecture**:
- **Data Collection (Event Stream)**: Every click, pause, add-to-cart, and scroll is pushed to Kafka -> Amazon S3 (Data Lake) as raw telemetry data.
- **Collaborative Filtering (Matrix Factorization)**:
    - The core algorithm comparing users to items. If User A likes Movies 1 and 2, and User B likes Movies 1, 2, and 3, the system recommends Movie 3 to User A.
    - Because creating a 100-Million-User by 1-Million-Movie matrix is computationally impossible in real-time, this is generated *offline* via nightly Spark/Hadoop batch jobs.
- **Near-Real-Time Updates**: If a user binges a horror series right now, you don't want to wait 24 hours to update their recommendations.
    - *Lambda Architecture*: The cold Batch Layer provides the baseline recommendations. A "Speed Layer" (Apache Flink) evaluates the user's clicks from the last 5 minutes and adjusts the ranking of the precomputed recommendations purely in cache, merging the two outputs before rendering the UI.
