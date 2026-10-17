# Infrastructure

Read for changed environments, networking, identity, capacity, deployment, or recovery. Focus on the affected resources and blast radius, not a complete infrastructure inventory. Link project architecture and operational conventions; do not prescribe a cloud or deployment tool.

## Consequential questions

- Which resources, environments, owners, and consumers change? Which dependencies and failure domains constrain the design?
- How do network reachability, trust boundaries, identities, least-privilege permissions, and secret references change?
- What availability, scaling, resource/cost, quota, and capacity constraints are material? Which require measurement?
- How will the change be provisioned, configured, promoted, and detected as drift? What existing source owns configuration?
- Can rollout be staged safely? What migrations, compatibility windows, health checks, rollback steps, or recovery limits apply?
- What happens under dependency failure or partial deployment? How are monitoring, alerting, backup/restore, and operational ownership affected?

## Detail worth preserving

Record changed topology and configuration contracts, resource dependencies, permission boundaries, and consequential rollout/rollback or recovery behavior. Distinguish desired configuration from verified runtime state. Reference secret names or safe configuration identities, never values. Link unchanged infrastructure definitions and runbooks.

## Validation and output hints

Plan appropriate configuration validation, isolated/local checks, permission and connectivity assertions, health signals, and recovery tests. Identify environment-specific checks that remain unverified. A design is not authorization to apply infrastructure, test production, change account security, or incur external cost; actual execution follows its own authorization and safety gates.

Use deployment detail for consequential environment/rollout changes, architecture for changed resource boundaries, and interactions only when ordering clarifies behavior. Keep a small configuration change in the overview. Include application, data, or LLM guidance only if their behavior or contracts also change.
