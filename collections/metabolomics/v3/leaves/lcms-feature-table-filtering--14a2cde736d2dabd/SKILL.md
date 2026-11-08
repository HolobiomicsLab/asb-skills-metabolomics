---
name: lcms-feature-table-filtering
description: Use when an LC/MS untargeted metabolomics feature table must be turned
  into the two input lists (significant m/z features and reference list of all detected
  features) that mummichog requires for pathway/network activity prediction.
when_to_use_negative:
- Input is already a cleaned feature table or annotation file (e.g. JMS empirical-compound
  JSON); the skill targets raw feature-table QC and significance filtering
- Data are tandem-MS spectra intended for metabolite identification; the article states
  mummichog is not designed to replace tandem mass spectrometry in metabolite identification
- The analysis goal is targeted quantification of known metabolites rather than untargeted
  pathway/network activity prediction
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: mummichog
  role: Consumes the filtered significant and reference feature lists to predict metabolic
    pathway and network activity from spectral feature tables without a priori metabolite
    identification
  repo: https://github.com/metabolomics-cloud/mummichog
- name: JMS
  role: Optional companion library used to perform annotation on a dataset and structure
    the feature/compound data before mummichog analysis
  repo: https://github.com/metabolomics-cloud/JMS
- name: LC/MS
  role: Instrument platform producing the raw spectral feature table (e.g. moDC activation
    experiment yielding 7,995 features after QC) that this skill filters
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:8966cab167315a6ed6c9ebcb0325c769024f8f34222b27950987c0df2a279004
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metabolomics-cloud/mummichog
  tools:
  - Azimuth DB
  - JMS
  - LC/MS
  - mummichog
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# LC/MS feature-table filtering

## Summary

Prepares the m/z feature tables (significant list and reference list) that mummichog consumes by applying QC-driven intensity, CV, imputation, and significance filters to LC/MS metabolomics data from untargeted profiling experiments.

## When to use

Use when an LC/MS untargeted metabolomics feature table must be turned into the two input lists (significant m/z features and reference list of all detected features) that mummichog requires for pathway/network activity prediction.

## When NOT to use

- Input is already a cleaned feature table or annotation file (e.g. JMS empirical-compound JSON); the skill targets raw feature-table QC and significance filtering
- Data are tandem-MS spectra intended for metabolite identification; the article states mummichog is not designed to replace tandem mass spectrometry in metabolite identification
- The analysis goal is targeted quantification of known metabolites rather than untargeted pathway/network activity prediction

## Inputs

- LC/MS feature table (e.g. Dataset S1 CSV with m/z, retention time, and per-sample intensities)
- Significant-list m/z features (e.g. 601 moDC features or 524 urine gender-difference features)
- Reference list of all detected spectral features (e.g. 7,995 moDC features or 11,086 urine features)
- Optional annotation table on the features (e.g. JMS empirical-compound JSON)

## Outputs

- Filtered reference feature list F_all (e.g. feature_table_qc.csv with 7,995 moDC features)
- Significant m/z feature list for pathway activity prediction (e.g. the 524 urine features)
- Feature table formatted for mummichog input (m/z, rtime, p-value)

## How to apply

Load the LC/MS feature table (e.g. the moDC Dataset S1 containing 7,995 spectral features, or the MetaboLights 439020 positive-ion table with 11,086 features after filtering to intensity >100,000 in at least 5 samples). Apply the reported QC filters: retain features with measured intensity above the study cutoff (e.g. ≥10,000 per sample class for the moDC data), impute missing values to 1000, log2-transform intensities, remove features whose in-class coefficient of variation exceeds 0.2, and average intensities across machine-replicate injections. Derive the significant list by a univariate statistic cutoff (e.g. the 601 moDC features significantly different between infected and control samples, or the 524 gender-difference features from the urine study where testosterone glucuronide is low in females and high in males). Supply mummichog with both the significant list and the full reference list F_all, since mummichog computes all possibly matched metabolites and searches the reference metabolic network for modules formed by these tentative metabolites.

## Related tools

- **mummichog** (Consumes the filtered significant and reference feature lists to predict metabolic pathway and network activity from spectral feature tables without a priori metabolite identification) — https://github.com/metabolomics-cloud/mummichog
- **JMS** (Optional companion library used to perform annotation on a dataset and structure the feature/compound data before mummichog analysis) — https://github.com/metabolomics-cloud/JMS
- **LC/MS** (Instrument platform producing the raw spectral feature table (e.g. moDC activation experiment yielding 7,995 features after QC) that this skill filters)

## Examples

```
python3 -m mummichog.main -f mummichog/tests/testdata0710.txt -o t3
```

## Evaluation signals

- Verify the reference feature list F_all contains exactly the expected feature count (e.g. 7,995 for the moDC Dataset S1, 11,086 for the MetaboLights 439020 positive-ion table)
- Check that in-class coefficient of variation for retained features does not exceed 0.2
- Confirm that missing values were imputed to 1000 and intensities are log2-transformed before any downstream statistics
- Confirm the significant feature list size matches the reported figure (e.g. 601 moDC features, 524 urine gender-difference features)
- Run mummichog on the filtered lists and verify the predicted activity network reproduces the published result (e.g. 45 metabolites, 97% agreement with original annotation)

## Limitations

- A single m/z feature may match to several metabolites in the same module/pathway, inflating the overlap number
- Features from a high-resolution profiling experiment by far exceed the current annotations in metabolite databases, leading to a de facto filtering when data are run on mummichog
- Quantification and feature selection, the steps upstream of mummichog, can still be problematic
- Current mummichog version does not incorporate a statistical matrix from the feature selection step prior to mummichog's analysis
- Compound chemical formulas may be missing in a genome-scale metabolic model, requiring lookup via compound identifiers or a translation module before adduct matching

## Evidence

- [results] mummichog requires two lists of m/z features, the significant list (e.g. selected by univariate statistics) and the reference list (all features detected in the MS experiment).: "mummichog requires two lists of m/z features, the significant list (e.g. selected by univariate statistics) and the reference list (all features detected in the MS experiment)."
- [results] The cell extracts from our activation experiment were analyzed by LC/MS metabolomics, and yielded 7,995 spectral features (denoted as ) after quality control.: "The cell extracts from our activation experiment were analyzed by LC/MS metabolomics, and yielded 7,995 spectral features (denoted as ) after quality control."
- [other] Features with measured intensity >= 10,000 in each sample class, impute missing values to 1000, and log2-transform the intensities. Remove features whose in-class coefficient of variation exceeds 0.2 and average intensities across machine-replicate injections.: [evidence span withheld — anchor status: not_found]
- [readme] User supplied features with m/z, rtime, p-value from a statistical test.: "User supplied features with m/z, rtime, p-value from a statistical test. If no unique feature ID is supplied, row_number will be used as ID."
- [readme] We need chemical formulas of the compounds, which may not be available in a GSMM.: "We need chemical formulas of the compounds, which may not be available in a GSMM."
