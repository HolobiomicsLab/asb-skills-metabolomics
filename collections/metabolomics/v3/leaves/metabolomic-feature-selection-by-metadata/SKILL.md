---
name: metabolomic-feature-selection-by-metadata
description: Use when a quality-controlled metabolomic feature table plus sample metadata
  (e.g., age group, sex, brain region) is available and the goal is metadata-driven
  feature selection and phenotype prediction.
when_to_use_negative:
- Input is raw vendor LC-MS data rather than a processed feature table — run MassCube's
  peak detection, adduct/ISF grouping, alignment, and QC modules first
- The goal is metabolite identification/structural elucidation — MassCube does not
  directly perform metabolite identification, and downstream experiments remain necessary
  to confirm biological function of selected metabolites
- The task needs a graphical interface or time-series/statistical modules not yet
  implemented (no GUI, no time-series analysis, no SERRF normalization in the current
  version)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MassCube
  role: Provides build_classifier (Python API and 'build-classifier' CLI) for metadata-
    and importance-based feature selection and random-forest phenotype prediction
    from QC'd feature tables
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Runtime environment for the MassCube classifier workflow and scikit-learn-style
    estimation and cross-validation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:04ce758440ebcedd9f0e785da695ba0060c0b2e5f3e41f6f283a2f4b73623736
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MassCube
  - Python
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# metabolomic-feature-selection-by-metadata

## Summary

Select discriminative metabolomic features from a QC'd LC-MS feature table by metadata group membership and feature-importance strength, then fit a random-forest phenotype classifier (e.g., adolescent vs middle-age mouse brain) and report AUC and cross-validated accuracy.

## When to use

Use when a quality-controlled metabolomic feature table plus sample metadata (e.g., age group, sex, brain region) is available and the goal is metadata-driven feature selection and phenotype prediction.

## When NOT to use

- Input is raw vendor LC-MS data rather than a processed feature table — run MassCube's peak detection, adduct/ISF grouping, alignment, and QC modules first
- The goal is metabolite identification/structural elucidation — MassCube does not directly perform metabolite identification, and downstream experiments remain necessary to confirm biological function of selected metabolites
- The task needs a graphical interface or time-series/statistical modules not yet implemented (no GUI, no time-series analysis, no SERRF normalization in the current version)

## Inputs

- Quality-controlled, aligned LC-MS feature table (e.g., from the Metabolome Atlas of the Aging Mouse Brain Zenodo deposit)
- Sample metadata with phenotype labels (e.g., age group: adolescence vs middle-age; also sex, brain region)
- MassCube package (Python) installed via pip

## Outputs

- classifier_metrics.json containing ROC AUC and 5-fold cross-validated accuracy (benchmark: AUC 1.000, accuracy 0.991 ± 0.008)
- Ranked list of selected metabolomic features (32 metabolites in the published benchmark)
- Fitted random-forest classifier distinguishing the metadata groups

## How to apply

Load the deposited feature table (e.g., the Metabolome Atlas of the Aging Mouse Brain Zenodo deposit) and subset samples to the target groups from the sample metadata — for the published benchmark, adolescence vs middle-age mouse brain samples. Call MassCube's build_classifier (exported in the package API and exposed as the 'build-classifier' command-line entry point) to select features by metadata group and importance strength; in the benchmark this converged on 32 selected metabolites. Fit a random forest classifier on the selected features using the scikit-learn-style estimation built into the MassCube workflow. Evaluate by computing ROC AUC on the age-group labels and 5-fold cross-validated accuracy, and write the ranked selected-feature list and metrics to classifier_metrics.json. The rationale is that MassCube's final data tables already pass quality controls and are explicitly intended for metabolome-informed phenotype prediction.

## Related tools

- **MassCube** (Provides build_classifier (Python API and 'build-classifier' CLI) for metadata- and importance-based feature selection and random-forest phenotype prediction from QC'd feature tables) — https://github.com/huaxuyu/masscube
- **Python** (Runtime environment for the MassCube classifier workflow and scikit-learn-style estimation and cross-validation)

## Examples

```
from masscube import build_classifier; model = build_classifier(feature_table, sample_metadata, target='age_group')
```

## Evaluation signals

- Reported ROC AUC equals 1.000 on the adolescence vs middle-age age-group labels
- 5-fold cross-validated accuracy falls at 0.991 ± 0.008, matching the published phenotype-prediction performance
- Selected-feature list contains 32 metabolites ranked by importance strength
- classifier_metrics.json is produced and parses with AUC and CV-accuracy fields populated
- Only samples from the two target metadata groups are included in training/evaluation (batch effects from other groups excluded from the comparison)

## Limitations

- Downstream biological or clinical experiments remain necessary to confirm the biological function of selected metabolites and validate the generated hypotheses
- MassCube does not directly perform metabolite identification; confident biological interpretation may require other software or databases for data interpretation or cross-assay integration
- No graphical user interface is available in the current version of MassCube, and planned features such as time-series analyses and SERRF data normalization are not yet available

## Evidence

- [abstract] Final data tables undergo quality controls and can be used for metabolome-informed phenotype prediction: "Final data tables undergo quality controls and can be used for metabolome-informed phenotype prediction"
- [abstract] MassCube automatically detected age, sex and regional differences in the Metabolome Atlas of the Aging Mouse Brain data despite batch effects.: "MassCube automatically detected age, sex and regional differences when applied to the Metabolome Atlas of the Aging Mouse Brain data despite batch effects."
- [discussion] Selected metabolites require downstream validation.: "downstream biological or clinical experiments remain necessary to confirm their biological function and validate the generated hypotheses."
- [discussion] MassCube does not directly perform metabolite identification.: "While MassCube does not directly perform metabolite identification, it enhances annotation accuracy in several ways."
- [readme] Feature detection, gap filling, and aligned feature grouping cache the parsed source for reuse.: "Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source."
