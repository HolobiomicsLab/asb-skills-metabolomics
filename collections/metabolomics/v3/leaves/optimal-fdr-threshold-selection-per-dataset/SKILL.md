---
name: optimal-fdr-threshold-selection-per-dataset
description: Use when annotating imaging MS datasets with METASPACE or METASPACE-ML
  and you must decide which default FDR threshold (e.g. 5%) gives the most reliable
  target-decoy-controlled annotations for a specific dataset or sample-type group.
when_to_use_negative:
- Input is already a per-dataset table of reliability scores or a pre-selected optimal
  FDR threshold — recomputation via target-decoy F-beta is redundant.
- You need a new FDR model or rescoring of raw spectra rather than threshold selection
  among the existing default FDR thresholds; use METASPACE-ML annotation itself instead.
- The dataset lacks decoy annotation results, since precision/recall of confident
  annotations cannot be estimated from target-decoy separation without decoys.
edam_operation: http://edamontology.org/operation_2238
tools:
- name: METASPACE-ML
  role: Machine-learning model whose FDR-controlled, context-specific metabolite annotations
    provide the target/decoy results scored by the F-beta reliability metric
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE
  role: Platform hosting the metabolite annotation engine and the spatial metabolite
    knowledge base; per-dataset target and decoy annotation results are sourced from
    it (REST/GraphQL API, python-client)
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:a1cfd932b0be9ccf61ed6a3def7b0834b8023e800afd749ecacfbbdf206fef62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  edam_operation: http://edamontology.org/operation_2238
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# optimal-fdr-threshold-selection-per-dataset

## Summary

Quantifies the reliability of METASPACE-ML metabolite annotations in imaging mass spectrometry by computing a precision-weighted F-beta score (beta = 0.5) of target-decoy FDR estimates at the four default FDR thresholds, then selecting the per-dataset FDR threshold that maximizes this reliability score.

## When to use

Use when annotating imaging MS datasets with METASPACE or METASPACE-ML and you must decide which default FDR threshold (e.g. 5%) gives the most reliable target-decoy-controlled annotations for a specific dataset or sample-type group.

## When NOT to use

- Input is already a per-dataset table of reliability scores or a pre-selected optimal FDR threshold — recomputation via target-decoy F-beta is redundant.
- You need a new FDR model or rescoring of raw spectra rather than threshold selection among the existing default FDR thresholds; use METASPACE-ML annotation itself instead.
- The dataset lacks decoy annotation results, since precision/recall of confident annotations cannot be estimated from target-decoy separation without decoys.

## Inputs

- Per-dataset target annotation results from the METASPACE knowledge base (imaging mass spectrometry datasets)
- Per-dataset decoy annotation results used for target-decoy FDR estimation
- List of the four default FDR thresholds offered by METASPACE
- Sample-type group labels (e.g. animal testing datasets) for per-group aggregation

## Outputs

- reliability_scores.csv with per-dataset F-beta scores at each FDR threshold, the optimal (score-maximizing) threshold, and sample-type group
- Fig. 5C-style reliability-score distribution plot (PNG)
- Fig. 5D-style optimal-FDR breakdown plot (PNG)
- Aggregate metrics: fraction of datasets with optimal FDR at 5% and median reliability score per sample-type group

## How to apply

Load per-dataset target and decoy annotation results for the METASPACE knowledge base datasets (repository: github.com/metaspace2020/metaspace, documented in its GitHub Wiki; annotations are accessible via the GraphQL API or the python-client). At each of the four default FDR thresholds, compute precision and recall of confident metabolite annotations from the target-decoy estimates, then compute the F-beta score with beta = 0.5 (precision-weighted harmonic mean) as the per-dataset, per-threshold reliability_score. Select the FDR threshold that maximizes the F-beta score per dataset, then aggregate within a sample-type group (e.g. fraction of animal testing datasets whose optimum is 5%, and the median reliability score, reported as ~70% at 5% with median 0.88). Write reliability_scores.csv with per-dataset F-beta scores, optimal threshold, and sample-type group, and render reliability-score distribution and optimal-FDR breakdown plots (Fig. 5C/D style) as PNGs. Validate by checking the reproduced aggregates match the reported ~70% and median 0.88; flag the run as failed if either deviates substantially.

## Related tools

- **METASPACE-ML** (Machine-learning model whose FDR-controlled, context-specific metabolite annotations provide the target/decoy results scored by the F-beta reliability metric) — https://github.com/metaspace2020/metaspace
- **METASPACE** (Platform hosting the metabolite annotation engine and the spatial metabolite knowledge base; per-dataset target and decoy annotation results are sourced from it (REST/GraphQL API, python-client)) — https://github.com/metaspace2020/metaspace

## Examples

```
from metaspace.sm_annotation_utils import SMInstance; ann = SMInstance().get_annotations(fdr=0.05, dataset=ds_id)
```

## Evaluation signals

- Approximately 70% of animal testing datasets have their score-maximizing FDR threshold at 5% (reproducing Fig. 5D)
- Median F-beta reliability score across animal datasets is approximately 0.88 at the 5% threshold (reproducing Fig. 5C); flag the run as failed if either aggregate deviates substantially
- reliability_scores.csv contains one row per dataset with an F-beta score for each of the four default FDR thresholds, a single optimal threshold, and a sample-type group label
- Per-dataset F-beta values are bounded in [0, 1], since the score is a harmonic mean of precision and recall with beta = 0.5
- For every dataset, the selected optimal threshold is the argmax of that dataset's four computed F-beta scores

## Limitations

- Decoy ions show heterogeneity in similarity to target ions, and a reliable decoy generation method needs further investigation, which propagates uncertainty into FDR estimation and hence into the reliability score
- Optimal-threshold fractions and median scores are aggregate findings from the animal testing dataset group; they may not transfer to underrepresented sample types or technologies (e.g. QTOF datasets are underrepresented due to lack of publicly available data)
- Training data selection is biased towards public datasets represented in METASPACE (from 47 labs), so reliability estimates reflect this bias

## Evidence

- [methods] F-beta with beta=0.5 as reliability_score: "METASPACE-ML is a machine learning-based approach addressing this challenge which incorporates new scores and computationally-efficient False Discovery Rate estimation."
- [abstract] target-decoy FDR estimates from METASPACE knowledge base: "The METASPACE platform hosts an engine for metabolite annotation of imaging mass spectrometry data as well as a spatial metabolite knowledgebase"
- [discussion] decoy generation caveat: "Further investigations are needed into how this affects FDR estimation and in finding the most reliable way of decoy generation"
- [readme] source repository for per-dataset results and workflow docs: [evidence span withheld — anchor status: external_doc]
