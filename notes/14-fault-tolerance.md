# 14. Fault Tolerance

Fault tolerance is the lecture where the trainer admits *no system is perfect* and then shows us what to do about it. It sits right between message queues and monitoring in the series, and it answers one question: **when things break (and they will), how do we keep the users from noticing?** Interview-gold territory — this is where you stop reciting architecture and start sounding like you've actually lived through a pager going off at 3 a.m.

---

## The story first

The lecture opens with a plain confession: **no application is perfect**. Any moment a system "is not responding the way it should" — that is a **fault/error**. Wide definition on purpose, because faults come at you from everywhere.

The trainer's point is that the paths they arrive by are countable:

- **Hardware** — the physical world deciding it's done with you.
- **Software** — code, configuration, or deploys that you wrote wrong.
- **Human** — the people operating all of it.

And one framing sentence that ties the whole series together: fault tolerance is a **non-functional requirement**, the kind we first mapped in [functional vs non-functional requirements](./03-functional-vs-nonfunctional-requirements.md). You don't bolt it on after an outage — you design it in from the start.

> **Key takeaway:** a distributed system WILL experience faults. The whole discipline of fault tolerance is about deciding, in advance, that the users never feel them.

---

## Breaking faults into three buckets

The trainer gives a clean taxonomy — **hardware, software, human** — with each bucket having its own personality and its own fix.

```mermaid
flowchart TD
    F["faults: system not responding as it should"]
    F --> H["hardware faults"]
    F --> S["software faults"]
    F --> U["human faults"]
    H --> H1["random, abrupt, inevitable"]
    H --> H2["disk full, OOM, server down, cables, power"]
    S --> S1["deterministic, reproducible"]
    S --> S2["bad code, unhandled exceptions, config, deploys"]
    U --> U1["unpredictable"]
    H2 --> M1["answer: redundancy + scaling"]
    S2 --> M2["answer: testing, exceptions, config care"]
    U1 --> M3["answer: deep fixes + accountability"]
```

If you've read fault-tolerance write-ups before, you'll recognise the same ideas under the classic names: **crash faults** (the server died mid-invocation), **omission faults** (something that should be there isn't — a broken link in the chain, a dropped response), **timing/performance faults** (the answer arrives so late it might as well not have), **Byzantine/corrupt responses** (the node answers, but with garbage). The lecture folds all of these into its three buckets, so I'll keep the three and tag the classic terms where the examples line up.

### Hardware faults

The trainer's list: **disk full, out of memory, server down, damaged network cables, power-supply problems, a misconfigured node, a request spike**, even a **compromised database**.

- They are **random, abrupt, inevitable**. Some give you a bit of warning through alerts; most just happen.
- Because you can't predict them, the response is **redundancy and scaling** — more servers, more databases — so that when one piece dies, the group absorbs it.
- The honest gloss from the lecture: hardware is the bucket you have the *least* control over.

### Software faults

These trace straight back to something a human did:

- badly written code, unhandled exceptions, edge cases you only ever discover in production;
- bad microservice / API configuration, deployment issues;
- the classic **dev-vs-prod environment mismatch** — "works in prod not dev, or vice versa — day-to-day life of a software developer."

Two extra points the trainer makes:

- **Slow requests also count as a software fault.** A response that arrives too late is a failure for the user who already left — that's the timing/performance flavour.
- The **merge-request pain point**: two developers working on the same feature/file in parallel must both be thoroughly tested, or they break each other on staging/prod — "one of the most painful things of any developer."

The good news that makes up for it: software faults are **deterministic**. Follow the same steps and you get the same break, which means you can reproduce it, then fix carefully and ASAP.

### Human faults

The one the trainer calls "not really a fault but it comes into the category" — and **the most important one**.

- Humans are the **most unreliable component** of software development. Inanimate components give you alerts beforehand; people sometimes just don't.
- And yet humans are also the **most powerful**: if they stop working, nothing works. Even AI-generated code (agents, open-source models) still needs a human to rectify it and take accountability.
- The lesson for us: that makes you *more* responsible for handling code carefully, not less.

---

## The remediation playbook

One fix per bucket, straight from the lecture:

- **Human faults** — good behaviour; no sloppy band-aid fixes done wrong. Go deep into the system and fix it entirely, everywhere.
- **Software faults** — don't write bad code; handle exceptions; catch edge cases; thoroughly test every build; configure every environment variable carefully; replicate prod configuration on dev so you can reproduce and solve config issues.
- **Hardware faults** — no real control, mostly unpredictable, so **scale the system**.

---

## Faults must not become failures

The core idea of the whole lecture: **a fault in one component must not escalate into a failure of the whole system.** Design so a single dying piece is a local event, never an outage:

```
   BEFORE:                            AFTER:
   component A dies                   component A dies
        |                                  |
        v                                  v
   WHOLE SYSTEM DOWN                 remaining replicas still
   users hit an outage               serve -- no one notices
```

The trainer returns to this principle in the monitoring lecture too — the maintenance checklist literally asks *"if a component is down, do other systems take over (failover)?"*. That expectation is the point. You design the answer to be *yes* before anything ever breaks.

---

## Redundancy, replication and failover

The mechanism is **redundancy**: more than one server, more than one database, so there is always somewhere to hand the work when the current one fails. This is exactly the replication story from [replication and partitioning](./11-replication-and-partitioning.md) paying off at runtime.

The operational word is **failover**:

- **Active (hot) servers** — the ones taking traffic right now. If one dies, the load balancer just stops sending it work and the others absorb the slack.
- **Passive (standby) servers** — fully configured, ready, sitting idle. When the active node dies, the standby takes over so the users don't see a gap.
- Failover is only as good as the detection that triggers it — which is why health checks (next section) and replication go hand in hand: the *data* is already copied, so a different machine can pick up exactly where the dead one stopped.

```mermaid
flowchart LR
    C["clients"] --> LB["load balancer"]
    LB --> A["server A (active)"]
    LB --> B["server B (active)"]
    B --> S["server C (passive standby)"]
    A -- "dies" --> LB
    LB -- "traffic fails over" --> S
    A --> D["database primary"]
    S --> D2["database replica"]
```

And the same story in plain ASCII:

```
   request --> load balancer --> server A (active, serving)
                                server B (passive standby)

   server A dies --> LB spots it via health check
                  --> traffic moves to server B
                  --> B becomes active -- clients never noticed
```

> **Key takeaway:** redundancy + replication + failover is the hardware bucket's answer. You can't prevent a disk from filling up, but you can make sure no single disk is ever a single point of failure.

---

## Health checks and retries — the load balancer's eyes

Failover needs something to detect the death, and that something is the **health check** — the same theme we met in the [load balancing](./10-load-balancing.md) lecture.

- **Passive health checks** — watch the server's real traffic and infer it's sick because it stopped answering.
- **Active health checks** — deliberately probe the component (a ping, a `/health` endpoint) and **constantly watch for 200 responses**; anything else marks it unhealthy and traffic stops being routed there.

The trainer folds this into monitoring as part of the three API metrics — **throughput, error codes, health checks** — detailed in [monitoring and observability](./15-monitoring-and-observability.md). (That 200-longing for healthy responses is the thing I keep going back and forth on in interviews.)

Retries are the natural partner, and the message-queue rule from the previous lecture is the guardrail: a message a subscriber already processed must **never be processed twice**, so the queue deletes a message once handled to guarantee that. Same discipline applies to retrying anything — retrying is safe only when the operation at the other end is safe to run again (idempotent). Retry blindly and a transient timeout turns into a double-processing bug.

---

## Creating is different from maintaining

The lecture shifts gear with a line worth stealing: **"creating a product is entirely different from maintaining that product."** The lifecycle is developed → deployed → maintained, and fault tolerance lives in the last phase.

The trainer's maintenance checklist — the questions you ask once the system is live:

- Are there errors present?
- Is every component fine — server, microservice, database — what is the health of each?
- Are errors being recorded correctly?
- Are there logs to track every customer request?
- Can we do an **RCA (root cause analysis)** so developers can actually debug?
- If a component is down, do other systems take over (failover)?
- Are proper alert systems in place?

Fault tolerance and monitoring are two halves of one discipline: this lecture is the *design so it survives* half; [monitoring and observability](./15-monitoring-and-observability.md) is the *keep watching so you find out when it hurts* half.

---

## Quick revision

- No system is perfect; "not responding the way it should" = a **fault/error**, and faults are guaranteed in distributed systems.
- Three buckets: **hardware** (random, abrupt → redundancy + scaling), **software** (deterministic → testing, exceptions, careful config), **human** (unpredictable → deep fixes everywhere + accountability).
- Software fault means bad code, config, deploys, dev/prod mismatch, merge-request clashes — and **slow responses** count too.
- Core principle: a single component's **fault must not become a system failure** — containment by design.
- The answer is **redundancy + replication** → **failover**: active servers doing work, passive standby ready to take over.
- Failover is blind without **health checks** (passive + active, watch the 200s); retries are safe only when the operation is idempotent (MQs already delete a message once processed).
- Creating a product ≠ maintaining it; the maintenance checklist ends with failover and alerts wired in.

## Interview questions

- What's the difference between a fault and a failure, and how do you stop one becoming the other?
- Name the fault categories you'd design for and the mitigation strategy you'd apply to each.
- Your service must never go down, but any single server can die at any moment — how do you build that?