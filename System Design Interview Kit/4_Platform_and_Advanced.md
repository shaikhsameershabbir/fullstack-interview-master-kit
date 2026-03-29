# Module 4: Platform Engineering & Real Company Architectures

This module covers the advanced, massive-scale systems that power the internet's largest companies. These questions test your ability to combine all previous concepts (Databases, Caching, CDNs, Streaming, and Gateways) into cohesive, global platforms.

---

## Real Company Interview Problems

### 25. Design Netflix's Streaming Architecture
**Requirements**: Stream video globally to 200M+ users with near-zero buffering, zero downtime, and highly personalized dashboards.
**Architecture**:
- **Control Plane (AWS)**: The "brain" of Netflix runs entirely on AWS (EC2, Cassandra, EVCache). 
    - This handles everything *except* the actual video bytes: Login, Billing, Recommendations, and DRM (Digital Rights Management).
    - Uses Cassandra heavily for cross-region active-active replication so a user's viewing history is perfectly synced whether they log in from California or Tokyo.
- **Data Plane / Open Connect (Custom CDN)**: Netflix DOES NOT use AWS to serve video. They built Open Connect Appliances (OCAs) — custom hardware filled entirely with high-speed SSDs arrayed in petabytes.
    - These OCAs are physically installed directly inside the server racks of major global ISPs (Verizon, Comcast, Vodafone). 
    - *How it works*: When you click "Play", the AWS Control Plane determines your location and ISP. It returns the IP address of the OCA installed *inside your specific neighborhood's ISP*. You stream the video locally, meaning the video traffic almost never traverses the open internet.
- **Predictive Caching**: Not all OCAs hold the entire Netflix library. The AWS analytics engine predicts that a new Marvel show will be highly popular in New York. During off-peak hours (3 AM), AWS preemptively pushes the 4K video files to the New York OCAs, so they are ready for the 8 PM surge.

### 26. Design Uber's Driver Matching System
**Requirements**: Millions of drivers and riders moving in real-time. Calculate ETAs and match the closest driver under 5 seconds.
**Architecture**:
- **Geospatial Indexing**: You cannot query a traditional SQL database `WHERE lat > X AND long < Y` for millions of moving cars every second. It requires specialized indexing.
    - *The Solution (H3 / S2)*: Uber uses H3 (an open-source hexagonal hierarchical spatial index). The entire globe is divided into hexagons. Every driver's GPS coordinate is mapped to an H3 Hexagon ID.
- **The Dispatcher Engine**:
    - *Driver Location Updates*: Drivers' phones emit GPS pings via WebSockets every 4 seconds. These hit a massive Kafka stream and update a fast, in-memory datastore (e.g., Redis Geospatial indexes or custom Ringpop clusters).
    - *The Match*: When a rider requests a car, the system calculates the rider's Hexagon ID. It then queries the datastore for all active drivers currently inside that hexagon (and the 6 immediately adjacent hexagons). 
- **ETA Calculation**: Straight-line distance is useless (rivers/highways). Once the 10 closest drivers are found via H3, their locations are sent to a massive offline Machine Learning model that calculates ETAs based on historic traffic data for those specific street segments at that specific time of day.

### 27. Design Google Docs' Collaborative System
*(Note: Expanded from Module 1's brief mention).*
**Requirements**: 50 people editing the exact same text document simultaneously without locking the document or overwriting each other's words.
**Architecture**:
- **The Core Conflict**: Bob deletes a word. Alice bolds the same word at the exact same time. If the server locks the document, the UI feels laggy.
- **Operational Transformation (OT) vs CRDTs**:
    - Google Docs famously relies on OT. Every keystroke is treated as an operation (`Insert "A" at index 5`). 
    - *The Math*: When Alice and Bob type concurrently, the central server receives both operations. The server applies a transformation matrix. If Bob inserted a character *before* Alice's edit, the server shifts Alice's index from `5` to `6` before broadcasting the final state to all clients.
- **Client-Side Buffering**: To ensure the user types instantly (zero latency), the local JavaScript applies the operation instantly to the local DOM, adds the operation to a "Pending" queue, and sends it to the server. If the server rejects or transforms it, the local client rolls back the DOM and reapplies the transformed state in milliseconds (unnoticeable to the human eye).

### 28. Design Zoom's Video Conferencing Architecture
**Requirements**: 100 people in a single HD video call without melting the clients' CPUs or network bands.
**Architecture**:
- **The Naive Approach (Mesh)**: Every person sends their video stream to every other person. 100 people * 99 streams = 9,900 concurrent connections per client. Mobile phones instantly crash.
- **The MCU Approach (Multipoint Control Unit)**: Everyone sends 1 stream to a central server. The server stitches all 100 faces into one giant 1080p video file and streams it out. 
    - *Cons*: Immense CPU cost for the server; massive latency; users cannot customize their layouts.
- **The Zoom Approach (SFU - Selective Forwarding Unit)**: 
    - Everyone sends 1 stream to the server. The server does *zero* video processing. It acts simply as a highly optimized UDP packet router. 
    - *Client customization*: The server forwards the 5 "active speaker" streams in 720p, and the remaining 95 streams as tiny 144p thumbnails. The client CPU decodes them.
- **Simulcast**: A user doesn't just send one 1080p stream. Their laptop encodes the video simultaneously in 1080p, 720p, and 360p and sends all three to the SFU server. The server checks the internet speed of the *receivers*. It forwards the 1080p stream to users on Fiber internet, and the 360p stream to users on 3G mobile networks.

### 29. Design Slack's Messaging Architecture
**Requirements**: Millions of active users. Workspaces with 50,000 members. Highly read-heavy but bursts of extreme write-heavy traffic.
**Architecture**:
- **The Edge (WebSockets)**: Like other chat apps, users maintain an open WebSocket connection to a "Message Router" server.
- **The Database Paradigm (Vitess)**: Slack cannot use standard MySQL. They migrated to **Vitess** (a database clustering system horizontally scaling MySQL).
    - *Data Locality*: A fundamental rule in Slack's architecture is that all data for a specific "Workspace" (e.g., the IBM workspace) lives on the same physical database shard. If an IBM employee queries `#general`, the query hits exactly one shard, preventing massive cross-shard joins.
- **The Fan-out Challenge (The 50,000 member channel)**:
    - If a CEO messages `#announcements` with 50,000 people, the system cannot do a synchronous DB read for 50k users.
    - *Solution*: A specialized Pub/Sub channel. The channel itself is a topic. The active WebSocket servers subscribe to this topic. The message hits the DB once, then hits the Pub/Sub bus. The 50 WebSocket servers holding those 50k connections receive the event and push the payload in parallel.

---

## Global Caching & Multi-Region Strategies

### 30. Design a Global CDN System
**Requirements**: Serve static assets globally with sub-20ms latency.
**Architecture**:
- **Edge Locations (PoPs)**: Deploy thousands of tiny server clusters globally in ISP data centers.
- **Anycast Routing**: Use the BGP Anycast protocol. All edge locations share the exact same IP address (e.g., `8.8.8.8`). The core internet routers evaluate the BGP tables and automatically route User A to the physically closest server announcing that IP address.
- **Tiered Cache Architecture**:
    - If a file is missing at the Edge, fetching it from the Origin Server in Virginia takes 200ms.
    - *Solution*: A Regional Edge Cache. If the Paris Edge node misses a file, it checks the massive Frankfurt Regional node. Only if both miss does it cross the Atlantic to the Origin Server. This reduces the load on the Origin Server by orders of magnitude. 
- **Invalidation**: Invalidating a file globally is historically slow. Systems like Fastly solved this by dropping the TTL concept entirely. When the origin updates a file, it publishes an HTTP PURGE request to an extremely fast RabbitMQ/Kafka control plane that executes the invalidation across all global edge servers in under 150 milliseconds.
