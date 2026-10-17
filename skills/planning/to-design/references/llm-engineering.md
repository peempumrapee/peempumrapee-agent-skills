# LLM Engineering

Read for changed model, prompt, retrieval, tool-use, or evaluation behavior. Combine with other guides only for affected boundaries, such as a web interface or ingestion pipeline. Do not assume a vector database, agent framework, or model provider is required.

## Consequential questions

- Which responsibilities belong to deterministic code, model inference, retrieval, or tools? What output contract do callers rely on?
- What changes in model/prompt versions, context construction, structured output, retrieval selection, or grounding/citation behavior?
- How do representative evaluation cases measure task success and unacceptable failures? Which regressions block release?
- How are untrusted user/retrieved content, prompt injection, sensitive data, and tool permissions bounded? What requires human approval?
- What happens on invalid output, missing evidence, provider errors, rate limits, timeouts, or streaming interruption? Are retries safe for tool side effects?
- What latency, token/cost, context-size, and observability constraints matter? How will model or retrieval drift be detected and a change rolled back?

## Detail worth preserving

Specify changed input/output schemas, relevant prompt/model configuration or version references, retrieval boundaries, tool authorization, and deterministic failure/fallback behavior. Separate intended quality targets from measured results. Link existing datasets, prompts, evaluation definitions, and provider contracts; never include secrets or sensitive raw traces.

## Validation and output hints

Plan representative quality evaluations and applicable adversarial, malformed-output, timeout, fallback, and tool-permission checks. Account for nondeterminism; one successful sample is not quality evidence. State dataset scope, comparison baseline, pass thresholds, and unverified assumptions when consequential.

Use API detail only for changed caller contracts, interactions for retrieval/tool/fallback ordering, architecture for changed boundaries, and validation detail for substantial evaluation plans. A prompt-only change may need only an overview and references to existing evaluations. Do not generate a full web or infrastructure design unless those areas change.
