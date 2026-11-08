---
name: target-decoy-fdr-estimation
description: Use when assigning FDR-controlled confidence values to metabolite ion
  annotations from METASPACE-ML prediction scores, requiring a single weighted target-decoy
  ranking and Laplace-smoothed per-ion FDR output.
when_to_use_negative:
- The scoring model has not been trained and raw METASPACE-ML prediction scores for
  targets and SD-sampled decoys are not yet available — this skill only post-processes
  existing scores.
- The annotation task uses the rule-based Metabolite Signal Matching (MSM) scoring
  path rather than METASPACE-ML prediction scores.
- Data types such as QTOF-based or single-cell metabolomics datasets that are underrepresented
  in the model, where FDR estimates may not be statistically reliable.
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: METASPACE-ML
  role: Trained GBDT model that provides the prediction scores for target and SD-sampled
    decoy ions consumed by the FDR ranking procedure
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:7bebb9626a61e9a3c640fb40fdc0e6259a56df2f86d4ebea00bd4bf631c76364
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE-ML
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# target-decoy-fdr-estimation

## Summary

Reconstructs the METASPACE-ML target-decoy False Discovery Rate estimation, which ranks target and SD-sampled decoy ions by machine-learning prediction score and converts the cumulative decoy-weight/target-count ratio into a smoothed, continuous per-ion FDR confidence value.

## When to use

Use when assigning FDR-controlled confidence values to metabolite ion annotations from METASPACE-ML prediction scores, requiring a single weighted target-decoy ranking and Laplace-smoothed per-ion FDR output.

## When NOT to use

- The scoring model has not been trained and raw METASPACE-ML prediction scores for targets and SD-sampled decoys are not yet available — this skill only post-processes existing scores.
- The annotation task uses the rule-based Metabolite Signal Matching (MSM) scoring path rather than METASPACE-ML prediction scores.
- Data types such as QTOF-based or single-cell metabolomics datasets that are underrepresented in the model, where FDR estimates may not be statistically reliable.

## Inputs

- scores table with METASPACE-ML prediction scores for target ions
- scores table with SD-sampled decoy ion prediction scores and per-decoy standard deviations (SD) of sampled similarities

## Outputs

- fdr_estimates.csv containing per-ion continuous FDR values, prediction scores, and target/decoy labels
- validation report comparing computed FDR values against Supplementary Data 10

## How to apply

Load the METASPACE-ML prediction scores of target ions and SD-sampled decoy ions from the scores table into a pandas DataFrame, then merge both classes into a single ranking sorted in descending order by prediction score, assigning each decoy ion a weight of 1/SD derived from the standard deviation of that decoy's sampled similarities. Traverse the ranking cumulatively with numpy, summing weighted decoy counts and target counts at each ion position. Apply the Laplace rule-of-succession pseudocount correction to the cumulative decoy-weight/target-count ratio, producing a smoothed, continuous FDR value for every ion rather than only discrete operating points. Write the per-ion FDR values alongside prediction scores and target/decoy labels to fdr_estimates.csv, then verify against the reported scores/FDR table in Supplementary Data 10 across the prediction-score range. The rationale is that decoy heterogeneity in similarity to targets is captured by the 1/SD weighting, while the pseudocount correction stabilizes FDR estimates where target counts are small.

## Related tools

- **METASPACE-ML** (Trained GBDT model that provides the prediction scores for target and SD-sampled decoy ions consumed by the FDR ranking procedure) — https://github.com/metaspace2020/metaspace

## Examples

```
df.sort_values('prediction_score', ascending=False).assign(dw=lambda d: d['is_decoy'] / d['SD']); fdr = (np.cumsum(df.dw) + 1) / (np.cumsum(~df.is_decoy) + 1)
```

## Evaluation signals

- Per-ion FDR values written to fdr_estimates.csv agree with the reported scores/FDR table in Supplementary Data 10 across the prediction-score range.
- Cumulative target count reaches the total number of target ions and cumulative weighted decoy count reaches the total decoy weight at the end of the ranking traversal.
- FDR is monotonically non-decreasing as prediction score descends through the merged ranking.
- Every decoy row carries the correct 1/SD weight and every target row carries zero decoy weight in the ranking.
- FDR values are continuous and bounded, with no NaN or negative entries after the Laplace correction.

## Limitations

- Decoy ions show heterogeneity in similarity to target ions; the article notes further investigations are needed into how this affects FDR estimation and in finding the most reliable way of decoy generation.
- FDR estimates inherit the model's training-data bias towards public METASPACE datasets (from 47 labs) and underrepresentation of less common data types such as single-cell metabolomics.
- Target-decoy separation is driven mainly by rho_spectral, with rho_chaos6 having only a minor impact on model performance.

## Evidence

- [abstract] METASPACE-ML is a machine learning-based approach addressing this challenge which incorporates new scores and computationally-efficient False Discovery Rate estimation.: "METASPACE-ML is a machine learning-based approach addressing this challenge which incorporates new scores and computationally-efficient False Discovery Rate estimation."
- [discussion] Decoy heterogeneity and generation: "Further investigations are needed into how this affects FDR estimation and in finding the most reliable way of decoy generation"
- [discussion] Spatial informativeness driven by rho_spectral: "rho_chaos6 had only a minor impact on the model performance (Fig. 6A), and target-decoy separation was mainly driven by rho_spectral"
- [readme] Repository documentation and project layout: "The METASPACE platform hosts an engine for
 metabolite annotation of imaging mass spectrometry data as well as a
 spatial metabolite knowledgebase of the metabolites from thousands of
 public datasets"
