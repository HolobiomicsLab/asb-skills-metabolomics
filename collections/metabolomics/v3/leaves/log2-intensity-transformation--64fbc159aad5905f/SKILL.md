---
name: log2-intensity-transformation
description: Use when a raw LC/MS metabolomics feature table (e.g. moDC Dataset S1)
  with missing values and untransformed intensities must be normalized before mummichog
  QC filtering and reference-list construction.
when_to_use_negative:
- The feature table is already log2-transformed and imputed — re-transforming would
  compound the log scale and break comparison with the published 7,995-feature reference
  list
- The goal is to build the significant feature list (e.g. the 601 features selected
  by a cutoff value in the moDC data) rather than the reference list of all detected
  features
- The data are not untargeted LC/MS feature intensities, e.g. tandem MS fragmentation
  data, which mummichog is explicitly not designed to replace for metabolite identification
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: pandas
  role: Loads and manipulates the LC/MS feature table (Dataset S1) when applying the
    impute-to-1000 and log2-transform steps
- name: mummichog
  role: 'Downstream consumer: the log2-transformed, QC-filtered table defines the
    reference feature list F_all required by mummichog''s pathway/network analysis'
  repo: https://github.com/shuzhao-li/mummichog/blob/master/mummichog/models.py
- name: LC/MS
  role: Instrument platform producing the raw spectral feature intensities that are
    imputed and log2-transformed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:3d819360c18cf65ec1c36bb65626a182701381302c7a9a37f682f7fffabb1e6c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LC/MS
  - mummichog
  - pandas
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# log2-intensity-transformation

## Summary

Prepares an LC/MS metabolomics feature table for mummichog pathway/network analysis by imputing missing intensities to 1000 and log2-transforming measured intensities, so that downstream QC filters (in-class CV <= 0.2, intensity >= 10,000 per sample class) operate on a stabilized scale and yield the reference feature list F_all.

## When to use

Use when a raw LC/MS metabolomics feature table (e.g. moDC Dataset S1) with missing values and untransformed intensities must be normalized before mummichog QC filtering and reference-list construction.

## When NOT to use

- The feature table is already log2-transformed and imputed — re-transforming would compound the log scale and break comparison with the published 7,995-feature reference list
- The goal is to build the significant feature list (e.g. the 601 features selected by a cutoff value in the moDC data) rather than the reference list of all detected features
- The data are not untargeted LC/MS feature intensities, e.g. tandem MS fragmentation data, which mummichog is explicitly not designed to replace for metabolite identification

## Inputs

- Raw LC/MS metabolomics feature table (e.g. moDC Dataset S1) with per-sample measured intensities and missing values
- Sample class labels used for the in-class intensity threshold and CV filter
- Machine-replicate injection grouping information for averaging

## Outputs

- Imputed, log2-transformed feature intensity table (feature_table_qc.csv)
- Reference feature list F_all of all detected spectral features (7,995 features for the moDC experiment)

## How to apply

Load the deposited LC/MS feature table (e.g. Dataset S1 from the moDC YF17D activation experiment) with pandas, then impute missing intensity values to 1000 before log2-transforming the measured intensities; this ordering matters because the imputed floor enters the transform on the linear scale. The article's moDC workflow applies this transformation as step 2 of QC, between the intensity threshold (measured intensity >= 10,000 in each sample class) and the in-class coefficient-of-variation filter (remove features with CV exceeding 0.2) and averaging across machine-replicate injections. The transformed, filtered table defines the reference feature list F_all of all detected spectral features that mummichog requires alongside the significant feature list. For the moDC cell extracts analyzed by LC/MS metabolomics, the resulting F_all contains exactly 7,995 spectral features, which serves as the ground-truth size check.

## Related tools

- **pandas** (Loads and manipulates the LC/MS feature table (Dataset S1) when applying the impute-to-1000 and log2-transform steps)
- **mummichog** (Downstream consumer: the log2-transformed, QC-filtered table defines the reference feature list F_all required by mummichog's pathway/network analysis) — https://github.com/shuzhao-li/mummichog/blob/master/mummichog/models.py
- **LC/MS** (Instrument platform producing the raw spectral feature intensities that are imputed and log2-transformed)

## Examples

```
df = pd.read_csv('Dataset_S1.csv'); df = df.fillna(1000); log2_int = np.log2(df)
```

## Evaluation signals

- After imputation, no NaN values remain in the intensity table and all imputed cells equal exactly 1000 (log2(1000) ≈ 9.97 after transformation)
- All post-transform values are finite and non-negative on the log2 scale
- The resulting reference feature list F_all contains exactly 7,995 spectral features for the moDC cell-extract dataset
- Only features with in-class coefficient of variation <= 0.2 survive the subsequent filter, applied to the transformed intensities
- Machine-replicate injections have been averaged into a single intensity per feature before the final table is saved

## Limitations

- The imputation value of 1000 is tied to this experiment's intensity scale (threshold of 10,000); reusing it for data with different dynamic ranges needs justification
- Quantification and feature selection, the steps upstream of mummichog that include this transformation, can still be problematic per the article's discussion
- The transformed table only supports reference-list construction; mummichog additionally needs a significant feature list (e.g. chosen by a cutoff value in the moDC data) as a second input

## Evidence

- [other] Apply the reported QC filters: retain features with measured intensity >= 10,000 in each sample class, impute missing values to 1000, and log2-transform the intensities.: [evidence span withheld — anchor status: not_found]
- [results] The cell extracts from our activation experiment were analyzed by LC/MS metabolomics, and yielded 7,995 spectral features (denoted as ) after quality control.: "The cell extracts from our activation experiment were analyzed by LC/MS metabolomics, and yielded 7,995 spectral features"
- [results] mummichog requires two lists of m/z features, the significant list and the reference list (all features detected in the MS experiment).: "mummichog requires two lists of m/z features, the significant list"
- [discussion] Quantification and feature selection, the steps upstream of mummichog, can still be problematic.: "Quantification and feature selection, the steps upstream of mummichog, can still be problematic."
