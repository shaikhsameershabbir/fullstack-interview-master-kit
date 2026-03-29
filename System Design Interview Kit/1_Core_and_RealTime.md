# Module 1: Core System Design & Real-Time Systems

This module tackles the foundational system design interview questions. A senior system design interview is rarely about finding the "correct" answer; it's about navigating trade-offs, defending architectural choices, and demonstrating a deep understanding of scalability bottlenecks (network, memory, disk I/O, and CPU).

---

## Core System Design Problems

### 1. Design a URL shortener (like Bitly)
**Requirements**: 
- *Functional*: Given a long URL, return a short URL. Clicking short URL redirects to long URL.
- *Non-Functional*: Highly available, low latency for redirects, unguessable URLs, must scale to 100M new URLs/month and 1B redirects/month.

**Deep Dive & Architecture**:
- **Data Capacity**: 100M URLs * 5 years * ~500 bytes per record = ~3TB. A relational database with sharding or a NoSQL DB (like DynamoDB/Cassandra) easily handles this.
- **Algorithm (Base62 Conversion)**: An MD5 hash of the URL is 128-bit (too long for a short URL). Base62 (`[0-9a-zA-Z]`) allows 62^7 = 3.5 trillion unique 7-character strings.
    - *Approach 1 (Counter Engine)*: Use a distributed counter (e.g., Zookeeper or Redis) to generate a unique integer ID (e.g., 10,000,001). Convert that integer to Base62.
    - *Approach 2 (KGS - Key Generation Service)*: Pre-generate random 7-character Base62 strings and store them in a fast RDBMS. The application simply asks the KGS for the next free key. This guarantees zero collisions and avoids realtime generation latency.
- **The Redirect**: To redirect quickly, the Short URL is heavily cached globally via CDNs and Redis. Read-to-Write ratio is roughly 10:1.
    - *HTTP 301 vs 302*: A 301 (Permanent) redirect is cached by the browser heavily, reducing server load. A 302 (Temporary) redirect hits your server every time, which is terrible for load but essential if you need to track precise click analytics.
- **Trade-offs**: Storing 3TB in Redis is expensive. The database should be the source of truth, with only the most frequently accessed URLs kept in a Redis LRU (Least Recently Used) cache.

### 2. Design a pastebin service
**Requirements**: 
- *Functional*: Users paste text, get a URL to share it. View text via the URL. Optionally expire pastes.
- *Non-Functional*: High availability, text size up to 10MB, highly read-heavy.

**Deep Dive & Architecture**:
- **Storage Strategy**: Unlike URL shorteners, the payload is massive (10MB text). Storing this in an RDBMS like PostgreSQL will destroy the database's performance due to bloated pages.
    - *Metadata*: Store the short URL, user ID, creation date, and expiration timestamp in a fast DB (Cassandra or MySQL).
    - *Object Storage*: Store the actual 10MB text file in AWS S3 or a similar highly scalable blob storage system. The DB metadata row points to the S3 URI.
- **Caching Clause**: If a paste goes viral on Reddit, the S3 bill will skyrocket, and latency will suffer. Use AWS CloudFront (CDN) to serve the raw text object directly from Edge locations, bypassing application servers entirely.
- **Data Purging**: A background worker (chron job) sweeps the metadata database daily for expired timestamps, sending async requests to delete the corresponding objects from S3.

### 3. Design a file storage system (like Google Drive)
**Requirements**: 
- *Functional*: Upload/download files, sync across devices, share links.
- *Non-Functional*: Extreme durability (data loss is unacceptable), highly resilient.

**Deep Dive & Architecture**:
- **Chunking (Block Servers)**: Uploading a 5GB 4K video over HTTP regularly fails. The client must split the file into chunks (e.g., 4MB blocks). 
    - The client computes a hash for each chunk and uploads them concurrently. 
    - *Benefits*: Resumable uploads (if a chunk fails, retry only that chunk). Deduplication (if two users upload the exact same 4MB chunk of a movie, the system recognizes the hash and only stores it once, saving petabytes of storage).
- **Metadata Database**: A highly normalized relational database (or Spanner/CockroachDB for global consistency) tracking `user_id`, `file_id`, `block_hashes`, `version`.
- **Synchronization (Delta Sync)**: If a user modifies 1 line of a 10MB text file, the client doesn't upload 10MB; it only uploads the modified 4MB block. A notification service (WebSockets) tells other linked devices to pull *only* that new block to sync.

### 4. Design a video streaming platform (like YouTube)
**Requirements**: 
- *Functional*: Upload videos, watch videos without buffering.
- *Non-Functional*: Storage of petabytes of video data, ultra-low latency playback globally.

**Deep Dive & Architecture**:
- **The Upload Pipeline**: Uses a massive, asynchronous task queue (Kafka + Celery/Worker nodes).
    - When a user uploads a `.mp4`, it hits raw storage.
    - A transcode queue splits the video into different formats (1080p, 720p, 480p) and different codecs (H.264, VP9) suitable for iPhones, Androids, and Smart TVs.
    - *Watermarking & Thumbnails*: Generated synchronously during transcoding.
- **The Delivery System (DASH / HLS)**: Videos are not served as massive files. They are split into tiny 5-second chunks (e.g., `.ts` files).
    - *Adaptive Bitrate Streaming*: As the user's internet speed changes while watching on a train, the browser automatically requests the next 5-second chunk in a lower or higher resolution.
- **Global CDNs**: YouTube survives because of its Edge network. Videos are replicated to Points of Presence (PoPs) physically located inside major ISPs (Comcast, Verizon) globally. The core data centers rarely serve video traffic; they only serve the metadata (comments, recommended videos).

### 5. Design a music streaming service (like Spotify)
**Requirements**: 
- *Functional*: Play songs instantly, create playlists, offline mode.
- *Non-Functional*: 100M Daily Active Users (DAU), zero latency between track transitions.

**Deep Dive & Architecture**:
- **Storage Profile vs YouTube**: Music files are vastly smaller than video. A standard MP3/Ogg Vorbis file is ~5MB. The focus is less on raw bandwidth and more on *immediate responsiveness*.
- **Client-Side Caching**: Mobile apps aggressively cache playing songs and the user's favorite playlists into local phone storage.
- **Playback Architecture**: Does not necessarily need Complex DASH streaming like video. Fast fetching of full or chunked 5MB files from CDN edge nodes into an in-memory buffer is enough.
- **The Recommendation Engine Bottleneck**: Computing "Discover Weekly" for 100M users cannot happen in real-time. It requires massive offline MapReduce jobs (Hadoop/Spark) running against the data warehouse, outputting precomputed JSON playlists into a fast NoSQL database (Cassandra) weekly.

---

## Messaging & Real-Time Systems

### 6. Design a real-time chat system (like WhatsApp)
**Requirements**: 
- *Functional*: 1-on-1 chat, group chat, online status, read receipts.
- *Non-Functional*: 1 Billion users, minimum latency, messages delivered exactly once and in order.

**Deep Dive & Architecture**:
- **Connection Management (The C10M Problem)**: 1 billion users means 1 billion open TCP connections. Standard HTTP pooling dies.
    - *Solution*: A massive fleet of Chat Servers utilizing WebSockets or Long Polling. They maintain sticky state in memory using non-blocking I/O (Erlang, Go, Netty in Java).
- **Routing & Service Discovery**: When Alice connected to Server #1 messages Bob connected to Server #80, how do they connect?
    - Server #1 uses a key-value store (Redis) to look up: `Bob -> Server #80`.
    - Server #1 pushes the message via a message broker (RabbitMQ or a custom pub/sub backplane) to Server #80, which pushes the WebSocket message to Bob's phone.
- **Message Storage**: Do not use RDBMS. Chat platforms generate billions of tiny text rows daily. Cassandra or HBase is the standard because of their unmatched write-throughput and column-family design optimized for append-only time-series data (e.g., `Partition Key = Chat_ID`, `Clustering Key = Timestamp`).
- **Offline Delivery**: If Bob’s phone is dead, the push-backplane stores the message in a temporary queue. When Bob's phone connects and authenticates, it drains the queue.

### 7. Design a live notification system
**Requirements**: 
- *Functional*: Send notifications (Push, Email, SMS) system-wide, opt-in/opt-out preferences.
- *Non-Functional*: Deliver 10M notifications per minute without crashing downstream providers like Apple APNS or Firebase FCM.

**Deep Dive & Architecture**:
- **Fan-Out Architecture**: An event (e.g., "Trending Video Uploaded") triggers a single message to a Kafka topic. 
    - A consumer reads the event, looks up the millions of subscribed users from PostgreSQL, and fans out (multiplies) that into millions of individual notification jobs pushed to a high-throughput queue like RabbitMQ or Redis List.
- **Worker Queues by Priority & Type**: Do not mix a "Password Reset" email with a "Weekly Digest" push notification. The workers reading the queues must be categorized by priority so the fast lane is never blocked.
- **Deduplication & Rate Limiting (Crucial)**: Protect downstream APIs. If you send 10M pushes to Apple APNS in one second, Apple will blacklist your servers. Workers must utilize Token Bucket rate limiters before dispatching the HTTP call to external providers. Redis `SETNX` (Set if Not Exists) prevents the system from accidentally texting a user the same notification twice during worker retries.

### 8. Design a real-time collaborative editor (like Google Docs)
**Requirements**: 
- *Functional*: Multiple users edit the same document in real-time, cursor tracking, offline-sync.
- *Non-Functional*: Sub 50ms latency, strict conflict resolution.

**Deep Dive & Architecture**:
- **The Core Problem**: Concurrency. Alice deletes word 3; Bob inserts a word at position 3 at the exact same millisecond.
- **Conflict Resolution Algorithms**:
    - *OT (Operational Transformation)*: The legacy engine used by Google Wave and early Docs. Every keystroke is an "Operation". The central server intercepts Bob's and Alice's operations, transforms them against each other mathematically, and broadcasts the resolved, non-conflicting state. Extremely complex to mathematically prove.
    - *CRDTs (Conflict-free Replicated Data Types)*: The modern approach (Figma, Notion). Characters in the document don't have integers as indexes (1, 2, 3) because inserting between 1 and 2 breaks the array. Instead, they use fractional indexing (`0.5`) or unique identifiers. CRDTs guarantee that if Alice and Bob apply the same exact edits in *any* order asynchronously, their local documents will eventually look identical without a centralized server needing to "resolve" anything.
- **WebSocket Gateway**: Every client opens an active WebSocket connection. The backend acts as a highly optimized pub/sub relay mechanism for these tiny operational packets. 
    
### 9. Design a presence system (Online/Offline status)
**Requirements**: 
- *Functional*: See green dot when users are online, timestamp when they go offline.
- *Non-Functional*: Real-time updates for million-user rosters, highly read/write intensive.

**Deep Dive & Architecture**:
- **The Heartbeat Pattern**: Clients send a tiny ping every 5 seconds over their open WebSocket to say, "I am still alive."
- **In-Memory Tracking**: The Chat Gateway updates a Redis cache with a short TTL (Time to Live) of 10 seconds: `SET user:123:status "online" EX 10`. 
    - If the heartbeat stops (phone dies), the Redis key automatically expires and vanishes.
- **Presence Fan-out**: When Alice goes online, the system cannot afford to query the DB and push a message to all 5,000 of her contacts.
    - *Pull mechanism (Scaling compromise)*: The client app only fetches the online status of the 20 contacts currently visible on the screen.
    - *Push mechanism*: Use a Pub/Sub channel for small groups. If Alice is in a group chat of 5 people, her presence change is published to that group's specific channel.

### 10. Design a multiplayer game backend (e.g., a massive Battle Royale server)
**Requirements**: 
- *Functional*: Sync player movements, shoot events, global leaderboards.
- *Non-Functional*: 60 "ticks" (updates) per second, zero noticeable lag, highly anti-cheat resistant.

**Deep Dive & Architecture**:
- **Protocol**: HTTP and TCP are completely banned for gameplay. The overhead of TCP headers and packet retransmission (ensuring order) creates "lag spikes". UDP is mandatory. If a player movement packet drops, you don't care; another one is coming 16ms later anyway.
- **Authoritative Server**: To prevent hacking (aimbots/wallhacks), the client is physically untrusted. The client sends an *input* ("I pressed W"). The centralized C++ or Rust Game Server calculates the physics, checks if a wall was in the way, and updates the global game state, then blasts that state via UDP to all 99 other players.
- **Client-Side Prediction**: Because waiting 50ms for the server to confirm your movement feels sluggish, the client moves the character locally *immediately* and predicts the server's approval. If the server disagrees, the client's screen snaps back to reality ("rubber banding").
- **State Partitioning**: 100 players in a giant map cannot receive updates for every other player (N^2 problem). The physics server divides the map into a spatial grid (Quadtree). It only sends Player A the UDP packets for players residing in Player A's immediate grid quadrant.
