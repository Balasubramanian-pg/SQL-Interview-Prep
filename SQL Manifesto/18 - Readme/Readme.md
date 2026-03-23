## Enterprise SQL Manifesto

### 17 Golden Rules for ACID Compliance, Business Continuity, and Production Readiness

This manifesto defines **non-negotiable principles** for writing SQL in enterprise environments.
It is vendor-agnostic and applies across platforms such as Snowflake, SQL Server, Oracle, PostgreSQL, and cloud data warehouses.

The reasoning approach used here is straightforward:
I identified the failure modes that cause data corruption, outages, security leaks, and unmaintainable systems in large organizations, then codified the counter-principles that consistently prevent them.

## I. Foundational Guarantees

### Rule 1: Transactions Are Sacred

All data-changing operations **must be transactional**.

* Use `BEGIN`, `COMMIT`, and `ROLLBACK` explicitly.
* Never rely on implicit commits.
* Partial success is considered failure.

**Why:** Guarantees Atomicity and Consistency.

### Rule 2: ACID Is Not Optional

Every write path must preserve:

* **Atomicity:** All or nothing.
* **Consistency:** Business rules never violated.
* **Isolation:** Concurrent sessions do not leak state.
* **Durability:** Committed data survives failures.

If any letter is compromised, the system is **not enterprise-ready**.

### Rule 3: Determinism Over Cleverness

SQL must produce the **same output given the same input**, always.

* No reliance on unordered results.
* No ambiguous joins.
* No logic that depends on execution timing.

**ASSUMPTION:** Parallel execution exists.
This assumption is necessary because all enterprise databases execute queries concurrently.

## II. Data Integrity and Modeling

### Rule 4: Schema Is a Contract

Schemas are **public APIs**, not scratchpads.

* No silent column changes.
* No type weakening.
* No reused columns with new meanings.

Breaking a schema is a breaking change.

### Rule 5: Constraints Over Trust

Enforce rules at the database level.

* Primary keys
* Foreign keys
* Check constraints
* Not null constraints

Application-only validation is insufficient.

### Rule 6: Idempotency Is Mandatory

All scripts must be safe to run **multiple times**.

* Use `CREATE OR REPLACE` where appropriate.
* Guard inserts with keys or hashes.
* Avoid side effects tied to execution count.

This is essential for disaster recovery and replays.

## III. Performance With Predictability

### Rule 7: Predictable Performance Beats Fast Queries

A stable 500 ms query is better than a volatile 50 ms query.

* Avoid data-dependent execution paths.
* Avoid correlated subqueries at scale.
* Avoid hidden Cartesian explosions.

### Rule 8: Explicit Is Faster Than Implicit

Never rely on engine defaults.

* Explicit joins
* Explicit casts
* Explicit filters

Implicit behavior changes across versions and vendors.

### Rule 9: Design for Scale, Not Today

Assume data volume will grow by **10x without notice**.

* No `SELECT *`
* No logic that assumes “small tables”
* No hard-coded limits that affect correctness

## IV. Security and Access Control

### Rule 10: Least Privilege Always

SQL must assume **minimal permissions**.

* Separate read and write roles.
* No direct table access when views suffice.
* Never embed credentials or secrets.

### Rule 11: Security Logic Lives Near the Data

Row-level and column-level security must be enforced in SQL.

* Views
* Policies
* Secure functions

UI-only security is not security.

## V. Business Continuity and Resilience

### Rule 12: Failure Is a First-Class Scenario

Every write operation must define:

* What happens if it fails
* How it rolls back
* How it is retried safely

Unhandled failure is technical debt.

### Rule 13: Time Travel and Recovery Are Assumed

SQL design must support:

* Backfills
* Replays
* Historical correction

Destructive updates without recovery paths are forbidden.

### Rule 14: Backward Compatibility Is a Promise

Production SQL must not break existing consumers.

* Additive changes preferred.
* Deprecation before removal.
* Versioned views when necessary.

## VI. Observability and Maintainability

### Rule 15: SQL Must Explain Itself

Readable SQL is a production requirement.

* Clear naming
* Logical CTE structure
* Comments that explain **why**, not **what**

Unreadable SQL fails future audits.

### Rule 16: Business Logic Is Explicit and Centralized

Do not scatter logic across queries.

* One definition of metrics
* One definition of hierarchies
* One definition of security rules

Duplication guarantees inconsistency.

### Rule 17: If You Can’t Test It, It’s Not Done

Every critical SQL asset must be testable.

* Known input, known output
* Edge cases documented
* Failure cases validated

Untested SQL is experimental code.

## Closing Statement

Enterprise SQL is not about writing queries that work.
It is about writing **systems that survive growth, failure, audits, and humans**.

If even one rule is knowingly violated, the system is operating on borrowed time.
