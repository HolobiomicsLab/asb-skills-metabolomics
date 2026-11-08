---
name: median-statistics-summarization-across-datasets
description: Use when two annotation methods (e.g. METASPACE-ML vs MSM) have been
  run on many imaging MS datasets and you need per-group median annotation-count gains
  and log2 fold-changes at fixed FDR cutoffs (5% and 10%).
when_to_use_negative:
- You need per-annotation identity changes (which specific metabolites were gained/lost)
  rather than aggregate count statistics — MSM-vs-ML per-ion comparisons need a different
  skill.
- Only a single dataset is available, so per-dataset medians and group-level fold-changes
  are undefined or meaningless.
- The annotation tables lack FDR estimates, so confident-annotation filtering at 5%/10%
  FDR cannot be applied.
edam_operation: http://edamontology.org/operation_3435
tools:
- name: METASPACE-ML
  role: Machine-learning annotation method whose per-dataset confident-annotation
    counts (at 5% and 10% FDR) form the numerator of the median difference and LFC
    statistics
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE
  role: Rule-based MSM predecessor supplying the baseline per-dataset annotation counts
    against which METASPACE-ML gains are summarized
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:9379ab9d694d74253f2aa5f7208e16a18b673cb0476d05feba7eafc38be04708
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  edam_operation: http://edamontology.org/operation_3435
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# median-statistics-summarization-across-datasets

## Summary

Summarize per-dataset benchmarking deltas between METASPACE-ML and rule-based MSM annotation counts into median differences, median log2-fold-changes, and a consolidated CSV summary table, mirroring the METASPACE-ML (Nature Communications, 2024) evaluation.

## When to use

Use when two annotation methods (e.g. METASPACE-ML vs MSM) have been run on many imaging MS datasets and you need per-group median annotation-count gains and log2 fold-changes at fixed FDR cutoffs (5% and 10%).

## When NOT to use

- You need per-annotation identity changes (which specific metabolites were gained/lost) rather than aggregate count statistics — MSM-vs-ML per-ion comparisons need a different skill.
- Only a single dataset is available, so per-dataset medians and group-level fold-changes are undefined or meaningless.
- The annotation tables lack FDR estimates, so confident-annotation filtering at 5%/10% FDR cannot be applied.

## Inputs

- METASPACE-ML annotation result tables (per dataset) with FDR scores
- MSM (rule-based METASPACE) annotation result tables (per dataset) with FDR scores
- Dataset-to-group mapping (animal, plant; Orbitrap, FTICR instrument contexts)

## Outputs

- Median annotation-count differences (METASPACE-ML minus MSM) per testing group at 10% FDR
- Median log2 fold-changes (METASPACE-ML/MSM) per testing group at 5% FDR
- Median Orbitrap-vs-FTICR annotation-count difference at the reported FDR level
- annotation_comparison_stats.csv summary table

## How to apply

Load the METASPACE-ML and MSM annotation result tables for the testing datasets (animal, plant, Orbitrap, FTICR contexts) with pandas. For each dataset and each FDR threshold (5% and 10%), filter to confident annotations and count annotations per dataset for both methods. Compute per-dataset median differences (METASPACE-ML minus MSM) at 10% FDR for the animal and plant testing groups, and per-dataset median log2 fold-change (log2 of the METASPACE-ML/MSM count ratio) at 5% FDR for the same groups. Also compute the median difference in annotation counts between Orbitrap and FTICR datasets at the reported FDR level. Assemble the recomputed statistics into a summary table saved as annotation_comparison_stats.csv; acceptance requires reproducing the reported benchmark values (≈20 and ≈70 extra median annotations at 10% FDR for animal and plant, ≈1.64x and ≈1.80x median LFC at 5% FDR, ≈70 extra median annotations for Orbitrap vs FTICR).

## Related tools

- **METASPACE-ML** (Machine-learning annotation method whose per-dataset confident-annotation counts (at 5% and 10% FDR) form the numerator of the median difference and LFC statistics) — https://github.com/metaspace2020/metaspace
- **METASPACE** (Rule-based MSM predecessor supplying the baseline per-dataset annotation counts against which METASPACE-ML gains are summarized) — https://github.com/metaspace2020/metaspace

## Examples

```
import pandas as pd; annos = pd.read_csv('annotation_comparison_stats_input.csv'); counts = annos[annos.fdr<=0.10].groupby(['method','group','dataset']).size(); counts.to_csv('annotation_comparison_stats.csv')
```

## Evaluation signals

- Recomputed median difference at 10% FDR is approximately 20 annotations for animal and 70 for plant testing datasets versus MSM.
- At least 50 (animal) and 100 (plant) extra annotations are observed in ≥50% of datasets at 10% FDR.
- Recomputed median log2 fold-change at 5% FDR corresponds to approximately 1.64x (animal) and 1.80x (plant) annotation counts.
- Recomputed median Orbitrap-vs-FTICR annotation-count difference is approximately 70 annotations at the reported FDR level.
- Output file annotation_comparison_stats.csv is produced and contains one row per recomputed statistic with consistent column schema.

## Limitations

- MSM's sensitivity to its feature scores, particularly the rho_spatial score, causes some annotations to be captured by MSM but not METASPACE-ML, so METASPACE-ML/MSM count ratios can dip below 1 for some datasets and are best summarized by medians.
- Comparisons are anchored to METASPACE knowledge-base datasets and contexts (animal, plant, Orbitrap, FTICR); generalization to underrepresented contexts (e.g. QTOF) is not established.
- Medians and fold-changes summarize annotation counts only; they do not validate FDR calibration itself, which depends on target/decoy generation.

## Evidence

- [other] At the default FDR 10%, METASPACE-ML identified approximately 20 and 70 more annotations on average (and at least 50 and 100 more annotations in 50% of datasets) for animal and plant-based testing datasets: [evidence span withheld — anchor status: not_found]
- [intro] intro benchmark statement: "We demonstrate that METASPACE-ML outperforms the traditional MSM rule-based approach by delivering more annotations, especially at low FDR thresholds and for ions of low intensity."
- [abstract] abstract performance claim: "METASPACE-ML outperforms its rule-based predecessor, exhibiting higher precision, increased throughput, and enhanced capability in identifying low-intensity and biologically-relevant metabolites."
- [discussion] MSM rho_spatial caveat: "some annotations were captured by MSM but not by METASPACE-ML due to MSM’s sensitivity to its feature scores, particularly the rho_spatial score"
- [readme] python-client access: [evidence span withheld — anchor status: external_doc]
