---
name: random-forest-classifier-training
description: Use when a quality-controlled LC-MS feature table with sample metadata
  (e.g. age group, sex, brain region) is available and the goal is metabolome-informed
  phenotype prediction via a random forest with feature selection and ROC-AUC / cross-validation
  evaluation.
when_to_use_negative:
- Input is raw LC-MS data (mzML or Bruker .d) that has not yet been processed to an
  aligned, quality-controlled feature table — run MassCube's peak detection and alignment
  pipeline first
- The goal is metabolite identification rather than phenotype prediction — MassCube
  does not directly perform metabolite identification, it only enhances annotation
  accuracy
- Labels for the phenotype of interest are absent from the sample metadata, since
  feature selection is driven by metadata and importance strength
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MassCube
  role: Provides build_classifier (API and 'build-classifier' CLI) for metadata- and
    importance-driven feature selection and random forest phenotype prediction on
    final data tables
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Execution environment; MassCube leverages Python's numerical library ecosystem
    for scikit-learn-style estimation inside the workflow
  repo: https://github.com/huaxuyu/masscube
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:0f8e58fb4362e4bed6a8bf9346161c14a42991f5c3bbb80ad532857aaabac23d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MassCube
  - Python
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# random-forest-classifier-training

## Summary

Train a random forest phenotype classifier on quality-controlled metabolomic feature tables using MassCube's build_classifier (Python), selecting features by metadata association and importance strength. This is how MassCube closes the loop from raw LC-MS processing to phenotype prediction, e.g. distinguishing adolescent from middle-age mouse brain metabolomes.

## When to use

Use when a quality-controlled LC-MS feature table with sample metadata (e.g. age group, sex, brain region) is available and the goal is metabolome-informed phenotype prediction via a random forest with feature selection and ROC-AUC / cross-validation evaluation.

## When NOT to use

- Input is raw LC-MS data (mzML or Bruker .d) that has not yet been processed to an aligned, quality-controlled feature table — run MassCube's peak detection and alignment pipeline first
- The goal is metabolite identification rather than phenotype prediction — MassCube does not directly perform metabolite identification, it only enhances annotation accuracy
- Labels for the phenotype of interest are absent from the sample metadata, since feature selection is driven by metadata and importance strength

## Inputs

- Quality-controlled feature table (post-alignment, e.g. from Metabolome Atlas of the Aging Mouse Brain Zenodo deposit)
- Sample metadata with phenotype labels (e.g. age group: adolescence vs middle-age, sex, brain region)

## Outputs

- Fitted random forest classifier on the selected metabolomic features
- Ranked list of selected metabolites (32 features in the published benchmark)
- classifier_metrics.json containing ROC AUC (1.000) and 5-fold CV accuracy (0.991 ± 0.008)

## How to apply

Load the deposited feature table (e.g. the Metabolome Atlas of the Aging Mouse Brain subset from the Zenodo deposit) and use sample metadata to subset the groups of interest, such as adolescence vs middle-age. Call MassCube's build_classifier (exposed both in the package API and as the 'build-classifier' command-line entry point) to select features by metadata association and importance strength; in the published benchmark this selected 32 metabolites. Fit a random forest classifier on the selected features with scikit-learn-style estimation inside the MassCube workflow. Evaluate by ROC AUC and 5-fold cross-validated accuracy on the age-group labels, then write the metrics and the ranked selected-feature list to classifier_metrics.json. Rationale: the paper positions classifier building as the terminal module after peak detection, adduct/ISF grouping, alignment, and quality control of final data tables.

## Related tools

- **MassCube** (Provides build_classifier (API and 'build-classifier' CLI) for metadata- and importance-driven feature selection and random forest phenotype prediction on final data tables) — https://github.com/huaxuyu/masscube
- **Python** (Execution environment; MassCube leverages Python's numerical library ecosystem for scikit-learn-style estimation inside the workflow) — https://github.com/huaxuyu/masscube

## Examples

```
masscube build-classifier --input feature_table.csv --metadata sample_metadata.csv
```

## Evaluation signals

- Reported ROC AUC equals 1.000 on the adolescence vs middle-age labels
- Fivefold cross-validation accuracy falls at 0.991 ± 0.008
- The selected-feature list contains 32 metabolites with importance-based ranking
- classifier_metrics.json is written and parses with the AUC and CV-accuracy fields populated
- Predictions correctly distinguish the two age groups despite known batch effects in the aging mouse brain dataset

## Limitations

- ISFs cannot be automatically annotated when the parent ion is not detected due to complete fragmentation in the ion source, which can affect upstream feature quality feeding the classifier
- Downstream biological or clinical experiments remain necessary to confirm biological function of selected metabolites and validate generated hypotheses
- Confident biological insights would benefit from other software or databases for data interpretation or cross-assay integration
- No graphical user interface is available in the current version of MassCube

## Evidence

- [abstract] Final data tables undergo quality controls and can be used for metabolome-informed phenotype prediction: "Final data tables undergo quality controls and can be used for metabolome-informed phenotype prediction"
- [abstract] MassCube automatically detected age, sex and regional differences when applied to the Metabolome Atlas of the Aging Mouse Brain data despite batch effects.: "MassCube automatically detected age, sex and regional differences when applied to the Metabolome Atlas of the Aging Mouse Brain data despite batch effects."
- [discussion] downstream biological or clinical experiments remain necessary to confirm their biological function and validate the generated hypotheses.: "downstream biological or clinical experiments remain necessary to confirm their biological function and validate the generated hypotheses."
- [readme] masscube is an integrated Python package for liquid chromatography-mass spectrometry (LC-MS) data processing.: "masscube is an integrated Python package for liquid chromatography-mass spectrometry (LC-MS) data processing."
