# Data engineering review

Trace representative records through ingestion, transformation, and storage against the design. Focus on relevant failure modes:

- Schema, type, nullability, units, timezone, and key contracts across boundaries; schema evolution and consumers.
- Join cardinality, aggregation grain, deduplication, ordering, late arrivals, and event-time versus processing-time assumptions.
- Retry idempotency, partial writes, checkpoints, replay/backfill behavior, and transaction boundaries.
- Data quality checks and reconciliation: distinguish missing, invalid, duplicated, and delayed data.
- Partitioning, query cost, lineage, retention, and observability where these affect stated requirements.

Cite the transformation or contract and a concrete input that produces the wrong output. Do not query live data or launch pipelines as a reviewer; request worker validation and state unverified assumptions.
