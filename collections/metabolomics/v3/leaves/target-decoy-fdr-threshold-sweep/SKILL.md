---
name: target-decoy-fdr-threshold-sweep
description: Use when you have per-dataset METASPACE/METASPACE-ML target and decoy
  annotation results and need to decide which of the default FDR thresholds (e.g.
  5%, 10%, 20%, 50%) is most reliable for a sample-type group such as animal testing
  datasets.
when_to_use_negative:
- When no target-decoy annotation results are available — e.g. you are still designing
  the decoy generation method, which the article flags as needing further investigation
  because decoy ions show heterogeneity in similarity to target ions
- When the goal is to train or tune the METASPACE-ML GBDT ranking model itself rather
  than to evaluate annotation reliability at FDR thresholds
- When the data are not imaging-MS annotation results from METASPACE — the article
  notes the training/evaluation selection is biased towards public datasets represented
  in METASPACE, so thresholds may not transfer to other pipelines
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
tools:
- name: METASPACE-ML
  role: Produces the context-specific, FDR-controlled metabolite annotations whose
    reliability is quantified by the F-beta threshold sweep
- name: METASPACE
  role: Source platform/engine hosting the annotations database and knowledge base
    from which per-dataset target and decoy results are retrieved
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:24008187c616b596640709591b54fa2b43d38fb84bcf3f6b45f619ee6b73819b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# target-decoy-fdr-threshold-sweep

## Summary

A reliability-quantification procedure for METASPACE-ML imaging-MS metabolite annotations: precision and recall are computed from target-decoy estimates at each of the four default FDR thresholds, combined into a precision-weighted F-beta score (beta = 0.5), and the FDR threshold maximizing the reliability score is selected per dataset.

## When to use

Use when you have per-dataset METASPACE/METASPACE-ML target and decoy annotation results and need to decide which of the default FDR thresholds (e.g. 5%, 10%, 20%, 50%) is most reliable for a sample-type group such as animal testing datasets.

## When NOT to use

- When no target-decoy annotation results are available — e.g. you are still designing the decoy generation method, which the article flags as needing further investigation because decoy ions show heterogeneity in similarity to target ions
- When the goal is to train or tune the METASPACE-ML GBDT ranking model itself rather than to evaluate annotation reliability at FDR thresholds
- When the data are not imaging-MS annotation results from METASPACE — the article notes the training/evaluation selection is biased towards public datasets represented in METASPACE, so thresholds may not transfer to other pipelines

## Inputs

- Per-dataset target annotation results from METASPACE knowledge base datasets (imaging mass spectrometry)
- Per-dataset decoy annotation results (target-decoy FDR estimates) for the same datasets
- Sample-type group labels (e.g. animal vs. other tissue/dataset groups)

## Outputs

- reliability_scores.csv: per-dataset F-beta (beta=0.5) reliability scores at each default FDR threshold, optimal threshold, and sample-type group
- Fig. 5C/D-style PNG plots: reliability-score distribution per FDR threshold and optimal-FDR breakdown per group
- Group-level aggregates: fraction of datasets with optimal FDR = 5% and median reliability score

## How to apply

Load per-dataset target and decoy annotation results from the METASPACE knowledge base (platform code at github.com/metaspace2020/metaspace; the repository's python-client exposes the annotations database for programmatic retrieval). For each dataset, compute precision and recall of confident metabolite annotations at each of the four default FDR thresholds using the target-decoy estimates. At each threshold compute the F-beta score with beta = 0.5 (a precision-weighted harmonic mean, so false positive control is favored over recall — appropriate because METASPACE-ML's value proposition is 'more annotations, especially at low FDR thresholds'). Select, per dataset, the FDR threshold maximizing this reliability_score, then aggregate over the sample-type group (e.g. fraction of animal datasets whose optimal FDR is 5% and the median reliability score across datasets). Write reliability_scores.csv with per-dataset F-beta scores, optimal threshold, and sample-type group, and render Fig. 5C/D-style PNG plots showing the reliability-score distribution and optimal-FDR breakdown.

## Related tools

- **METASPACE-ML** (Produces the context-specific, FDR-controlled metabolite annotations whose reliability is quantified by the F-beta threshold sweep)
- **METASPACE** (Source platform/engine hosting the annotations database and knowledge base from which per-dataset target and decoy results are retrieved) — https://github.com/metaspace2020/metaspace

## Examples

```
from sklearn.metrics import fbeta_score; reliability = fbeta_score(y_true, y_confident_at_fdr, beta=0.5)  # per dataset, at each default FDR threshold
```

## Evaluation signals

- Reproduced aggregate over animal testing datasets: approximately 70% of datasets have their optimal FDR threshold at 5%; flag the run as failed if this deviates substantially
- Reproduced median reliability score across animal testing datasets ≈ 0.88 (Fig. 5C, D); substantial deviation indicates a failed run
- Schema check: reliability_scores.csv contains one row per dataset per default FDR threshold with columns for F-beta score, optimal threshold, and sample-type group
- Invariant: for every dataset, the selected optimal threshold is the argmax of the F-beta (beta=0.5) scores across the four default FDR thresholds
- Invariant: F-beta values lie in [0, 1] and are monotone-consistent with the underlying per-threshold precision/recall pairs

## Limitations

- Decoy ions show heterogeneity in similarity to target ions, so target-decoy FDR estimation quality varies and 'reliable decoy generation' remains an open issue
- QTOF-based datasets are underrepresented due to lack of publicly available data needed for statistically reliable evaluation
- Dataset selection is biased towards public datasets represented in METASPACE (from 47 labs), so aggregates may not generalize to private or non-METASPACE data
- Less common data types (e.g. single-cell metabolomics from cultured cells, industrial lab data) are underrepresented

## Evidence

- [abstract] METASPACE-ML is a machine learning-based approach addressing this challenge which incorporates new scores and computationally-efficient False Discovery Rate estimation.: "METASPACE-ML is a machine learning-based approach addressing this challenge which incorporates new scores and computationally-efficient False Discovery Rate estimation."
- [intro] More annotations at low FDR thresholds: "We demonstrate that METASPACE-ML outperforms the traditional MSM rule-based approach by delivering more annotations, especially at low FDR thresholds and for ions of low intensity."
- [discussion] Decoy generation reliability caveat: "Further investigations are needed into how this affects FDR estimation and in finding the most reliable way of decoy generation"
- [readme] METASPACE platform hosts annotation engine and knowledgebase: "hosts an engine for metabolite annotation of imaging mass spectrometry data as well as a spatial metabolite knowledgebase of the metabolites from thousands of public datasets provided by the community"
- [methods] Documentation pointer for accessing METASPACE annotations: "Documentation lives in the [GitHub Wiki](https://github.com/metaspace2020/metaspace/wiki)"
