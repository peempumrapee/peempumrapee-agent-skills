# AI engineering review

Trace a representative request through prompts, retrieval, model calls, tools, and output consumers:

- Prompt roles and trust boundaries, untrusted retrieved/tool content, tool permissions, and confirmation requirements for external actions.
- Structured output and tool-call schemas, validation, retries, idempotency, termination conditions, and error propagation.
- Retrieval relevance, chunk/source attribution, stale knowledge, missing evidence, and behavior when retrieval fails.
- Evaluation coverage for task success, unsupported answers, adversarial inputs, and realistic failure cases; distinguish measured quality from assumptions.
- Context/token limits, latency/cost budgets, fallbacks, model/version compatibility, and tracing where relevant to the requirement.

Use concrete failure paths and supplied evaluation evidence. Do not make paid model calls, run tools against external services, or treat a prompt-only instruction as enforced authorization. Report needed runtime verification to the parent.
