# Module 1: Architecture, Systems Thinking & Platform Engineering

This module covers high-level architectural patterns, domain-driven design, and the emerging field of platform engineering. For a senior level (8-10 years), the expectation is not just knowing "what" but "why," "when," and "what are the trade-offs." Answers here are structured to highlight real-world applicability and deep technical reasoning.

---

## Architecture & System Thinking

### 1. How do you design a scalable backend architecture?
**Context**: Designing for scale requires a multi-layered approach that addresses bottlenecks at the application, data, and infrastructure levels before they become critical failures.

**Deep Dive**: 
- **Application Layer Decomposition**: Move from a monolithic structure to modular components or microservices based on business domains (DDD) to allow independent scaling of high-load services.
- **Statelessness**: Application servers must be strictly stateless. Any state (sessions, temporary processing data) must be moved to external stores like Redis.
- **Asynchronous Processing**: Decouple synchronous requests from long-running tasks using Message Queues (RabbitMQ, Kafka). The web server should acknowledge the request (`HTTP 202 Accepted`) immediately and process it in the background.
- **Database Partitioning & Caching**: Implement multi-level caching (CDN -> Edge Cache -> Redis -> Database). As data grows, utilize read-replicas for read-heavy loads, and eventual database sharding for write-heavy loads.
- **API Gateway Pattern**: Centralize cross-cutting concerns (authentication, rate limiting, and request routing) at the gateway layer to simplify downstream microservices.

**Trade-offs**: Extreme decomposition increases operational complexity and network latency. Caching introduces "stale data" anomalies and complex cache invalidation strategies. Asynchronous processing requires robust Dead Letter Queue (DLQ) architectures.

### 2. Monolith vs microservices — when should you choose each?
**Context**: This is the classic architectural debate. The choice dictates the organizational structure and operational overhead for years.

**Deep Dive**: 
- **Monolith**: A single deployable unit connecting to a single database.
    - *When to use*: MVP stages, small teams (<10 engineers), or systems with low to medium domain complexity.
    - *Pros*: Simple CI/CD, simple end-to-end testing, zero network latency between components, transactional integrity (ACID) is trivial.
    - *Cons*: "Spaghetti code" arises quickly without rigid discipline; deployment bottlenecks (one failing test blocks the entire release); vertical scaling limits.
- **Microservices**: A suite of independently deployable, small, modular services, each running its own process and communicating via lightweight mechanisms.
    - *When to use*: High organizational scale (Conway's Law applies here—multiple autonomous teams), systems requiring polyglot persistence, or extreme disparate scaling needs (e.g., a massive recommendation engine vs a low-traffic admin panel).
    - *Pros*: Independent deployments, fault isolation (one service crashing doesn't down the app), and technology diversity.
    - *Cons*: "Distributed monoliths" occur if coupled too tightly. Eventual consistency challenges replace ACID transactions. Observability and debugging become exponentially harder.

### 3. What is the Strangler Fig pattern in detail, and how do you handle the database migration part?
**Context**: Migrating from a monolith to microservices is highly risky. A "big bang" rewrite almost always fails. The Strangler Fig pattern mitigates this risk by gradually replacing specific pieces of functionality.

**Deep Dive**:
1. **Identify the Bounded Context**: Pick a low-risk, highly independent domain (like "User Notifications").
2. **API Gateway Interception**: Introduce a reverse proxy or API Gateway in front of the monolith. It routes `/notifications` to the new microservice, and everything else `/*` to the old monolith.
3. **Database Migration (The Hard Part)**: 
    - *Phase 1 (Shared Database)*: The new microservice reads/writes directly to the monolith's database tables. It proves the code works but leaves data coupled.
    - *Phase 2 (Data Synchronization)*: The microservice gets its own database. Use Change Data Capture (CDC) via tools like Debezium to replicate data from the monolith DB to the new DB in real-time. Both databases are active.
    - *Phase 3 (Cutover)*: Once data is fully synced and verified, update the monolith's code to stop writing to those tables and instead call the new microservice's API.
    - *Phase 4 (Decommission)*: Drop the tables from the monolith DB.

**Real-world applicability**: This pattern is the industry standard for safe migrations at massive enterprises. It ensures that if the new service fails, routing can be instantly flipped back to the monolith.

### 4. What is domain-driven design (DDD)?
**Context**: DDD is a software engineering approach that centers on programming a domain model that has a deep understanding of the processes and rules of a domain.

**Deep Dive**:
- **Strategic Design**: The high-level map. It involves defining *Bounded Contexts* (explicit boundaries where a specific model applies, such as "Billing" vs "Shipping"), *Context Maps* (how these boundaries communicate), and the *Ubiquitous Language* (shared terminology between developers and domain experts).
- **Tactical Design**: The code-level patterns. It utilizes *Entities* (objects with distinct identity, like User), *Value Objects* (immutable objects determined by their attributes, like Money or Address), *Aggregates* (clusters of domain objects treated as a single unit, managed by an Aggregate Root), and *Repositories* (interfaces for data access).

**Trade-offs**: DDD requires significant upfront design and continuous collaboration with domain experts (who are often busy). It is overkill for simple CRUD applications.

### 5. What is bounded context in DDD?
**Context**: Misunderstanding bounded contexts leads to massive, rigid data models that try to satisfy every requirement across the business.

**Deep Dive**: A bounded context is a linguistic boundary. Within it, a domain model is strictly defined and uniquely applicable. 
- *Example*: In an E-Commerce system, the term "Product" means something entirely different depending on the context. In the *Catalog Context*, "Product" has attributes like `Image_URL`, `SEO_Tags`, and `Description`. However, in the *Inventory Context*, "Product" only cares about `SKU`, `Warehouse_Location`, and `Stock_Count`. 
- By splitting these into distinct models rather than one giant `Product` table, microservices can be isolated, and teams can evolve their schemas independently.

### 6. How do you design a modular architecture?
**Context**: Modular architecture (often called a "Modular Monolith") is gaining immense popularity as a reaction against the complexity of microservices.

**Deep Dive**:
- **Strict Boundaries**: Modules communicate strictly through well-defined, public API interfaces/facades, not by reaching into another module's database or internal classes.
- **Internal Visibility**: Classes inside a module should have package-private or internal visibility to prevent external coupling.
- **Dependency Management**: Cross-module dependencies are managed via an event bus or direct interface calls. Circular dependencies are strictly forbidden and tested against in CI/CD (using tools like ArchUnit or Nx workspace guards).
- **Separation of Concerns**: Each module represents a distinct Bounded Context and is theoretically capable of being extracted into a microservice in the future if scaling needs demand it.

**Trade-offs**: Requires extreme discipline from the development team. Without strict architectural enforcement at the compile/test stage, a modular monolith quickly devolves into a regular, tightly coupled monolith.

### 7. How do you manage technical debt in large systems?
**Context**: Technical debt is inevitable. The goal of a senior engineer is debt management and prioritization, not naive eradication.

**Deep Dive**:
- **Visibility & Tracking**: Tech debt must be tracked alongside product work. Create engineering tasks, tag them as tech debt in Jira, and assign a "cost of delay" (e.g., "this old library adds 5 minutes to every build").
- **Budgeting (The 20% Rule)**: Negotiate with Product Management to allocate 15-20% of every sprint/cycle purely to refactoring and debt reduction.
- **Pragmatic Refactoring (Boy Scout Rule)**: Refactor code incrementally whenever you touch it for a feature. "Leave the campground cleaner than you found it."
- **Risk Assessment**: Not all debt is bad. If a terrible, unreadable piece of code works perfectly and hasn't needed modification in 5 years, leave it alone. Prioritize debt that severely hinders developer velocity or causes production outages.

### 8. What is Clean Architecture?
**Context**: Coined by Uncle Bob Martin, Clean Architecture aims to separate the core business rules from infrastructure and UI dependencies.

**Deep Dive**:
- **Layers**: 
    1. *Entities (Core)*: Enterprise-wide business rules.
    2. *Use Cases (Application)*: Application-specific business rules.
    3. *Interface Adapters*: Controllers, Presenters, and Gateways (converting data between use cases and external agencies).
    4. *Frameworks & Drivers*: The Database, Web Framework, external APIs.
- **The Dependency Rule**: Source code dependencies must *only point inward*. The core Entities and Use Cases must *never* import a SQL library or an HTTP framework. 
- **Inversion of Control**: To save data to a database without knowing the database layer, the Use Case defines an interface (e.g., `UserRepository`), and the outer infrastructure layer implements it.

**Trade-offs**: High initial scaffolding overhead. The excessive boilerplate (mapping database entities to domain entities to DTOs) can frustrate developers on simple tasks.

### 9. What is Hexagonal Architecture (Ports and Adapters)?
**Context**: Hexagonal Architecture is conceptually identical to Clean Architecture but utilizes different nomenclature. It focuses on symmetrical communication with the outside world.

**Deep Dive**:
- **The Hexagon (Core)**: The pure business logic resides inside the hexagon. It is entirely agnostic to the outside world.
- **Ports**: Interfaces that define how the application communicates.
    - *Primary/Driving Ports*: Interfaces exposed to the outside (e.g., `createUserUseCase`). Driven by Web Controllers or CLI tools.
    - *Secondary/Driven Ports*: Interfaces the application needs to get information (e.g., `UserRepository`).
- **Adapters**: Concrete implementations of the ports. 
    - A REST Controller is an adapter mapping HTTP requests to a Driving Port.
    - A PostgreSQL Repository is an adapter implementing a Driven Port.
- **When to Use**: Highly complex business domains where testing business rules independently of the database or network is critical. Extremely useful if you anticipate swapping out databases or moving from REST to gRPC.

### 10. How do you tackle a system rewrite vs. gradual refactoring (The Ship of Theseus problem)?
**Context**: A system rewrite is often seen as a panacea for tech debt, but it carries a massive failure rate inside the industry ("The Second System Effect").

**Deep Dive**:
- **The Rewrite Trap**: A full rewrite halts new feature development on the current system for months or years. By the time the new system is ready, the business requirements have changed.
- **Gradual Refactoring (Ship of Theseus)**: Replace parts of the system one by one while the system continues to operate. Over time, the entire system is modernized without downtime or feature freezes.
- *How to execute*: 
    1. Build rigorous automated test suites around the legacy code.
    2. Use the **Branch by Abstraction** pattern: isolate the legacy component behind an interface.
    3. Build the new implementation.
    4. Use a configuration toggle (Feature Flag) to switch between the old and new implementation, optionally running both in parallel and comparing results ("Shadowing").
    5. Retire the old implementation.

---

## Event Sourcing & CQRS

### 11. How do you design an Event-Sourcing architecture, and when is it appropriate?
**Context**: Traditional databases store current state (CRUD). Event Sourcing stores every change to the state as an immutable sequence of events.

**Deep Dive**:
- **Implementation**: Instead of updating an `AccountBalance` from $100 to $50, you append an `AccountDebited` event of $50 to an "Event Store" (like EventStoreDB or an append-only Kafka topic). The current balance is derived by replaying all events from the beginning of time.
- **Snapshots**: Because replaying thousands of events is slow, the system periodically saves a "snapshot" of the current state. Next time, it loads the snapshot and only replays the handful of events that occurred *after* it.
- **Trade-offs**: Provides an exact, unalterable audit trail (perfect for finance, healthcare, or complex game states) and allows "time travel" to reconstruct historical states. However, it requires a massive mental shift, makes data migrations/GDPR deletions (Right to be Forgotten) incredibly difficult, and introduces eventual consistency delays.

### 12. What is the CQRS (Command Query Responsibility Segregation) pattern, and how does it relate to Event Sourcing?
**Context**: In systems with high read/write disparity, using the same data model for both leads to performance and scaling bottlenecks.

**Deep Dive**:
- **Mechanics**: CQRS splits the application into two distinct sides:
    - **Commands**: Methods that change state (Create, Update, Delete). These execute business logic, validate constraints, and write to a specialized "Write Database."
    - **Queries**: Methods that read state. These bypass complex domain logic and read from a specialized "Read Database" (like Elasticsearch or Redis) optimized for specific UI views.
- **The Event Sourcing Link**: CQRS is almost mandatory with Event Sourcing. The "Command" side writes the Event. An Event Handler listens to that event and updates a materialized view in the "Read Database" (the Query side).
- **Trade-offs**: Introduces significant architectural complexity and forces the UI to handle Eventual Consistency (the Read model might be slightly behind the Write model when the user hits "refresh").

---

## SaaS Architecture

### 13. How do you design for multi-tenancy in a SaaS platform architecture?
**Context**: SaaS platforms serve multiple clients ("tenants"). Deciding how to separate their data impacts security, scaling, and cost.

**Deep Dive**:
1. **Silo Model (Database-per-Tenant)**: Every tenant gets their own isolated database instance. 
    - *Pros*: Ultimate security and compliance (HIPAA/FedRAMP), easy to restore backups for a single tenant, "noisy neighbor" isolation.
    - *Cons*: Extremely expensive and operationally complex. Managing schema migrations across 10,000 separate databases is a nightmare.
2. **Pool Model (Shared Database, Shared Schema)**: All tenants share the same database tables. Every row requires a `tenant_id` column.
    - *Pros*: Highly cost-effective, easy to deploy schema changes.
    - *Cons*: High risk of data leakage (bugs where `tenant_id` is forgotten in a query), noisy neighbors can degrade performance for everyone.
3. **Bridge Model (Schema-per-Tenant)**: Tenants share the same database server, but each gets their own logical schema (e.g., in PostgreSQL).
    - *Pros*: Balances security (isolated tables) with cost (shared infrastructure).
- **Implementation Choice**: Often a hybrid approach is best. Provide the Pool model for standard tiers, and offer Silo models for Enterprise tiers paying premium pricing.

---

## Platform Engineering

### 14. What is platform engineering and how does it differ from DevOps?
**Context**: DevOps was meant to combine Dev and Ops, but often failed, forcing developers to learn complex infrastructure tools (Kubernetes, Terraform) taking them away from product work.

**Deep Dive**: Platform Engineering treats the internal development team as their primary customer.
- **The Goal**: Build an Internal Developer Platform (IDP) that provides a paved road ("Golden Paths") for deploying software.
- **The Difference**: DevOps is a culture/practice. Platform engineering is the productization of that practice. A platform engineer builds the self-service web portal (like Backstage) so a product engineer can click "Create New Service" and automatically get a repo, CI/CD pipeline, staging environment, and metrics dashboard—all with zero YAML configuration required from the developer.

### 15. What is a monorepo vs polyrepo, and what are the trade-offs at enterprise scale?
**Context**: Source code organization significantly impacts developer velocity and tooling requirements.

**Deep Dive**:
- **Monorepo**: All code across the company in one Git repository (used by Google, Meta, Uber).
    - *Pros*: Unified versioning (no "Dependency Hell"), atomic commits across multiple services, trivial code sharing.
    - *Cons*: Standard Git breaks down at scale (requires virtual file systems). CI/CD must be highly intelligent (using tools like Nx or Bazel) to only test affected code, otherwise, pipelines take hours.
- **Polyrepo**: One repository per project/service.
    - *Pros*: Perfect boundary isolation, simple git checkouts, very fast localized CI/CD pipelines.
    - *Cons*: Upgrading a core logging library requires PRs across 100 repositories. "Version drift" is inevitable.

### 16. How do you design shared libraries in large projects?
**Context**: Shared libraries (`common-utils`) often become dumping grounds, resulting in brittle dependencies that slow everyone down.

**Deep Dive**:
- **Strict Single Responsibility**: Do not create a single `shared-core` package. Create hyper-focused packages: `@acme/logger`, `@acme/auth`, `@acme/http-client`.
- **Zero Transitive Dependencies**: A shared utility library should avoid importing massive external frameworks (like Lodash or Moment.js) unless absolutely necessary, as it forces all consumers to bundle those large dependencies.
- **Versioning Strategy**: Use strict Semantic Versioning. If changing a method signature, increment the major version. Never make a breaking change in a minor version bump, as automation will auto-upgrade consumers and break their builds.

### 17. How do you manage dependency conflicts in monorepos?
**Context**: In monorepos containing multiple applications, misaligned dependency versions cause severe runtime errors.

**Deep Dive**:
- **Single Version Policy**: Enforce an enterprise-wide rule that only one version of a third-party library can exist in the monorepo (e.g., only React 18 is allowed). This forces the entire company to upgrade simultaneously.
- **Peer Dependencies**: Use `peerDependencies` in shared internal packages. If `@acme/ui` relies on React, it shouldn't install React itself; it should declare it as a peer, ensuring it uses the exactly matched version from the consuming application.
- **Tooling Enforcement**: Use dependency checkers (like `syncpack` or native workspace configurations in Nx/Turborepo) to fail the CI build if version discrepancies are detected across `package.json` files.

---

## Developer Experience (DX) & Tooling

### 18. What is Developer Experience (DX), and how do you track it?
**Context**: DX directly correlates to developer retention and feature throughput. Poor DX leads to burnout and high turnover.

**Deep Dive**: DX is the total sum of friction (or lack thereof) a developer encounters while doing their job. 
- **Tracking Metrics**: Don't track "lines of code." Track:
    - *Time to First PR*: How long does it take a new hire to merge code to production?
    - *Pipeline Duration*: If CI takes 45 minutes, developers context-switch and lose focus. Aim for <10 minutes.
    - *Local Setup Time*: Can a developer spin up the entire stack locally with a single `docker-compose up` command, or does it require a 10-page wiki guide?
- **Feedback Loops**: Conduct anonymous internal surveys (e.g., quarterly Developer Satisfaction surveys) asking: "Does our tooling slow you down?"

### 19. How do you enforce architectural standards across autonomous teams?
**Context**: You have 10 teams building microservices. How do you ensure they don't all invent different ways to log, authenticate, and communicate?

**Deep Dive**:
- **Golden Image Templates**: Provide scaffolding tools (e.g., Yeoman, Backstage templates). The easiest way to enforce standard is to make the standard the easiest path to start.
- **Linting at Scale**: Use custom AST linters (Custom ESLint rules) to automatically flag antipatterns like direct database queries from controllers.
- **Architectural Decision Records (ADRs)**: Require teams to submit open-source style RFCs (Requests for Comments) for major technical changes. This builds consensus rather than using top-down mandates.
- **Automated Guardrails**: Use tools like ArchUnit in backend languages to fail CI builds if standard layering rules (Dependency Rule) are violated.

### 20. Designing for "Inner Sourcing" in large organizations.
**Context**: Teams often get blocked waiting for other teams to build features. Inner sourcing adopts open-source models internally to solve this.

**Deep Dive**:
- **The Concept**: Allow Team A to submit a Pull Request to Team B's microservice code repository to add the feature they desperately need.
- **Requirements**:
    - *Extensive Test Coverage*: Team B must trust that Team A won't break their service. 90%+ test coverage is required.
    - *CONTRIBUTING.md*: Clear guidelines on architecture, code style, and PR processes.
    - *Maintainer Roles*: Team B retains "Maintainer" status; they review and merge the PR but don't do the heavy lifting of writing it.
- **Real-world advantage**: Massively breaks down silos, accelerates time-to-market, and spreads knowledge across departments.
