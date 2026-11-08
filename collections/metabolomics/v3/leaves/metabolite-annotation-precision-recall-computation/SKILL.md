---
name: metabolite-annotation-precision-recall-computation
description: 'Decision support for choosing the most reliable FDR threshold when analyzing
  METASPACE-ML annotation results: per-dataset target and decoy annotation tables
  are available and reliability must be compared across default FDR thresholds.'
when_to_use_negative:
- Input is raw imaging MS centroid or profile data that has not yet been annotated
  by METASPACE-ML — this skill evaluates existing annotations, not spectra
- Evaluating annotation quality against manually curated ground-truth databases rather
  than target-decoy FDR estimates
- Comparing METASPACE-ML against the rule-based MSM approach on precision/throughput
  rather than selecting a per-dataset FDR threshold
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: METASPACE-ML
  role: Machine-learning annotation model whose target/decoy results provide the precision
    and recall estimates scored here; uses computationally-efficient False Discovery
    Rate estimation
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE
  role: Engine and knowledge base supplying the public datasets and annotations database
    queried for per-dataset target/decoy results
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:e5c366ac185fd24596174ff96915c9646a3ca77fc474ba448902924c4218fcb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# metabolite-annotation-precision-recall-computation

## Summary

Quantify the reliability of METASPACE-ML metabolite annotation results per dataset by computing an F-beta (beta=0.5) reliability score from target-decoy precision and recall at each default FDR threshold, then selecting the threshold that maximizes it.

## When to use

Decision support for choosing the most reliable FDR threshold when analyzing METASPACE-ML annotation results: per-dataset target and decoy annotation tables are available and reliability must be compared across default FDR thresholds.

## When NOT to use

- Input is raw imaging MS centroid or profile data that has not yet been annotated by METASPACE-ML — this skill evaluates existing annotations, not spectra
- Evaluating annotation quality against manually curated ground-truth databases rather than target-decoy FDR estimates
- Comparing METASPACE-ML against the rule-based MSM approach on precision/throughput rather than selecting a per-dataset FDR threshold

## Inputs

- Per-dataset target annotation tables from METASPACE-ML (METASPACE knowledge base)
- Per-dataset decoy annotation tables for target-decoy FDR estimation
- The four default FDR thresholds used by METASPACE (including 5%)
- Sample-type labels to group datasets (e.g. animal testing datasets)

## Outputs

- reliability_scores.csv with per-dataset F-beta scores, optimal FDR threshold, and sample-type group
- Fig. 5C/D-style PNG plots: reliability-score distribution and optimal-FDR breakdown
- Aggregate statistics: fraction of animal datasets with optimal FDR = 5% and median reliability score

## How to apply

Load per-dataset target and decoy annotation results from the METASPACE knowledge base (accessed via the metaspace2020/metaspace platform and its Python client). For each dataset, compute precision and recall of confident metabolite annotations at each of the four default FDR thresholds using target-decoy FDR estimates, then compute the F-beta score with beta=0.5 (precision-weighted harmonic mean) at every threshold to obtain the reliability_score. Select, per dataset, the FDR threshold maximizing this score, then aggregate within sample-type groups — e.g. the fraction of animal testing datasets whose optimal FDR is 5% and the median reliability score. Write reliability_scores.csv and render Fig. 5C/D-style plots of reliability-score distribution and optimal-FDR breakdown; mark the run failed if aggregates deviate substantially from the reported ~70% at 5% FDR and median score 0.88.

## Related tools

- **METASPACE-ML** (Machine-learning annotation model whose target/decoy results provide the precision and recall estimates scored here; uses computationally-efficient False Discovery Rate estimation) — https://github.com/metaspace2020/metaspace
- **METASPACE** (Engine and knowledge base supplying the public datasets and annotations database queried for per-dataset target/decoy results) — https://github.com/metaspace2020/metaspace

## Examples

```
f_beta = {f: (1+0.5**2)*p[f]*r[f]/(0.5**2*p[f]+r[f]) for f in fdrs}; best_fdr = max(f_beta, key=f_beta.get)
```

## Evaluation signals

- Approximately 70% of animal testing datasets have an optimal FDR threshold at 5% (Fig. 5D)
- Median reliability score across animal datasets at the optimal threshold is 0.88 (Fig. 5C); flag the run as failed if either aggregate deviates substantially
- F-beta is computed with beta = 0.5 so precision is weighted over recall in the harmonic mean
- Optimal threshold selection is per dataset (argmax of F-beta over the four default FDR thresholds) before any aggregation
- Output CSV contains per-dataset F-beta scores plus the sample-type group column

## Limitations

- Decoy ions show heterogeneity in similarity to target ions, and the most reliable decoy generation method for FDR estimation needs further investigation
- QTOF-based datasets are underrepresented in the METASPACE-ML model, limiting generalization of reliability estimates for those instruments
- Training data selection is biased toward public datasets represented in METASPACE (from 47 labs), which may bias downstream reliability aggregates

## Evidence

- [abstract] METASPACE-ML FDR scoring context: "METASPACE-ML is a machine learning-based approach addressing this challenge which incorporates new scores and computationally-efficient False Discovery Rate estimation."
- [discussion] Decoy generation caveat: "Further investigations are needed into how this affects FDR estimation and in finding the most reliable way of decoy generation"
- [readme] METASPACE platform scope: "The METASPACE platform hosts an engine for metabolite annotation of imaging mass spectrometry data as well as a spatial metabolite knowledgebase"
- [intro] Low-FDR annotation claim: "METASPACE-ML outperforms the traditional MSM rule-based approach by delivering more annotations, especially at low FDR thresholds"
