---
name: supplementary-table-fdr-validation
description: Use when a reimplementation of METASPACE-ML FDR estimation (weighted
  target-decoy ranking with Laplace correction) produces per-ion FDR values that must
  be checked against the paper's published Supplementary Data 10 scores/FDR table.
when_to_use_negative:
- When the goal is to compute FDR from raw scores rather than validate already-computed
  FDR values against a published table
- When validating FDR from the rule-based MSM scoring system, whose feature-weighting
  differs from METASPACE-ML's machine-learned scores
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: METASPACE-ML
  role: Source of the machine-learned prediction scores whose FDR estimates are being
    validated against Supplementary Data 10
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE
  role: Annotation engine platform whose target/decoy scoring outputs feed the FDR
    comparison; python-client and GraphQL API provide access to annotations
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:72f77142d25867330971cc28c0c2719cefdd2e7cb23774c9fdeb5b14479ce0d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# supplementary-table-fdr-validation

## Summary

A validation skill for independently recomputed METASPACE-ML FDR estimates: per-ion FDR values from the weighted target-decoy ranking are compared, across the prediction-score range, against the reported scores/FDR table published in Supplementary Data 10, with disagreements documented in a validation report.

## When to use

Use when a reimplementation of METASPACE-ML FDR estimation (weighted target-decoy ranking with Laplace correction) produces per-ion FDR values that must be checked against the paper's published Supplementary Data 10 scores/FDR table.

## When NOT to use

- When the goal is to compute FDR from raw scores rather than validate already-computed FDR values against a published table
- When validating FDR from the rule-based MSM scoring system, whose feature-weighting differs from METASPACE-ML's machine-learned scores

## Inputs

- fdr_estimates.csv with per-ion FDR values, METASPACE-ML prediction scores, and target/decoy labels
- Supplementary Data 10 scores/FDR table from DOI 10.1038/s41467-024-52213-9
- Prediction scores of target ions and SD-sampled decoy ions from the scores table

## Outputs

- Validation report recording agreement between computed and reported FDR across the prediction-score range
- Flagged list of ions where computed FDR diverges from Supplementary Data 10 values

## How to apply

After computing per-ion FDR via the single weighted target-decoy ranking (decoys weighted 1/SD, cumulative weighted decoy counts over cumulative target counts, Laplace rule-of-succession smoothing), join the recomputed values to the Supplementary Data 10 table by ion identity and compare FDR across the full METASPACE-ML prediction-score range rather than at a single threshold. Check both the ordering of ions by FDR and the actual FDR magnitudes, since the Laplace pseudocount shifts low-count regions of the ranking. Record agreement and any discrepancies in a validation report, noting the prediction-score range over which agreement holds. The repository (github.com/metaspace2020/metaspace, Apache-2.0) and its wiki are the reference implementation context for the annotation engine producing these scores.

## Related tools

- **METASPACE-ML** (Source of the machine-learned prediction scores whose FDR estimates are being validated against Supplementary Data 10) — https://github.com/metaspace2020/metaspace
- **METASPACE** (Annotation engine platform whose target/decoy scoring outputs feed the FDR comparison; python-client and GraphQL API provide access to annotations) — https://github.com/metaspace2020/metaspace

## Examples

```
import pandas as pd; est = pd.read_csv('fdr_estimates.csv'); ref = pd.read_csv('supplementary_data_10.csv'); cmp = est.merge(ref, on='ion_id'); report = cmp[['prediction_score','fdr_x','fdr_y']].describe()
```

## Evaluation signals

- Per-ion FDR values from the reimplementation agree with Supplementary Data 10 across the prediction-score range, not only at a fixed FDR cutoff
- Every ion in fdr_estimates.csv carries a target/decoy label, a prediction score, and a non-negative continuous FDR value
- The validation report documents the compared prediction-score range and any residual discrepancies
- Laplace-smoothed FDR is monotone non-decreasing as prediction score decreases along the merged target-decoy ranking

## Limitations

- Decoy ions show heterogeneity in similarity to target ions; 'Further investigations are needed into how this affects FDR estimation and in finding the most reliable way of decoy generation'
- Agreement is checked against values reported in Supplementary Data 10; exact per-decoy SD sampling realizations may cause small deviations

## Evidence

- [abstract] computationally-efficient False Discovery Rate estimation: "METASPACE-ML is a machine learning-based approach addressing this challenge which incorporates new scores and computationally-efficient False Discovery Rate estimation."
- [intro] integration into METASPACE engine: "integrate METASPACE-ML into the existing cloud METASPACE software as well as a separate web app"
- [readme] repository README describing the platform projects: "The METASPACE platform hosts an engine for metabolite annotation of imaging mass spectrometry data as well as a spatial metabolite knowledgebase"
- [discussion] decoy generation reliability caveat: "Further investigations are needed into how this affects FDR estimation and in finding the most reliable way of decoy generation"
