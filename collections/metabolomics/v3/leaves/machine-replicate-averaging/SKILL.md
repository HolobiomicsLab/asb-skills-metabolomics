---
name: machine-replicate-averaging
description: Use when an LC/MS metabolomics feature table contains machine-replicate
  injections per sample and a consensus feature intensity table is needed before pathway/network
  analysis with mummichog.
when_to_use_negative:
- The feature table already contains one averaged value per sample (no machine-replicate
  injections remain to collapse).
- Features fail QC on their own merits (in-class CV > 0.2 or intensity < 10,000 per
  class); averaging must not be used to rescue unstable, low-intensity features.
- The data are not LC/MS spectral features (e.g. gene counts or MS/MS identification
  results) — mummichog and this QC pipeline operate on m/z feature tables.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mummichog
  role: Downstream consumer of the replicate-averaged, QC-filtered feature table;
    it needs the reference list of all detected m/z features plus a significant list
    for pathway/network analysis.
  repo: https://github.com/metabolomics-cloud/mummichog
- name: pandas
  role: Used to load the deposited feature table, apply intensity/CV thresholds, impute
    missing values to 1000, log2-transform, and average intensities across machine-replicate
    injections.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:bcd8cfbaa46ccdab95df67c4c3e22daff95ebf49f69ea0cd9981b637ad9da5ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metabolomics-cloud/mummichog
  tools:
  - mummichog
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# machine-replicate-averaging

## Summary

Collapses multiple machine-replicate LC/MS injections into a single consensus intensity per spectral feature, after QC filtering (intensity ≥ 10,000 per class, in-class CV ≤ 0.2), to build the reference feature list F_all that mummichog consumes.

## When to use

Use when an LC/MS metabolomics feature table contains machine-replicate injections per sample and a consensus feature intensity table is needed before pathway/network analysis with mummichog.

## When NOT to use

- The feature table already contains one averaged value per sample (no machine-replicate injections remain to collapse).
- Features fail QC on their own merits (in-class CV > 0.2 or intensity < 10,000 per class); averaging must not be used to rescue unstable, low-intensity features.
- The data are not LC/MS spectral features (e.g. gene counts or MS/MS identification results) — mummichog and this QC pipeline operate on m/z feature tables.

## Inputs

- Deposited LC/MS metabolomics feature table (e.g. Dataset S1, moDC experiment) with intensities for machine-replicate injections
- Sample-class metadata mapping injections to sample classes (for in-class CV and intensity-threshold filtering)

## Outputs

- QC-filtered, replicate-averaged feature intensity table (feature_table_qc.csv)
- Reference feature list F_all of all detected spectral features (7,995 features for the moDC data)

## How to apply

Load the deposited LC/MS feature table (e.g. Dataset S1 from the moDC YF17D activation experiment) with pandas, then apply the reported QC pipeline: retain features with measured intensity ≥ 10,000 in each sample class, impute missing values to 1000, and log2-transform intensities. Remove features whose in-class coefficient of variation exceeds 0.2, since features unstable across replicate injections would otherwise propagate measurement noise into the reference list. Average the remaining features' intensities across machine-replicate injections so each feature carries one value per biological sample. The rationale is that mummichog requires a reference list of all detected m/z features, so replicate dispersion must be controlled before features are counted as 'detected'. Verify that the resulting reference feature list F_all contains exactly 7,995 spectral features and save it as feature_table_qc.csv.

## Related tools

- **mummichog** (Downstream consumer of the replicate-averaged, QC-filtered feature table; it needs the reference list of all detected m/z features plus a significant list for pathway/network analysis.) — https://github.com/metabolomics-cloud/mummichog
- **pandas** (Used to load the deposited feature table, apply intensity/CV thresholds, impute missing values to 1000, log2-transform, and average intensities across machine-replicate injections.)

## Examples

```
df = pd.read_csv('Dataset_S1.csv', sep='\t'); qc = df[df.intensity >= 10000]; qc = qc.groupby('feature_id').mean(); qc.to_csv('feature_table_qc.csv')
```

## Evaluation signals

- The resulting reference feature list F_all contains exactly 7,995 spectral features for the moDC Dataset S1 input.
- No retained feature has an in-class coefficient of variation exceeding 0.2.
- Every retained feature has measured intensity ≥ 10,000 in each sample class (before log2 transformation), with missing values imputed to 1000.
- The averaged table has one intensity value per feature per sample, i.e. replicate injections no longer appear as separate columns/rows.
- The saved feature_table_qc.csv loads cleanly with pandas and its feature count matches the reported QC yield.

## Limitations

- The in-class CV ≤ 0.2 threshold is dataset-specific (moDC LC/MS); other experiments may require recalibrated thresholds.
- Imputing missing values to 1000 and log2-transforming are order-sensitive operations; applying them inconsistently changes the averaged intensities and the final feature count.
- Features discarded by QC cannot be recovered downstream — mummichog treats the reference list as all features detected in the MS experiment, so over-aggressive filtering shrinks F_all and can inflate pathway enrichment statistics.

## Evidence

- [results] The cell extracts from our activation experiment were analyzed by LC/MS metabolomics, and yielded 7,995 spectral features (denoted as ) after quality control.: "The cell extracts from our activation experiment were analyzed by LC/MS metabolomics, and yielded 7,995 spectral features (denoted as ) after quality control."
- [results] mummichog requires two lists of m/z features: the significant list and the reference list of all features detected.: "mummichog requires two lists of m/z features, the significant list  (e.g. selected by univariate statistics) and the reference list  (all features detected in the MS experiment)."
- [readme] README: mummichog predicts functional activity directly from feature tables, so replicate-averaged inputs matter.: "It leverages the organization of metabolic networks to predict functional activity directly from feature tables, bypassing metabolite identification."
