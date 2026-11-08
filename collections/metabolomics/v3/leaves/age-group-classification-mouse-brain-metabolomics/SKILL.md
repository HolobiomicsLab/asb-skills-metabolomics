---
name: age-group-classification-mouse-brain-metabolomics
description: Use when a labeled LC-MS metabolomics feature table (e.g., the Zenodo-deposited
  Aging Mouse Brain data) must be turned into a phenotype-prediction classifier distinguishing
  sample groups such as adolescence vs middle-age.
when_to_use_negative:
- Input is raw vendor LC-MS data (mzML or Bruker .d) that has not yet been processed
  to an aligned, quality-controlled feature table — run MassCube's untargeted workflow
  first
- The goal is metabolite identification rather than phenotype prediction — MassCube
  does not directly perform metabolite identification
- The phenotype classes are not represented in the sample metadata (e.g., time-series
  or unmatched labels); planned but unavailable features like time-series analyses
  are not in the current version
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: MassCube
  role: Provides build_classifier (Python API and 'build-classifier' CLI) for metadata-
    and importance-based feature selection and random-forest phenotype prediction
    on quality-controlled feature tables
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Language ecosystem in which MassCube is implemented and through which the
    random-forest classifier and scikit-learn-style estimation are run
  repo: https://github.com/huaxuyu/masscube
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:819578ed03e34e94a307b733d62b88f7e3c7764ed0931e1bf9a92a7101c82d97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MassCube
  - Python
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# age-group-classification-mouse-brain-metabolomics

## Summary

Reproduces the MassCube phenotype-classifier benchmark in which a random forest built from 32 metabolites selected by metadata and importance strength predicts mouse age group (adolescence vs middle-age) from the Metabolome Atlas of the Aging Mouse Brain feature table, achieving AUC 1.000 and 5-fold CV accuracy 0.991 ± 0.008.

## When to use

Use when a labeled LC-MS metabolomics feature table (e.g., the Zenodo-deposited Aging Mouse Brain data) must be turned into a phenotype-prediction classifier distinguishing sample groups such as adolescence vs middle-age.

## When NOT to use

- Input is raw vendor LC-MS data (mzML or Bruker .d) that has not yet been processed to an aligned, quality-controlled feature table — run MassCube's untargeted workflow first
- The goal is metabolite identification rather than phenotype prediction — MassCube does not directly perform metabolite identification
- The phenotype classes are not represented in the sample metadata (e.g., time-series or unmatched labels); planned but unavailable features like time-series analyses are not in the current version

## Inputs

- Feature table from the Metabolome Atlas of the Aging Mouse Brain (Zenodo deposit) with sample metadata
- Sample metadata labels distinguishing adolescence vs middle-age groups

## Outputs

- classifier_metrics.json containing performance metrics (AUC 1.000, 5-fold CV accuracy 0.991 ± 0.008)
- Ranked list of the 32 selected metabolite features

## How to apply

Load the deposited feature table from the Metabolome Atlas of the Aging Mouse Brain (Zenodo) and subset samples to the adolescence and middle-age groups using the sample metadata. Call MassCube's build_classifier — exported in the package API and exposed as the 'build-classifier' command-line entry point — to select features by metadata association and importance strength; in the published benchmark this yielded 32 selected metabolites. Fit a random forest classifier (scikit-learn-style estimation inside the MassCube workflow) on the selected metabolomic features, then evaluate with ROC AUC and 5-fold cross-validated accuracy against the age-group labels. Rationale: the paper shows that final MassCube data tables that have passed quality controls are suitable for metabolome-informed phenotype prediction, and that the framework automatically detected age, sex, and regional differences despite batch effects.

## Related tools

- **MassCube** (Provides build_classifier (Python API and 'build-classifier' CLI) for metadata- and importance-based feature selection and random-forest phenotype prediction on quality-controlled feature tables) — https://github.com/huaxuyu/masscube
- **Python** (Language ecosystem in which MassCube is implemented and through which the random-forest classifier and scikit-learn-style estimation are run) — https://github.com/huaxuyu/masscube

## Examples

```
masscube build-classifier --feature-table aging_mouse_brain_features.csv --metadata samples.csv --labels age_group --output classifier_metrics.json
```

## Evaluation signals

- Reported ROC AUC equals 1.000 on the adolescence vs middle-age labels
- 5-fold cross-validated accuracy falls at 0.991 ± 0.008
- The selected-feature list contains exactly 32 metabolites ranked by importance strength
- classifier_metrics.json is written with both the AUC and CV accuracy fields populated
- Sample subset contains only adolescence and middle-age groups per the sample metadata

## Limitations

- Downstream biological or clinical experiments remain necessary to confirm the biological function of selected metabolites and validate generated hypotheses
- Batch effects are present in the Aging Mouse Brain data and must be tolerated by the classifier rather than removed by the skill
- MassCube has no GUI in the current version and does not directly perform metabolite identification

## Evidence

- [abstract] Final data tables undergo quality controls and can be used for metabolome-informed phenotype prediction: "Final data tables undergo quality controls and can be used for metabolome-informed phenotype prediction"
- [abstract] MassCube automatically detected age, sex and regional differences in the Metabolome Atlas of the Aging Mouse Brain data despite batch effects.: [evidence span withheld — anchor status: not_found]
- [methods] Task workflow: load deposited feature table and subset to adolescence and middle-age, then use build_classifier to select features and fit a random forest.: [evidence span withheld — anchor status: not_found]
- [results] Validation criterion: run accepted if AUC equals 1.000 and 5-fold CV accuracy matches 0.991 ± 0.008.: [evidence span withheld — anchor status: not_found]
- [readme] README: MassCube is an integrated Python package for LC-MS data processing with feature quality evaluation.: "**masscube** is an integrated Python package for liquid chromatography-mass spectrometry (LC-MS) data processing."
