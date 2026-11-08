---
name: metabolome-informed-phenotype-prediction
description: Use when a labeled metabolomics feature table (samples × metabolite intensities
  with phenotype metadata, e.g. age, sex, region) is available and the goal is an
  automated, metabolome-informed predictive classifier.
when_to_use_negative:
- The goal is metabolite identification or structural annotation — MassCube does not
  directly perform metabolite identification; it only supports identity- and fuzzy-search
  annotation
- Downstream biological or clinical validation of the selected metabolites' function
  is the goal — experiments are required to confirm biological function of selected
  features
- The data is raw LC-MS files rather than a processed, quality-controlled final data
  table — use MassCube's peak detection/alignment workflow first
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3383
- http://edamontology.org/topic_0091
tools:
- name: MassCube
  role: Provides build_classifier (Python API) and the 'build-classifier' command-line
    entry point for feature selection by metadata and importance strength, and produces
    the QC'd final data table used for phenotype prediction
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Runtime environment and scikit-learn-style random forest estimation within
    the MassCube workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:05e7fe672fead5ff9ec9d3eda938fb2ad3597455d6c4334bb0202724b4b85589
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MassCube
  - Python
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3383
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metabolome-informed-phenotype-prediction

## Summary

Build a phenotype classifier (random forest) from MassCube's quality-controlled metabolomics feature table, selecting features by metadata association and importance strength to predict sample labels such as mouse age group.

## When to use

Use when a labeled metabolomics feature table (samples × metabolite intensities with phenotype metadata, e.g. age, sex, region) is available and the goal is an automated, metabolome-informed predictive classifier.

## When NOT to use

- The goal is metabolite identification or structural annotation — MassCube does not directly perform metabolite identification; it only supports identity- and fuzzy-search annotation
- Downstream biological or clinical validation of the selected metabolites' function is the goal — experiments are required to confirm biological function of selected features
- The data is raw LC-MS files rather than a processed, quality-controlled final data table — use MassCube's peak detection/alignment workflow first

## Inputs

- Aligned, quality-controlled LC-MS feature table (samples × metabolite features) from MassCube processing, e.g. the Metabolome Atlas of the Aging Mouse Brain deposit on Zenodo (doi=10.5281/zenodo.11363475)
- Sample metadata sheet with phenotype labels (age group: adolescence vs middle-age; also sex and brain region)
- MassCube Python package (pip install masscube)

## Outputs

- Fitted random forest phenotype classifier
- Ranked list of selected metabolomic features (32 metabolites in the published benchmark)
- classifier_metrics.json with ROC AUC (1.000) and 5-fold CV accuracy (0.991 ± 0.008)

## How to apply

Load the deposited feature table (e.g. from the Metabolome Atlas of the Aging Mouse Brain Zenodo deposit) and subset samples to the phenotype groups of interest (adolescence vs middle-age) using the sample metadata. Call MassCube's build_classifier (exported in the package API and exposed as the 'build-classifier' CLI entry point) to select features by metadata and importance strength — in the published benchmark this yielded 32 selected metabolites. Fit a random forest classifier on the selected metabolomic features using the scikit-learn-style estimation inside the MassCube workflow. Evaluate with ROC AUC and 5-fold cross-validated accuracy on the age-group labels; the published run achieved AUC 1.000 and CV accuracy 0.991 ± 0.008. Write the metrics and the ranked selected-feature list to classifier_metrics.json. Note that final data tables must first pass MassCube's quality controls before being used for phenotype prediction.

## Related tools

- **MassCube** (Provides build_classifier (Python API) and the 'build-classifier' command-line entry point for feature selection by metadata and importance strength, and produces the QC'd final data table used for phenotype prediction) — https://github.com/huaxuyu/masscube
- **Python** (Runtime environment and scikit-learn-style random forest estimation within the MassCube workflow)

## Examples

```
from masscube import build_classifier; build_classifier(feature_table, sample_metadata, label_column='age_group', n_features=32)
```

## Evaluation signals

- Reported ROC AUC equals 1.000 on the adolescent vs middle-age classification task
- Fivefold cross-validated accuracy falls at 0.991 ± 0.008 on the aging mouse brain metabolome data
- The selected-feature list is ranked and contains the expected ~32 metabolites selected by metadata and importance strength
- classifier_metrics.json is written with both AUC and CV-accuracy fields populated
- The input feature table passed MassCube's final data-table quality controls before classifier building

## Limitations

- Batch effects were present in the Metabolome Atlas of the Aging Mouse Brain data; MassCube detected age, sex and regional differences despite them, but batch structure may confound feature selection
- Downstream biological or clinical experiments remain necessary to confirm biological function of selected metabolites and validate generated hypotheses
- External validated benchmarking datasets with large-scale ground-truth data are still missing, limiting rigorous benchmarking of classifier performance

## Evidence

- [abstract] Final data tables undergo quality controls and can be used for metabolome-informed phenotype prediction: "Final data tables undergo quality controls and can be used for metabolome-informed phenotype prediction"
- [abstract] MassCube detected age/sex/region differences despite batch effects: "MassCube automatically detected age, sex and regional differences when applied to the Metabolome Atlas of the Aging Mouse Brain data despite batch effects."
- [discussion] Selected metabolites require experimental validation: "downstream biological or clinical experiments remain necessary to confirm their biological function and validate the generated hypotheses."
- [discussion] Core modules demonstrated on the aging mouse brain dataset: "Using the Metabolome Atlas of the Aging Mouse Brain data, we demonstrated several core modules in MassCube"
