---
name: metabolomics-feature-quality-control
description: Use when starting from a raw LC/MS feature table (m/z, retention time,
  intensity per sample class) that must be QC-filtered before pathway or network activity
  analysis such as mummichog.
when_to_use_negative:
- Input is already a cleaned feature table with QC filtering applied — re-filtering
  would drop valid features and change F_all size.
- Data are tandem MS spectra intended for metabolite identification rather than an
  MS1 feature table for pathway/network analysis.
- The downstream tool does not require a reference feature list (e.g. direct metabolite
  annotation workflows).
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: mummichog
  role: Downstream consumer of the QC'd reference feature list; the skill's output
    defines F_all that mummichog requires alongside a significant feature list.
  repo: http://atcg.googlecode.com
- name: pandas
  role: Used to load and manipulate the LC/MS feature table (Dataset S1) during the
    QC filter chain.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:6fde24013b9cc886936a02b2e27bce5f31feab90205bd6f0c36bb423dbd188fe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - mummichog
  - pandas
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# metabolomics-feature-quality-control

## Summary

Quality-control an LC/MS untargeted metabolomics feature table (e.g. the 7,995-feature moDC Dataset S1) by filtering on intensity floor, class-level CV, and replicate averaging, producing a reference feature list F_all ready for mummichog pathway/network analysis.

## When to use

Use when starting from a raw LC/MS feature table (m/z, retention time, intensity per sample class) that must be QC-filtered before pathway or network activity analysis such as mummichog.

## When NOT to use

- Input is already a cleaned feature table with QC filtering applied — re-filtering would drop valid features and change F_all size.
- Data are tandem MS spectra intended for metabolite identification rather than an MS1 feature table for pathway/network analysis.
- The downstream tool does not require a reference feature list (e.g. direct metabolite annotation workflows).

## Inputs

- Raw LC/MS metabolomics feature table (Dataset S1 format: m/z, retention time, intensity per sample class, machine-replicate injections)
- Sample-class metadata defining which samples belong to each class (e.g. infected vs control moDC)

## Outputs

- QC-filtered feature table (feature_table_qc.csv) with log2-transformed, replicate-averaged intensities
- Reference feature list F_all of all detected spectral features (expected 7,995 for the moDC dataset)
- Significant feature list selected by univariate statistics (e.g. 601 differentially abundant features in moDC)

## How to apply

Load the deposited LC/MS feature table (Dataset S1) with pandas and apply the paper's QC chain: retain features whose measured intensity is at least 10,000 in each sample class, impute missing values to 1000, and log2-transform the intensities; then remove features whose in-class coefficient of variation exceeds 0.2 and average remaining intensities across machine-replicate injections. The result is the reference list F_all of all detected spectral features — the paper's moDC activation experiment yielded exactly 7,995 features after QC. This reference list is one of the two inputs mummichog requires (the other being the significant list selected by univariate statistics), so QC correctness directly feeds the downstream network-activity prediction.

## Related tools

- **mummichog** (Downstream consumer of the QC'd reference feature list; the skill's output defines F_all that mummichog requires alongside a significant feature list.) — http://atcg.googlecode.com
- **pandas** (Used to load and manipulate the LC/MS feature table (Dataset S1) during the QC filter chain.)

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('DatasetS1.csv'); df = df[df[df.columns[2:]].min(axis=1) >= 10000]; df.fillna(1000).to_csv('feature_table_qc.csv')"
```

## Evaluation signals

- The QC-filtered reference list F_all contains exactly 7,995 spectral features for the moDC activation dataset.
- No retained feature has an in-class coefficient of variation greater than 0.2.
- All retained features have measured intensity ≥ 10,000 in each sample class before log2 transformation.
- Missing values are imputed to 1000 prior to log2 transformation.
- The output is a single feature table (feature_table_qc.csv) that mummichog can consume as the reference feature list.

## Limitations

- The QC thresholds (intensity ≥ 10,000, CV ≤ 0.2) are tuned to the moDC LC/MS setup; other datasets may require re-derivation of cutoffs.
- Features from high-resolution profiling experiments far exceed current annotations in metabolite databases, so de facto filtering still occurs downstream.
- Quantification and feature selection upstream of mummichog can still be problematic even after QC.
- The current mummichog version does not incorporate a statistical matrix from the feature selection step.

## Evidence

- [results] The cell extracts from the moDC activation experiment, analyzed by LC/MS metabolomics, yielded 7,995 spectral features after quality control, defining the reference feature list F_all.: "The cell extracts from our activation experiment were analyzed by LC/MS metabolomics, and yielded 7,995 spectral features (denoted as ) after quality control."
- [results] mummichog requires a significant feature list and a reference list of all detected features, which is produced by this QC skill.: "mummichog requires two lists of m/z features, the significant list (e.g. selected by univariate statistics) and the reference list (all features detected in the MS experiment)."
- [discussion] Quantification and feature selection upstream of mummichog can still be problematic even after QC.: "Quantification and feature selection, the steps upstream of mummichog, can still be problematic."
- [discussion] In the moDC data, the significant feature set was chosen by a cutoff value during the feature selection step.: "In the moDC data, we chose by a cutoff value ."
- [intro] The workflow in the abstract describes the spectral peaks being extracted, quantified and aligned into a feature table, which is the raw input this QC skill operates on.: "the spectral peaks are extracted, quantified and aligned into a feature table"
