---
name: in-class-covariance-filtering
description: Use when preparing an untargeted LC/MS feature table (e.g. Dataset S1
  from a moDC activation experiment) as the reference feature list F_all before mummichog
  pathway/network analysis.
when_to_use_negative:
- The input is already a filtered, QC-passed feature table or a plain list of significant
  m/z features rather than a raw per-sample feature table
- The goal is mummichog pathway scoring or module analysis itself, not upstream feature-table
  QC
- The data are not replicate-measured LC/MS features with defined sample classes,
  so an in-class CV cannot be computed
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mummichog
  role: 'Downstream consumer: the QC-filtered reference feature list F_all is one
    of the two m/z feature lists mummichog requires, together with the significant
    list'
  repo: https://github.com/metabolomics-cloud/mummichog
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:5bdeaba7db5f7c8fc68e8d89f18ba6853d947848759c28432292771aef799491
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metabolomics-cloud/mummichog
  tools:
  - mummichog
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# in-class-covariance-filtering

## Summary

QC filtering of an LC/MS metabolomics feature table by removing spectral features whose in-class coefficient of variation exceeds 0.2, combined with intensity and replicate-averaging steps, to define a reproducible reference feature list (F_all) for mummichog analysis.

## When to use

Use when preparing an untargeted LC/MS feature table (e.g. Dataset S1 from a moDC activation experiment) as the reference feature list F_all before mummichog pathway/network analysis.

## When NOT to use

- The input is already a filtered, QC-passed feature table or a plain list of significant m/z features rather than a raw per-sample feature table
- The goal is mummichog pathway scoring or module analysis itself, not upstream feature-table QC
- The data are not replicate-measured LC/MS features with defined sample classes, so an in-class CV cannot be computed

## Inputs

- QC-annotated LC/MS metabolomics feature table (e.g. Dataset S1 from the moDC YF17D activation experiment), with intensities per sample, sample-class labels, and machine-replicate injection identifiers

## Outputs

- Filtered reference feature list F_all of spectral features surviving QC (7,995 features for the moDC experiment), saved as e.g. feature_table_qc.csv

## How to apply

Load the deposited feature table (Dataset S1) with pandas and apply the reported QC sequence: retain features with measured intensity >= 10,000 within each sample class, impute missing values to 1000, and log2-transform intensities. Then remove any feature whose in-class coefficient of variation exceeds 0.2, and average intensities across machine-replicate injections so each feature is represented once. The rationale is that high intra-class variability indicates unreliable quantification, and mummichog requires a stable reference list of all detected m/z features alongside the significant list. Save the filtered table (e.g. feature_table_qc.csv) and verify the feature count matches the reported result (7,995 spectral features for the moDC/YF17D experiment) before feeding it downstream.

## Related tools

- **mummichog** (Downstream consumer: the QC-filtered reference feature list F_all is one of the two m/z feature lists mummichog requires, together with the significant list) — https://github.com/metabolomics-cloud/mummichog

## Examples

```
import pandas as pd; ft = pd.read_csv('Dataset_S1.csv'); ft = ft[ft.groupby('class')['intensity'].transform('mean') >= 10000]; keep = ft.groupby('class')['intensity'].agg(lambda x: x.std()/x.mean()) <= 0.2; ft[keep].to_csv('feature_table_qc.csv')
```

## Evaluation signals

- The resulting reference list F_all contains exactly 7,995 spectral features for the moDC/YF17D dataset
- No retained feature has an in-class coefficient of variation above 0.2
- All retained features show measured intensity >= 10,000 in each sample class, missing values were imputed to 1000, and intensities are log2-transformed
- Machine-replicate injections are averaged so each feature appears once per biological condition
- The saved output (e.g. feature_table_qc.csv) parses cleanly with pandas and aligns with the significant-feature list submitted to mummichog

## Limitations

- Quantification and feature selection upstream of mummichog can still be problematic; a CV <= 0.2 filter reduces but does not eliminate unreliable features
- Features far exceed current metabolite database annotations, leading to a de facto filtering during downstream mummichog analysis regardless of QC stringency
- A single m/z feature may match several metabolites in the same pathway, so feature-level QC does not resolve downstream ambiguity

## Evidence

- [results] The cell extracts from our activation experiment were analyzed by LC/MS metabolomics, and yielded 7,995 spectral features (denoted as ) after quality control.: "The cell extracts from our activation experiment were analyzed by LC/MS metabolomics, and yielded 7,995 spectral features (denoted as ) after quality control."
- [results] mummichog requires two lists of m/z features, the significant list and the reference list of all features detected.: "mummichog requires two lists of m/z features, the significant list  (e.g. selected by univariate statistics) and the reference list  (all features detected in the MS experiment)."
- [discussion] In the moDC data, we chose by a cutoff value.: "In the moDC data, we chose  by a cutoff value ."
- [supplementary] This file contains LC/MS Metabolomics feature table from the moDC experiment: "Metabolomics feature table and mummichog predictions. This file contains LC/MS Metabolomics feature table from the moDC experiment"
- [readme] User supplied features with m/z, rtime, p-value from a statistical test.: "1. User supplied features with m/z, rtime, p-value from a statistical test. If no unique feature ID is supplied, row_number will be used as ID."
