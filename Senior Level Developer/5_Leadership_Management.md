# Module 5: Leadership & Engineering Management

A senior engineer (8-10 years) is expected to be a force multiplier. This module focuses on the soft skills, strategic thinking, incident management, and processes required to lead teams and drive technical excellence across an entire engineering department.

---

## Technical Strategy & Alignment

### 51. How do you align an engineering team's output with overarching OKRs (Objectives and Key Results)?
**Context**: Engineers can spend weeks refactoring a system that no customer cares about. Senior engineers must translate business goals into technical execution.

**Deep Dive**:
- **The Process**: If the company Objective is "Decrease Customer Churn by 10%," the engineering team must map their technical work to it.
    - *Example*: "Increase homepage load speed" is a technical metric, not a business one. But if data shows customers bounce when load time $> 3s$, the engineering Key Result becomes: "Reduce homepage p99 latency from 4s to 1.5s." This directly supports the churn objective.
- **The Veto**: A Senior Engineer must be comfortable rejecting "cool" technical projects (like migrating from REST to GraphQL) if they do not explicitly serve an immediate or medium-term OKR.
- **Evangelism**: Constantly reminding the team *why* they are building a feature. "We are refactoring this payment pipeline so we can launch in Europe next quarter."

### 52. How do you measure the success and ROI of a massive infrastructure migration?
**Context**: "We need to move from on-prem to AWS" or "We need to rewrite this Ruby monolith in Go." These projects take 18 months. How do you prove it was worth it?

**Deep Dive**:
- **Baselining**: Before writing a single line of code, establishing the current baseline. What is the current cloud bill? What is our deployment frequency? What is our mean time to recovery (MTTR)?
- **ROI Metrics (The Why)**:
    - *Velocity*: Did we increase deployments from once a week to 15 times a day (DORA Metrics)?
    - *Cost*: Did our infrastructure bill drop by 20% by utilizing auto-scaling?
    - *Reliability*: Did our error rate drop from 2% to 0.1%?
- **The Trap**: Most migrations claim success just by "turning the old servers off." A true senior engineering leader defines success by proving the new architecture actively improved the business baseline.

### 53. Describe how you drive technical consensus across disparate teams with conflicting priorities.
**Context**: Team A wants to adopt Kafka. Team B wants to standardize on RabbitMQ. Neither reports to each other. Stalemates occur.

**Deep Dive**:
- **Request For Comments (RFC) Process**: Mandate that architectural decisions are written down. Writing forces clarity. The document must define: Goal, Non-Goals, Proposed Solution, Alternatives Considered, and **Trade-offs**.
- **Depersonalization**: Shift the conversation from "My framework vs your framework" to "Which framework fits our unique business constraints better?"
- **Proof of Concept (POC) vs Prototypes**: Build a limited, 3-day timeboxed POC for both solutions emphasizing the hardest edge cases. Let the code, and the metrics, decide the winner.
- **Disagree and Commit**: If consensus cannot be reached, the Architect or Tech Lead makes a final ruling. Everyone else must fully support the decision, even if they initially argued against it.

---

## Engineering Culture & Processes

### 54. How do you handle production outages? (The Incident Commander Role)
**Context**: At 3 AM, the entire production database drops. Panic is counterproductive.

**Deep Dive**:
- **Roles & Responsibilities**: Instantly establish a clear hierarchy on the Zoom call.
    - *Incident Commander (IC)*: Runs the incident. Makes high-level decisions ("Fail over to Region B"). The IC **does not write code or touch terminals**.
    - *Primary Responder (Ops)*: Shares screen, executes terminal commands, and applies hotfixes.
    - *Scribe / Comms*: Logs every action taken and their timestamps in a Slack channel, and updates the public Status Page for customers.
- **Mitigation over RCA**: The only goal during an active P1 incident is to stop the bleeding. If restarting all servers fixes the issue, restart them. Do not spend two hours investigating *why* they crashed while the site is offline.
- **Blameless Post-Mortem (Root Cause Analysis - RCA)**: 48 hours later, run the "5 Whys" analysis. Focus exclusively on "How did the system allow a human to make this mistake?" instead of "Why did John delete the database?"

### 55. How do you manage "glue work" within a team, ensuring it's recognized and distributed fairly?
**Context**: Glue work is the non-coding effort that holds a project together: updating stale wikis, mentoring juniors, fixing CI/CD pipeline flakes, organizing meetings. It is often invisible during performance reviews.

**Deep Dive**:
- **The Risk**: Engineers who consistently pick up glue work are often penalized during promotions because they "shipped fewer features."
- **The Solution**: 
    1. *Make it visible*: Put glue work explicitly onto the Sprint board as a ticket. Assign points/hours to it.
    2. *Rotate the burden*: Do not let one "helpful" engineer absorb all of it. Establish a "Batman" or "Janitor" rotation—one engineer each sprint does zero product work and focuses entirely on tech debt, PR reviews, and pipeline unblocking.
    3. *Reward it*: In 1-on-1s and performance reviews, explicitly mention their impact as a force multiplier.

### 56. Describe your approach to interviewing and hiring senior engineers. What signals do you look for?
**Context**: Hiring a bad senior engineer is catastrophic for team morale and architecture.

**Deep Dive**:
- **Anti-Signals**: The "Brilliant Jerk." The candidate who steamrolls the interviewer, ignores edge cases, or blames their former team for failures.
- **The Signals**:
    - *Humility & Empathy*: A willingness to say "I don't know, but here is how I'd find out."
    - *Trade-off articulation*: They don't just say "Microservices are better." They say "Microservices solve organizational scaling but introduce brutal eventual consistency problems."
    - *Mentorship mindset*: Ask them about a time they elevated someone else's career. The true mark of a senior is their ability to level up the people around them.

### 57. How do you evaluate and improve developer productivity? (DORA Metrics)
**Context**: "We feel slow" is a subjective complaint. Senior leaders require objective data.

**Deep Dive**:
- Utilize the **DORA (DevOps Research and Assessment)** metrics:
    1. *Deployment Frequency*: How often do we push to production? (Aim for multiple times a day).
    2. *Lead Time for Changes*: How long does a commit take to go from Git to Production? (Track the CI/CD pipeline latency).
    3. *Change Failure Rate*: What percentage of deployments cause an incident? (A high rate means your testing suite is inadequate).
    4. *Time to Restore Service*: When it breaks, how fast can we roll back or fix it?
- **Implementation**: Do not use these metrics to punish individuals (e.g., "Developer A pushed the least code"). Use them to attack systemic bottlenecks (e.g., "Our code review process takes 72 hours, we need to fix that bottleneck.")
