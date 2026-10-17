# Data Platform & Pipeline

Read for changed ingestion, transformation, storage ownership, orchestration, or data-consumer behavior. Distinguish platform-wide boundaries from a local pipeline change. Apply relevant concerns without prescribing a platform or creating a document per concern.

## Consequential questions

- What are the sources, consumers, owners, and authoritative datasets? Which schema, key, quality, retention, or access contracts change?
- Is processing batch, streaming, or both? What freshness, volume, latency, and cost constraints matter?
- How are incremental progress and recovery tracked: partitions, watermarks, checkpoints, or equivalent project mechanisms?
- What happens with duplicate, late, out-of-order, missing, or malformed data? Define replay and deduplication behavior rather than assuming exactly-once processing.
- Can jobs be retried, backfilled, or run concurrently without corrupting live outputs? What bounds and dependencies govern orchestration?
- How do schema evolution, lineage, consumer compatibility, and partial publication affect rollout? Who owns reconciliation and recovery?

## Detail worth preserving

Specify changed schema and key semantics, processing boundaries, write/publication behavior, recovery state, and material invariants. Explain consequential scheduling, retries, backfills, and failure handling. Link existing catalogs, contracts, lineage, access policies, and platform architecture instead of duplicating them.

## Validation and output hints

Select checks for transformation correctness, data quality, schema compatibility, freshness, reconciliation, and relevant replay/failure paths. Define expected outcomes for duplicates, partial runs, and recovery where applicable. Label scale or cost assumptions that need measurement.

Keep a small transformation design in the overview. Add architecture detail for ownership/storage boundaries, interactions for consequential ordering or replay, deployment for rollout changes, and validation detail when scenarios are substantial. Contracts may live in an existing schema source or a precise overview section; a pipeline does not automatically need an API file or frontend design.
