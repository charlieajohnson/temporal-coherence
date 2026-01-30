# Temporal Model (Draft)

This document defines a minimal, implementation-independent model for representing
time in long-horizon, intermittently connected sensing systems.

The goal is not to prescribe a synchronisation protocol or transport mechanism,
but to specify the smallest set of properties required for temporal coherence
to survive drift, buffering, and delayed observation.

---

## 1. Scope and Non-Goals

This model applies to systems where:

- Observations are generated remotely or autonomously
- Continuous synchronisation is unavailable or unreliable
- Data may be buffered, reordered, or delivered long after observation
- Downstream systems are expected to reason about sequence, simultaneity, or causality

This model explicitly does **not** attempt to:

- Define a new clock synchronisation protocol
- Replace existing transport or storage systems
- Enforce a global notion of “true time”
- Optimize for latency, throughput, or precision

It concerns **coherence**, not accuracy.

---

## 2. Core Principle

Time must be representable as a **bounded estimate with provenance**, not as a
single scalar value whose meaning is implicit or context-dependent.

A system is temporally coherent if it can answer:

> *What did we know, and when did we know it?*

---

## 3. Event Representation

Each observation MUST carry the following temporal state:

### 3.1 Event Time Interval

An observation’s event time is represented as an interval:
t_event ∈ [t_min, t_max]

This interval bounds when the physical event could have occurred, given all
known sources of drift, buffering, and delay at the time of observation.

The interval MAY be narrow or wide, but it MUST be explicit.

---

### 3.2 Uncertainty Growth

The width of the event time interval MUST be non-decreasing unless a new
temporal anchor is applied.

Uncertainty growth may arise from:

- Clock drift
- Lack of synchronisation
- Buffering duration
- Transport latency
- Environmental effects (e.g. temperature)

---

### 3.3 Clock Provenance

Each event MUST record the provenance of its time estimate, such as:

- Sensor-local clock
- Gateway or aggregator clock
- External reference (e.g. GNSS)
- Inferred or reconstructed time

Provenance is descriptive, not normative.

---

### 3.4 Anchor State

If a temporal anchor is applied (e.g. synchronisation, recovery, calibration),
the event MUST record:

- The anchor source
- The time since last anchor at observation
- The effect of the anchor on uncertainty

Anchors constrain uncertainty; they do not rewrite history.

---

## 4. Invariants

A temporally coherent system MUST uphold the following invariants:

### 4.1 Event Time Is Immutable

Event time intervals are never overwritten.
Corrections are applied as additional constraints, not destructive edits.

---

### 4.2 Uncertainty Never Shrinks Without Evidence

The uncertainty interval MAY only contract in response to a new,
explicit temporal anchor.

Implicit correction is forbidden.

---

### 4.3 Corrections Are Additive and Auditable

All temporal corrections MUST be:

- Additive
- Traceable
- Reversible in principle

At any point, the system must be able to reconstruct its prior temporal state.

---

### 4.4 Arrival Time Is Not Event Time

Ingest or arrival timestamps MUST NOT replace event time.
They may coexist, but they are semantically distinct.

---

## 5. Temporal Ordering Semantics

Ordering between two events A and B is defined as follows:

- **Ordered** if `A.t_max < B.t_min`
- **Reverse ordered** if `B.t_max < A.t_min`
- **Ambiguous** if intervals overlap

Ambiguity is a valid state and MUST be representable.

---

## 6. Queryability

The model enables queries such as:

- What events could have influenced a decision?
- Which correlations are temporally plausible?
- What was known at a given moment?
- Which conclusions rely on ambiguous ordering?

A system that cannot express these queries is not temporally coherent.

---

## 7. Relationship to Existing Systems

Most real-world systems already reason about time uncertainty informally
through human interpretation, operational heuristics, or post-hoc analysis.

This model makes that reasoning:

- Explicit
- Machine-readable
- Enforceable
- Auditable

---

## 8. Summary

Temporal coherence is an architectural property, not a feature.
It emerges from a small number of invariants applied consistently
across sensing, buffering, transport, and ingestion.

This document defines the minimum required to preserve it.
