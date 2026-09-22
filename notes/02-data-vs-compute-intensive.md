# Data-intensive vs Compute-intensive

Before drawing a single box, this is the fork in the road: is the system data-intensive or compute-intensive? Getting this wrong wastes money and points every later decision in the wrong direction — getting it right is pure interview gold. Follows on from [what is system design](01-what-is-system-design.md).

---

## The very first question

- The anti-pattern the lecturer calls out: jumping straight into boxes — client, DB, cache, load balancer — **without understanding the app first**.
- > **First question before any solution:** *What is this application — data-intensive or compute-intensive?* Getting this right saves a lot of cost and steers the whole task correctly.
- Setup he uses: two apps, same user count, same network, both facing **latency** (system taking time to respond).
- Candidate fixes on the table:
  - add multiple databases
  - add caching layers / update caching mechanism
  - upgrade GPU or CPU power
- Which vector you pick depends entirely on the type. Wrong vector = money burned on the wrong component.

---

## Data-intensive systems

- **Definition:** core focus is **data** — store it, gather it, move it, keep updating it as new data arrives, write lots of it.
- Calculation is *not* the focus; the job is just moving data DB -> client.
- Bottlenecks live on **our side**, never the client: slow database response, bulky/inefficient network calls, low server configuration. ("We are not blaming the client to upgrade their CPU.")
- Claim from the lecture: **most applications you work on are data-intensive** — hence all the talk about cache enhancement and sharding.

### Instagram — the running example

- Feed is *not* solved with math; it's "gather a lot of data and show it."
- Feed operations: fetch posts -> sort by time or relevance -> show images/videos -> user ops: like, comment, share.
- The challenge: **millions of users reading and writing at the same time** on millions of pieces of content.
- How Instagram scales:
  - **Multiple databases** — data lives in several places.
  - **Aggressive caching** for fast get/store of hot data.
  - Images/videos are **not** stored in the DB directly — a **CDN** serves them; the DB stores only the location/pointer to the file.
- Crucial insight: to fix latency/scale, "we hardly touch the CPU… or GPU" — the bottleneck is **data movement between components**, not compute.

```
        mobile client
              |
     like / comment / feed request
              v
     +------------------+
     | application code |
     +------------------+
        |            |
    hot data      media file
        v            v
   +---------+   +-------+
   |  cache  |   |  CDN  |  <-- images & videos live here
   +---------+   +-------+
        |            ^ pointer only
        v            |
   +--------------------------+
   | multiple databases       |  <-- posts, likes, comments,
   +--------------------------+      users, metadata
```

### The four worries of data-intensive apps

1. How fast can we **read** data?
2. How **safely** do we store it?
3. How many users can use the **same data simultaneously**?
4. What if a **server / network call / database machine dies**?

- **Solution vocabulary** that grows out of these worries: databases, caching, replication, data sharding, data consistency.
- WhatsApp scale aside: handles roughly **2 to 10 million messages per day** — heavy data-intensive.
- Other examples named: bank transactions, analytics dashboards, log processing systems.

---

## Compute-intensive systems

- **Definition:** exactly the opposite of data-intensive — data retrieved from DB is **low**, but **calculations are heavy**.
- Focus shifts away from DB / caching / replication / sharding -> onto **CPU and GPUs**. Systems are **compute-bounded** (CPU/GPU bound).
- Examples from the lecture:
  - image processing
  - heavy video rendering
  - ML model training / inference
  - simulation
  - cryptography

### The three worries (compute side)

1. How fast can we **compute**?
2. Can we do processes **in parallel** to show data faster? (user keeps using other features while the image processes in the background)
3. How to **reduce computational cost** — can we use GPU instead of CPU?

### The flight-simulator story

- A country hires many pilots yearly; can't spend on real aircraft for untrained pilots -> **simulator machines** train them first, then the real plane.
- Simulator must work **exactly like the actual machine** — terrain, earth, controls computed on the fly.
- That's heavy computation: **no amount of storage or caching fixes it**. Bigger disk = still have to compute every frame.
- Simulator worries: parallel processing? GPU vs CPU? reduce simulator cost? gather good algorithms to do the frequent tasks.
- Recap contrast: in data-intensive apps, focusing on GPU/CPU brings **no big change** — the issue lies in database, network calls, caching algorithms, or server.

---

## Where the money goes

- Components aren't free — putting money in the wrong place = "bad system design for the entire game."
- Money/scale argument: as the user base grows into the **millions**, both cost and architecture diverge. You *must* know which kind you are building before you spend.
- They are **not mutually exclusive**: modern apps do many tasks — intensiveness can depend on the **feature**, not the whole app.

### YouTube — the straddler

- Leans **data-intensive** for serving videos (billions of pieces of content to store, move, deliver).
- Leans **compute-intensive** for recommendations — it must analyze what you're watching to pick what you see next.
- Real systems **mix both**; classify per feature, not just per product.

```mermaid
flowchart TD
    A[App or feature] --> B{Where is the time lost?}
    B -->|Moving data around| C[Data-intensive]
    B -->|Computing on the data| D[Compute-intensive]
    C --> E[Multiple databases<br/>Aggressive cache<br/>CDN + DB pointer<br/>Replication / sharding]
    D --> F[CPU to GPU<br/>Parallel processing<br/>Better algorithms]
    E --> G[Instagram, WhatsApp,<br/>bank transactions,<br/>analytics, log processing]
    F --> H[Image processing,<br/>video rendering,<br/>ML training, simulation,<br/>cryptography]
    C -.->|feature-level split| I[YouTube: serving videos]
    D -.->|feature-level split| J[YouTube: recommendations]
```

---

## The trick — the lecturer's test

- > **If time is lost in data movement -> data-intensive. If time is lost in the computation itself -> compute-intensive.**
- "This one distinction save you a lot of money time and complexity."
- Practical loop:
  1. classify the issue (compute vs data)
  2. *then* pick components
  3. *then* fix
- Tiny sanity sketch of where the minutes go:

```
request arrives
      |
      v
+---------------+     data-intensive:     +------------------+
| spend the time|----> data moving   ---->| DB, cache, CDN,  |
| somewhere     |     between components  | network, servers |
+---------------+                         +------------------+
      |
      | compute-intensive:
      v
+------------------+
| CPU/GPU chewing  |  <-- image render, ML training,
| on the payload   |      simulation, crypto
+------------------+
```

- Wrong call in an interview (or production): pitching sharding and CDNs for a rendering job, or buying GPUs to serve an Instagram feed.

---

## Quick revision

- First question always: **data-intensive or compute-intensive?** Classify before drawing boxes.
- Data-intensive = data volume, transfer, storage, accessibility is the bottleneck; compute is barely touched.
- Instagram playbook: multiple databases + aggressive caching + CDN for media (DB keeps only the pointer).
- Four data worries: fast reads, safe storage, many concurrent users, machine/network dies.
- Compute-intensive = CPU cycles are the bottleneck; fix with GPU, parallelism, better algorithms — storage won't help.
- The trick: time lost moving data -> data; time lost computing -> compute. One distinction, lots of money saved.
- YouTube straddles both: serving videos (data) vs recommendations (compute) — classify per feature.

---

## Interview questions

1. You're designing a photo-sharing feed that's getting slow at 10 million DAU. How do you decide between adding caches/databases and upgrading CPUs/GPUs — and what's your first question?
2. A flight-simulator vendor keeps hitting latency when rendering terrain. Why won't sharding or a CDN fix this, and what should you investigate instead?
3. YouTube seems like one system — where is it data-intensive, where is it compute-intensive, and does that change how you'd split the design interview answer?
