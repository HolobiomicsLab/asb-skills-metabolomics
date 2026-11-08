---
name: cross-validated-classifier-performance-evaluation
description: Use when a final, QC-passed LC-MS feature table with sample metadata
  (e.g., age groups like adolescence vs middle-age) must be turned into a phenotype
  classifier and its performance quantified via ROC AUC and 5-fold cross-validated
  accuracy.
when_to_use_negative:
- Input is still raw vendor LC-MS data (mzML or Bruker .d) — run MassCube's peak detection,
  adduct/ISF grouping, and alignment first to produce a feature table
- The goal is metabolite identification or confirming biological function of the selected
  metabolites — the article states MassCube does not directly perform metabolite identification
  and that downstream biological or clinical experiments remain necessary
- Samples lack phenotype labels in metadata — build_classifier selects features by
  metadata, so no labeled groups means no classifier training
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MassCube
  role: Provides build_classifier (API and 'build-classifier' CLI entry point) to
    select metabolomic features by metadata and importance strength and fit the random-forest
    phenotype classifier
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Language ecosystem in which MassCube runs scikit-learn-style random-forest
    estimation and ROC AUC / cross-validation computation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:a597779b931138fc0321de47dacc124230f06947efd2e3833647887c0d94b7b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MassCube
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-validated-classifier-performance-evaluation

## Summary

Evaluates a metabolome-informed phenotype classifier built by MassCube by computing ROC AUC and fivefold cross-validated accuracy on age-group labels, reproducing the random-forest age prediction (32 metabolites, adolescence vs middle-age) from the Metabolome Atlas of the Aging Mouse Brain.

## When to use

Use when a final, QC-passed LC-MS feature table with sample metadata (e.g., age groups like adolescence vs middle-age) must be turned into a phenotype classifier and its performance quantified via ROC AUC and 5-fold cross-validated accuracy.

## When NOT to use

- Input is still raw vendor LC-MS data (mzML or Bruker .d) — run MassCube's peak detection, adduct/ISF grouping, and alignment first to produce a feature table
- The goal is metabolite identification or confirming biological function of the selected metabolites — the article states MassCube does not directly perform metabolite identification and that downstream biological or clinical experiments remain necessary
- Samples lack phenotype labels in metadata — build_classifier selects features by metadata, so no labeled groups means no classifier training

## Inputs

- Aligned, quality-controlled LC-MS feature table (e.g., MassCube final data table) from the Metabolome Atlas of the Aging Mouse Brain Zenodo deposit
- Sample metadata with age-group labels distinguishing adolescence and middle-age samples

## Outputs

- classifier_metrics.json containing ROC AUC (1.000) and 5-fold cross-validated accuracy (0.991 ± 0.008)
- Ranked list of selected metabolites (~32 features) ordered by importance strength
- Fitted random-forest classifier over the selected metabolomic features

## How to apply

Load the deposited feature table (e.g., the Zenodo deposit of the Metabolome Atlas of the Aging Mouse Brain) and subset samples to the phenotype classes of interest using the sample metadata — here the adolescence and middle-age groups. Call MassCube's build_classifier (exported in the package API and exposed as the 'build-classifier' command-line entry point) to select features by metadata association and importance strength, which yields ~32 selected metabolites. Fit a random forest classifier on those selected metabolomic features using scikit-learn-style estimation inside the MassCube workflow. Evaluate by computing ROC AUC on held predictions and the mean ± standard deviation of 5-fold cross-validated accuracy on the age-group labels. Accept the run when the reported AUC equals 1.000 and the 5-fold CV accuracy falls at 0.991 ± 0.008, matching the published phenotype-prediction performance.

## Related tools

- **MassCube** (Provides build_classifier (API and 'build-classifier' CLI entry point) to select metabolomic features by metadata and importance strength and fit the random-forest phenotype classifier) — https://github.com/huaxuyu/masscube
- **Python** (Language ecosystem in which MassCube runs scikit-learn-style random-forest estimation and ROC AUC / cross-validation computation)

## Examples

```
masscube build-classifier
```

## Evaluation signals

- Reported ROC AUC equals 1.000 on the adolescence vs middle-age classification
- 5-fold cross-validated accuracy falls at 0.991 ± 0.008 on the age-group labels
- classifier_metrics.json is produced with both metrics and the ranked selected-feature list
- The selected-feature list contains roughly 32 metabolites ranked by importance strength
- The classifier correctly distinguishes adolescent from middle-age mouse brain samples despite known batch effects in the Aging Mouse Brain data

## Limitations

- Selected metabolites require downstream biological or clinical experiments to confirm biological function and validate generated hypotheses
- The Aging Mouse Brain data contain batch effects that the classifier must overcome; phenotype prediction follows final data-table quality controls
- Planned supporting features such as SERRF data normalization and a GUI are not yet available in the current MassCube version

## Evidence

- [abstract] Final data tables undergo quality controls and can be used for metabolome-informed phenotype prediction: "Final data tables undergo quality controls and can be used for metabolome-informed phenotype prediction"
- [abstract] Age/sex/region detection despite batch effects: "MassCube automatically detected age, sex and regional differences when applied to the Metabolome Atlas of the Aging Mouse Brain data despite batch effects."
- [other] build_classifier entry point (source card): [evidence span withheld — anchor status: not_found]
- [other] Published performance metrics (source card): [evidence span withheld — anchor status: not_found]
- [discussion] Downstream validation caveat: "downstream biological or clinical experiments remain necessary to confirm their biological function and validate the generated hypotheses."
