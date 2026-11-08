---
name: untargeted-metabolomics-feature-extraction
description: Use when raw LC/MS mzXML files from untargeted metabolomics need peak
  picking, quantification and alignment into a feature table prior to statistical
  feature selection and pathway/network analysis.
when_to_use_negative:
- Input is already a processed feature table — go straight to mummichog input instead.
- The goal is metabolite identification by MS/MS fragmentation; the article states
  mummichog 'is not designed to replace tandem mass spectrometry in metabolite identification'.
- Targeted quantification with authentic standards is required rather than untargeted
  profiling.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MAVEN
  role: Untargeted LC/MS peak picking, quantification and alignment on mzXML files
    to build the feature table
- name: mummichog
  role: Downstream consumer of the feature table — takes m/z, rtime, p-value rows
    for pathway/network analysis
  repo: https://github.com/metabolomics-cloud/mummichog
- name: JMS
  role: Optional annotation of features via metDataModel structures before or after
    table generation
  repo: https://github.com/metabolomics-cloud/JMS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:af0e7dd158e232edef41e0cf8f70a89faaf25ad6b2538d637c873ed9f513881f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metabolomics-cloud/mummichog
  tools:
  - Azimuth DB
  - JMS
  - MAVEN
  - metDataModel
  - mummichog
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.65
n_claims: 20
---

# untargeted-metabolomics-feature-extraction

## Summary

Extract, quantify and align ionized LC/MS peaks from raw untargeted metabolomics runs into a spectral feature table (m/z, retention time, intensity), the mandatory upstream step before mummichog's pathway/network activity prediction.

## When to use

Use when raw LC/MS mzXML files from untargeted metabolomics need peak picking, quantification and alignment into a feature table prior to statistical feature selection and pathway/network analysis.

## When NOT to use

- Input is already a processed feature table — go straight to mummichog input instead.
- The goal is metabolite identification by MS/MS fragmentation; the article states mummichog 'is not designed to replace tandem mass spectrometry in metabolite identification'.
- Targeted quantification with authentic standards is required rather than untargeted profiling.

## Inputs

- Raw LC/MS untargeted metabolomics runs (mzXML files)
- Processed peak/feature table with m/z, retention time, intensity and p-value (e.g. inurons_ttest_1127.tsv)

## Outputs

- Aligned spectral feature table (7,995 features for moDC data; 5,707 for yeast data)
- Significant-feature subset selected by univariate statistics (e.g. 601 of 7,995 for moDC; 426 of 5,707 for yeast)

## How to apply

Run untargeted peak extraction on deposited mzXML files (e.g. with MAVEN) to pick peaks, quantify heights, and align them across samples into a single feature table — the paper's moDC experiment yielded 7,995 spectral features after quality control, and the yeast study yielded 5,707. Apply a QC step to remove noisy features, then perform univariate testing (e.g. t-test between groups) to select the significant feature list that mummichog requires alongside the reference list of all detected features. Retain m/z, retention time and a per-feature statistical value (p-value), since mummichog input rows are 'features with m/z, rtime, p-value'. Note that the number of features from high-resolution profiling far exceeds metabolite database annotations, so a de facto filtering occurs downstream in mummichog; upstream quantification and feature selection remain the problematic steps the paper flags. Export as a flat table (TSV) with one row per feature; the README test file inurons_ttest_1127.tsv is the expected shape.

## Related tools

- **MAVEN** (Untargeted LC/MS peak picking, quantification and alignment on mzXML files to build the feature table)
- **mummichog** (Downstream consumer of the feature table — takes m/z, rtime, p-value rows for pathway/network analysis) — https://github.com/metabolomics-cloud/mummichog
- **JMS** (Optional annotation of features via metDataModel structures before or after table generation) — https://github.com/metabolomics-cloud/JMS

## Examples

```
python3 -m mummichog.main -i tests/ineurons_ttest_1127.tsv -j testneuron -a tests/empCpds_with_annotations.json -d .
```

## Evaluation signals

- Feature table row count matches expected scale (e.g. 7,995 for the moDC LC/MS data; 5,707 for the yeast study)
- Every row carries m/z, retention time and a p-value column as mummichog expects
- Significant-feature subset size is consistent with the chosen cutoff (e.g. 601/7,995 or 426/5,707)
- Alignment across samples yields a single table, not per-file peaks
- Downstream mummichog run accepts the table without re-parsing (row_number usable as feature ID)

## Limitations

- Quantification and feature selection, the steps upstream of mummichog, can still be problematic
- Features from high-resolution profiling far exceed metabolite database annotations, so many extracted features have no metabolite match
- A de facto filtering occurs when data are run on mummichog because of limited database annotations
- Choosing significant features depends on a cutoff value; no statistical matrix is currently incorporated into mummichog's analysis

## Evidence

- [results] The cell extracts from our activation experiment were analyzed by LC/MS metabolomics, and yielded 7,995 spectral features (denoted as ) after quality control.: "The cell extracts from our activation experiment were analyzed by LC/MS metabolomics, and yielded 7,995 spectral features (denoted as ) after quality control."
- [readme] User supplied features with m/z, rtime, p-value from a statistical test.: "User supplied features with m/z, rtime, p-value from a statistical test. If no unique feature ID is supplied, row_number will be used as ID."
- [discussion] Quantification and feature selection, the steps upstream of mummichog, can still be problematic.: "Quantification and feature selection, the steps upstream of mummichog, can still be problematic."
- [intro] the spectral peaks are extracted, quantified and aligned into a feature table: "the spectral peaks are extracted, quantified and aligned into a feature table"
