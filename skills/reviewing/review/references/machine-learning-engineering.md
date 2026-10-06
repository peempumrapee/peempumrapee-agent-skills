# Machine learning engineering review

Follow the data and model lifecycle relevant to the change:

- Train/validation/test splits, temporal or entity leakage, preprocessing fitted on the correct split, and label availability.
- Evaluation metrics, baselines, cohort coverage, uncertainty, and whether evaluation matches deployment conditions.
- Reproducibility of data versions, code, seeds, configuration, and model artifacts; avoid promising determinism where unsupported.
- Training/serving feature consistency, missing features, model compatibility, and deployment rollback.
- Drift and quality monitoring, feedback-loop bias, resource limits, and failure handling where required.

Ground findings in code, artifact metadata, and supplied results. Do not infer model quality from architecture alone or start training/evaluation jobs. Ask the parent for worker-run evidence when needed.
