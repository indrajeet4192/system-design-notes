# System Design Notes — Index

My handwritten notes from the **Telusko System Design** series (Evan Reddi's intro, then trainer Akshay). I watched the lectures, turned each one into a dense notebook page, and put them all here so I can revise the whole course in one sitting. Each note ends with a **Quick revision** block and **Interview questions** — that's what I re-read the night before a loop.

The course tells one story (Alien Bank grows from one cashier to a planet-scale system) and every fix we invent turns out to be a real system-design term. The notes follow that arc.

---

## The map

```
 01 foundations ──► 02 compute vs data ──► 03 requirements
                          │
                          ▼
 04 dns ──► 05 apis ──► 06 rest ──► 07 sql ──► 08 nosql
                                      │
                                      ▼
 09 caching ──► 10 load balancing ──► 11 replication/partitioning ──► 12 cap
                                      │
                                      ▼
 13 message queues ──► 14 fault tolerance ──► 15 monitoring/observability
                          │
                          ▼
 16 the interview playbook ──► 17 case study: video streaming
```

## Foundations

- **[What is System Design?](notes/01-what-is-system-design.md)** — the Alien Bank story, "the bank is very slow", and the five issues that map to real concepts. The idea that *code that works* ≠ *a system that survives millions of users*.
- **[Data-Intensive vs Compute-Intensive Systems](notes/02-data-vs-compute-intensive.md)** — which resource is the bottleneck, and why the answer changes the architecture. 99.9% availability math included.
- **[Functional vs Non-Functional Requirements](notes/03-functional-vs-nonfunctional-requirements.md)** — what the system must *do* vs how well it must do it (availability, latency, scalability, consistency). The requirements table every interview opens with.

## The request path

- **[DNS — How a URL Becomes an IP](notes/04-dns.md)** — the ~10ms resolution chain, all 8+ levels from browser cache to TLD servers, and why DNS is the first thing you debug when "the site is down".
- **[APIs](notes/05-apis.md)** — the contract between client and server: HTTP verbs, status codes, idempotency, and how a good API surface makes every downstream decision easier.
- **[REST APIs](notes/06-rest-apis.md)** — REST principles in practice, resources vs actions, versioning, pagination, rate limiting. The one style I'll be asked to design most in interviews.

## Data layer

- **[SQL Databases](notes/07-sql-databases.md)** — relational modeling, ACID, indexes, joins, and the vertical-scaling ceiling that pushes everyone toward sharding.
- **[NoSQL Databases](notes/08-nosql-databases.md)** — document/key-value/column/graph families, why they scale horizontally, and when the flexibility beats consistency. Mongo, Cassandra, DynamoDB-style trade-offs.
- **[Caching](notes/09-caching.md)** — read-through / write-through / write-around / write-back, TTL, cache invalidation, eviction policies (LRU/LFU), and the "what dies first?" disasters caching fixes and causes.
- **[Load Balancing](notes/10-load-balancing.md)** — round-robin, least connections, consistent hashing, health checks, and how the LB sits between every layer (client-LB, LB-application, LB-DB).
- **[Replication and Partitioning](notes/11-replication-and-partitioning.md)** — leader/follower, quorum reads and writes, sharding keys, and why replication + partitioning together give both durability and scale.
- **[CAP Theorem](notes/12-cap-theorem.md)** — consistency vs availability vs partition tolerance, and how the choice (CP vs AP) shapes every real system. The note that breeds follow-up questions.

## Plumbing and resilience

- **[Message Queues](notes/13-message-queues.md)** — async decoupling, producers/consumers, at-least-once vs exactly-once, DLQs, and the classic "slow-down-your-customer traffic" pattern. Kafka/RabbitMQ-style thinking.
- **[Fault Tolerance and High Availability](notes/14-fault-tolerance.md)** — redundancy, failover, health checks, circuit breakers, graceful degradation, and the "design for failure" mindset with real SLOs.
- **[Monitoring and Observability](notes/15-monitoring-and-observability.md)** — metrics vs logs vs traces, dashboards, alerts, and what "the system is slow" actually means when you can see P95/P99.

## The playbook

- **[Designing a System Design Interview](notes/16-designing-a-system-interview.md)** — the structured approach: clarify requirements, estimate scale, outline components, drill into the hard part, then discuss trade-offs and follow-ups.
- **[Case Study: Video Streaming Service](notes/17-case-study-video-streaming.md)** — the full walk-through end to end: global CDN, video segmentation (50GB / 1200 segments), ABR at up to 300Mbps, caching, and load distribution — a whole course compressed into one design.

---

## How I revise

1. Read the map above, pick the topic I'm weakest on.
2. Read that note's `## Quick revision` first — if a bullet doesn't ring a bell, the whole note gets re-read.
3. Answer the `## Interview questions` out loud (no peeking).
4. For the final pass, [16](notes/16-designing-a-system-interview.md) + [17](notes/17-case-study-video-streaming.md) are the capstone — they string every earlier concept into one flow.

---

## Quick revision

- One story, many terms: every concept in this repo comes from a real bank that got too popular.
- Foundations (01-03) define *what* we're building and *how well*; the middle (04-12) is the request path + data layer; the end (13-17) is resilience + the interview itself.
- If a note needs a companion, it links with a relative link — follow the thread.
- Every note ends with revision bullets + interview questions: that's the 15-minute loop before any interview.

## Interview questions

1. Walk me through the path a request takes in a system you've designed — which note(s) cover each hop?
2. Pick any two notes and explain how they interact (e.g. caching + load balancing, or replication + CAP).
3. Which topics do you consider the "big three" for a system design interview, and why?