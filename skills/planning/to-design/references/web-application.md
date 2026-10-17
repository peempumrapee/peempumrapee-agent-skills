# Web Application

Read for changed user-facing behavior, web APIs, authentication, or frontend/backend contracts. A backend-only service change does not require UI design. Use only applicable concerns; this is not a document checklist or stack recommendation.

## Consequential questions

- Which routes, users, or API consumers are affected? What remains compatible?
- Where are validation, authorization, and data ownership enforced? Distinguish authentication from permission to access each resource.
- For affected UI only: what are loading, empty, success, error, and recovery states? What accessibility, keyboard, or responsive behavior changes?
- What changes across client/server boundaries: request and response shapes, errors, pagination, caching, or concurrency semantics?
- Can retries or repeated submissions duplicate effects? Are stale responses, partial success, or optimistic updates relevant?
- Does the change affect sensitive data, session handling, trust boundaries, or user-visible performance?

## Detail worth preserving

Keep changed contracts and permissions exact, with representative success and failure examples. Define consequential UI states or navigation only when affected. For backend-only flows, capture service ordering and state transitions when needed; do not invent frontend interactions. Link unchanged API schemas, components, and security conventions.

## Validation and output hints

Check applicable success and negative paths, unauthorized access, invalid inputs, compatibility, and repeated requests. For changed UI, include user-journey and relevant accessibility checks; require actual cross-layer checks when the contract spans client and server.

Use the overview for small changes. Add API detail for changed external contracts and interactions for consequential sequencing or state transitions. Architecture or deployment detail is justified only by a corresponding change. These hints do not require separate files or every possible test.
