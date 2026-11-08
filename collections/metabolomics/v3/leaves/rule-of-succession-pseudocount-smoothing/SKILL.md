---
name: rule-of-succession-pseudocount-smoothing
description: Use when you must convert a ranked list of target/decoy annotation scores
  (with per-decoy weights) into continuous, smoothed FDR confidence values for individual
  ions, e.g. estimating METASPACE-ML FDR from target-decoy rankings.
when_to_use_negative:
- Input already contains per-annotation FDR or q-values (e.g. the reported scores/FDR
  table from Supplementary Data 10) — re-estimation is unnecessary
- Scores come from the rule-based MSM (Metabolite Signal Matching) scoring without
  an SD-sampled target-decoy ranking, since the weighting scheme requires per-decoy
  SD values
- The analysis goal is model training or feature engineering rather than converting
  an existing ranked target-decoy score list into FDR confidence values
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: METASPACE-ML
  role: Produces the prediction scores for target and SD-sampled decoy ions that feed
    the weighted target-decoy FDR estimation
  repo: https://github.com/metaspace2020/metaspace
- name: pandas / numpy
  role: Used to build, merge, sort, and cumulatively traverse the target-decoy ranking
    and apply the pseudocount correction
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:b2f8872ee7bee10d3de6d529db3aa99fb03b1b9a47015330113c1796da585442
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE-ML
  - pandas / numpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# rule-of-succession-pseudocount-smoothing

## Summary

A Laplace rule-of-succession pseudocount correction applied to a weighted target-decoy cumulative ranking to produce smoothed, continuous per-annotation FDR values, as used by METASPACE-ML for computationally-efficient False Discovery Rate estimation in imaging mass spectrometry metabolite annotation.

## When to use

Use when you must convert a ranked list of target/decoy annotation scores (with per-decoy weights) into continuous, smoothed FDR confidence values for individual ions, e.g. estimating METASPACE-ML FDR from target-decoy rankings.

## When NOT to use

- Input already contains per-annotation FDR or q-values (e.g. the reported scores/FDR table from Supplementary Data 10) — re-estimation is unnecessary
- Scores come from the rule-based MSM (Metabolite Signal Matching) scoring without an SD-sampled target-decoy ranking, since the weighting scheme requires per-decoy SD values
- The analysis goal is model training or feature engineering rather than converting an existing ranked target-decoy score list into FDR confidence values

## Inputs

- Table of METASPACE-ML prediction scores for target ions (from the scores table)
- Table of prediction scores for SD-sampled decoy ions, including per-decoy standard deviation (SD) of sampled similarities used to derive 1/SD weights

## Outputs

- fdr_estimates.csv containing per-ion FDR values, prediction scores, and target/decoy labels
- Validation report comparing computed FDR values against the Supplementary Data 10 scores/FDR table

## How to apply

Load METASPACE-ML prediction scores for target ions and for SD-sampled decoy ions into a pandas DataFrame from the scores table. Merge targets and decoys into a single ranking sorted in descending order by prediction score, assigning each decoy ion a weight of 1/SD, where SD is the standard deviation of that decoy's sampled similarities. Traverse the ranking cumulatively with numpy, summing weighted decoy counts and target counts at each position, then apply the Laplace rule-of-succession correction: FDR = (cumulative_decoy_weight + 1) / (cumulative_target_count + 2), which smooths the raw decoy/target ratio and avoids zero FDR at the top of the ranking. The decision point is validation: compare computed per-ion FDR values against the reported scores/FDR table in Supplementary Data 10 and record agreement across the prediction-score range in a validation report.

## Related tools

- **METASPACE-ML** (Produces the prediction scores for target and SD-sampled decoy ions that feed the weighted target-decoy FDR estimation) — https://github.com/metaspace2020/metaspace
- **pandas / numpy** (Used to build, merge, sort, and cumulatively traverse the target-decoy ranking and apply the pseudocount correction)

## Examples

```
import pandas as pd, numpy as np; df = pd.read_csv('scores_table.csv').sort_values('score', ascending=False); w = np.where(df.label == 'decoy', 1 / df.sd, 0); df['fdr'] = (np.cumsum(w) + 1) / (np.cumsum(df.label == 'target') + 2); df.to_csv('fdr_estimates.csv')
```

## Evaluation signals

- Per-ion FDR values from fdr_estimates.csv agree with the reported scores/FDR table in Supplementary Data 10 across the prediction-score range
- FDR is monotonically non-decreasing as prediction score decreases in the merged target-decoy ranking
- All FDR values lie in the interval (0, 1]; no FDR of exactly 0 due to the +1/+2 pseudocounts
- Cumulative decoy sums reflect 1/SD weights (not raw decoy counts), so decoys with high SD contribute less to cumulative FDR
- Target/decoy labels in the output match the labels in the source scores table after merging

## Limitations

- Decoy ions show heterogeneity in similarity to target ions, and the most reliable decoy generation method still needs further investigation, affecting FDR estimation accuracy
- The method inherits biases of the METASPACE-ML training data, which is selected from public datasets in the METASPACE knowledge base (from 47 labs)
- Underrepresented data types (e.g. QTOF-based datasets, single-cell metabolomics) may yield less reliable target-decoy separations and thus less trustworthy FDR values

## Evidence

- [abstract] METASPACE-ML incorporates computationally-efficient FDR estimation: "METASPACE-ML is a machine learning-based approach addressing this challenge which incorporates new scores and computationally-efficient False Discovery Rate estimation."
- [discussion] Decoy generation reliability affects FDR estimation: "Further investigations are needed into how this affects FDR estimation and in finding the most reliable way of decoy generation"
- [readme] METASPACE hosts the metabolite annotation engine: "The METASPACE platform hosts an engine for metabolite annotation of imaging mass spectrometry data as well as a spatial metabolite knowledgebase"
