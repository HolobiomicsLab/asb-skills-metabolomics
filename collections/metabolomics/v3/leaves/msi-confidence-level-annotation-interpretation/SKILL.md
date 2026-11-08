---
name: msi-confidence-level-annotation-interpretation
description: Use when you need metabolite annotation for LC-MS features using accurate
  mass, isotopic patterns, and MS^E fragmentation spectra at MSI confidence level
  2, or when annotated features are required as ground truth to validate feature-matching
  results.
when_to_use_negative:
- When the goal is metabolite annotation itself rather than validating feature-matching
  results — use a dedicated annotation pipeline with MS/MS spectra and database search
  instead
- When annotated features are unavailable or too sparse to serve as ground truth for
  match validation (the article notes this strategy is 'limited by the number of annotations
  in our datasets')
- When the matched feature pairs lack consistent annotation labels across datasets
  — the label comparison step cannot distinguish correct from wrong matches without
  annotated labels on both sides
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: M2S (Match 2 Sets)
  role: Produces the final match tables whose correctness against manually annotated
    features is scored by this skill
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Environment in which the M2S matching toolbox and annotation-matching validation
    are executed
  repo: https://github.com/rjdossan/M2S
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:03aab07db90c336f6289e6f7c4315e9adb497836dd12dd9f245b190132b1b3be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S (Match 2 Sets)
  - MATLAB
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# msi-confidence-level-annotation-interpretation

## Summary

Annotate metabolomic features to Metabolomics Standards Initiative (MSI) confidence level 2 using accurate mass, isotopic patterns, and MS^E fragmentation spectra, then use those annotated labels as ground truth to score how many and what fraction of annotated features were correctly matched across LC–MS datasets. Serves as an orthogonal validation layer for the M2S feature-matching workflow.

## When to use

Use when you need metabolite annotation for LC-MS features using accurate mass, isotopic patterns, and MS^E fragmentation spectra at MSI confidence level 2, or when annotated features are required as ground truth to validate feature-matching results.

## When NOT to use

- When the goal is metabolite annotation itself rather than validating feature-matching results — use a dedicated annotation pipeline with MS/MS spectra and database search instead
- When annotated features are unavailable or too sparse to serve as ground truth for match validation (the article notes this strategy is 'limited by the number of annotations in our datasets')
- When the matched feature pairs lack consistent annotation labels across datasets — the label comparison step cannot distinguish correct from wrong matches without annotated labels on both sides

## Inputs

- Final M2S match tables for the reference (MESA) and target (Rotterdam) datasets, listing uniquely matched feature pairs after step 3 of the matching pipeline
- Annotated feature lists per dataset, where annotations were assigned at MSI confidence level 2 using accurate mass, isotopic patterns, and MS^E fragmentation spectra against in-house and online databases (LIPID MAPS, METLIN, HMDB, GNPS, MassBank)

## Outputs

- Per-dataset validation summary table (annotation_match_validation.csv) with columns: dataset, total_annotated, correct_matches, fraction_correct
- Counts of correctly matched annotated features (e.g., Dataset 1: 585/604; Dataset 2: 79/87) and the corresponding fraction_correct percentages (96.8% and 90.8%)

## How to apply

1) Prepare the annotated feature lists for both datasets (e.g., Dataset 1 with 604 annotations and Dataset 2 with 87 annotations) assigned at MSI confidence level 2. 2) For each annotated feature in the reference (MESA) dataset, retrieve its matched partner in the target (Rotterdam) dataset from the final M2S match tables (Dataset 1: 4953 unique matches; Dataset 2: 2324 unique matches) and compare the annotation labels; a match counts as 'correct' only if both features carry the same annotation. 3) Count total annotated features, correct matches, wrong matches, and annotated features falling outside initial thresholds (Dataset 1: 19 outside; Dataset 2: 8 outside/wrong). 4) Compute the fraction correct = correct matches / total annotated features per dataset, and assemble a validation summary table (annotation_match_validation.csv) with columns dataset, total_annotated, correct_matches, fraction_correct. 5) Judge correctness by checking that annotation labels agree on both sides of each match pair and that the fraction_correct value is in the reported ranges (Dataset 1: 96.8%; Dataset 2: 90.8%).

## Related tools

- **M2S (Match 2 Sets)** (Produces the final match tables whose correctness against manually annotated features is scored by this skill) — https://github.com/rjdossan/M2S
- **MATLAB** (Environment in which the M2S matching toolbox and annotation-matching validation are executed) — https://github.com/rjdossan/M2S

## Examples

```
# In MATLAB, after loading match tables and annotated feature lists:
run_m2s_validation(match_table_D1, annotated_D1, match_table_D2, annotated_D2); % writes annotation_match_validation.csv
```

## Evaluation signals

- Fraction of annotated features correctly matched should reach ~96.8% for Dataset 1 (585 of 604) and ~90.8% for Dataset 2 (79 of 87)
- Annotation labels on both sides of a matched pair must agree for the match to count as 'correct'; label disagreement marks the match as wrong
- Total annotated features, correct matches, wrong matches, and annotated features outside initial thresholds (Dataset 1: 19; Dataset 2: 8) must reconcile: total = correct + wrong + outside/wrong
- Output CSV annotation_match_validation.csv must contain the columns dataset, total_annotated, correct_matches, fraction_correct
- Annotated features must exist for both the MESA reference and Rotterdam target datasets; missing annotations on either side invalidate the fraction_correct denominator

## Limitations

- Validation by annotated features is limited by the number of annotations available in the datasets
- Most missed annotated matches are between features that behave very differently from modeled inter-dataset shifts of RT, MZ, and FI, so low-annotation-coverage datasets may hide real matching failures
- Step 3 of the M2S pipeline (removal of poor matches via tightened thresholds) may inadvertently delete some correct matches, which the annotation-based validation alone cannot always recover

## Evidence

- [abstract] Annotated feature match accuracy: "4953 features were uniquely matched, of which 585 (96.8%) of 604 manually annotated features were correct"
- [abstract] Second dataset validation: "2324 features could be uniquely matched, with 79 (90.8%) out of 87 annotated features correctly matched"
- [methods] Annotated features outside thresholds: [evidence span withheld — anchor status: not_found]
- [methods] Dataset 1 feature counts: "After QC, the MESA dataset contained 10 427 features and the Rotterdam dataset contained 14 097."
- [methods] Dataset 2 feature counts: "After QC, the MESA dataset contained 6793 features and the Rotterdam dataset contained 6315."
- [methods] MSI confidence level 2: "Features, including adducts and isotopologues, were annotated to confidence level 2 according to the Metabolomics Standards Initiative."
- [abstract] Why annotated matches fail: "Most of the missed annotated matches are between features that behave very differently from modeled inter-dataset shifts of RT, MZ, and FI"
