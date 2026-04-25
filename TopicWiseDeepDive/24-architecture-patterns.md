## 24) Architecture Patterns (MVC, Clean Architecture, microservices vs monolith, event-driven)

### Definition (technical)
**Architecture patterns** are reusable structural approaches for organizing code and services (layers, dependency direction, service boundaries, communication styles) to improve maintainability, testability, and scalability—e.g., MVC/layered architecture, Clean Architecture, monolith vs microservices, and event-driven architectures.

Architecture is “how you keep your codebase healthy as it grows.”

Teacher truth:
> Most architecture failures are not technical. They’re organizational:
> - unclear boundaries
> - shared mutable state everywhere
> - no ownership
> - no consistent patterns

Patterns are tools. The skill is choosing the right tool at the right time.

---

### Basic intuition (simple)
- Architecture is how you organize code and services to keep the system maintainable.
- Patterns are tools, not rules; pick what fits your constraints.

### Internal working (engine level)
#### MVC / layered approach
Separates concerns:
- controllers (HTTP/interface)
- services (business logic)
- repositories (data access)

#### Clean Architecture (idea)
Dependency direction matters:
- core domain logic should not depend on frameworks
- outer layers (web/db) depend inward

This improves testability and reduces coupling.

#### Monolith vs microservices
- Monolith: one deployable unit; simpler to build and operate early.
- Microservices: independent deploys; can scale teams but increases operational complexity (network, auth, tracing, data consistency).

#### Event-driven systems
Services communicate via events (pub/sub, queues).
Pros: decoupling, resilience, async workflows.  
Cons: debugging complexity, eventual consistency, message duplication.

### Edge cases
- Microservices too early: you pay distributed systems costs without benefits.
- Event-driven without idempotency and observability becomes a nightmare.

### Interview questions
- When would you choose a monolith over microservices?
- How does Clean Architecture help testing?
- What problems do event-driven systems solve, and what new problems appear?

### Real-world usage (Node.js)
- A common Node backend structure is controller/service/repo (layered).
- Event-driven + queues are common for background jobs and integration workflows.

### Practice (architecture exercises)
1) Take a simple “todo API” and sketch a layered architecture (controllers/services/repos). Where do validations live?
2) When does a monolith become painful? List concrete signals (deploy speed, team size, coupling).
3) Design an event-driven workflow (order placed → payment → email). How do you make consumers idempotent?

### Connections to other topics
- **Message queues**: the plumbing for event-driven architecture.
- **System design**: architecture patterns are applied system design.
- **Express**: often used in outer “interface” layers in layered architectures.
