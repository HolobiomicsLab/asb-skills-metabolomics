---
name: metabolite-overlap-comparison-across-datasets
description: Use when two metabolomics studies of related conditions (e.g., acute
  infection vs. chronic carriage) each yield significant-metabolite lists, and the
  goal is to classify shared metabolites by concordant/discordant direction and identify
  condition-only elevated metabolites.
when_to_use_negative:
- The input is a single-study differential-expression table with no second condition/study
  to compare against — use a within-study differential analysis instead
- Metabolite identities cannot be matched across datasets (different annotation standards,
  no shared names or spectra/retention-index matches) — cross-referencing by identity
  is not feasible
- The goal is classifier discrimination between sample groups (OPLS-DA/ROC), not overlap
  characterization across studies
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: GCxGC-TOFMS
  role: Two-dimensional gas chromatography coupled with time-of-flight mass spectrometry
    used to generate the plasma metabolite profiles (195 metabolites) whose significant
    members are cross-compared between acute enteric fever and chronic carriage
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:4a1b61b78063330772caa5ef999456375d59d1badbb344909191277557110c0e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# metabolite-overlap-comparison-across-datasets

## Summary

Cross-dataset metabolite concordance analysis: compare significant metabolites identified in one GCxGC-TOFMS study (e.g., chronic Salmonella gallbladder carriage) against those from a related study (e.g., acute enteric fever), classifying overlap by metabolite identity and direction of change to determine whether biomarker profiles are condition-unique.

## When to use

Use when two metabolomics studies of related conditions (e.g., acute infection vs. chronic carriage) each yield significant-metabolite lists, and the goal is to classify shared metabolites by concordant/discordant direction and identify condition-only elevated metabolites.

## When NOT to use

- The input is a single-study differential-expression table with no second condition/study to compare against — use a within-study differential analysis instead
- Metabolite identities cannot be matched across datasets (different annotation standards, no shared names or spectra/retention-index matches) — cross-referencing by identity is not feasible
- The goal is classifier discrimination between sample groups (OPLS-DA/ROC), not overlap characterization across studies

## Inputs

- Supplementary comparison table (S3 Table) of metabolites annotated with significance and direction of change in each condition
- Significant-metabolite list from the chronic carriage GCxGC-TOFMS study (195 metabolites profiled in plasma from cholecystectomy patients)
- Significant-metabolite list from the acute enteric fever study
- Mass spectra / retention index annotations for unknown-identity peaks

## Outputs

- Structured comparison table (e.g., metabolite_comparison_acute_vs_carriage.csv) with concordance counts (3 concordant-increased-in-carriers, 7 concordant-increased-in-controls, 16 discordant) and the carriage-only elevated metabolite list
- List of condition-unique elevated metabolites (glutaric acid, hexanoic acid, Unknown_087, Unknown_118, Unknown_399)

## How to apply

Load the supplementary comparison table (here S3 Table) as metabolites annotated with significance and direction of change in each condition. Cross-reference significant metabolites between the carriage study and the acute enteric fever study by metabolite identity, matching putatively identified metabolites (glutaric acid, hexanoic acid) and unknown-identity peaks via mass spectra/retention index. Classify each shared significant metabolite by concordance of direction of change — counting metabolites increased in the S. Typhi/S. Paratyphi A group in both conditions, increased in controls in both conditions, and direction-discordant between conditions. Extract metabolites elevated in the carrier group in one condition only (five here: glutaric acid, hexanoic acid, Unknown_087, Unknown_118, Unknown_399). Emit a structured comparison table with counts plus the condition-only metabolite list, and validate counts against the reported figures.

## Related tools

- **GCxGC-TOFMS** (Two-dimensional gas chromatography coupled with time-of-flight mass spectrometry used to generate the plasma metabolite profiles (195 metabolites) whose significant members are cross-compared between acute enteric fever and chronic carriage)

## Examples

```
python compare_overlap.py --carriage S3_table.csv --acute acute_fever_significant.csv --match-by name,spectrum,retention_index --out metabolite_comparison_acute_vs_carriage.csv --expect 3:7:16 --carriage-only 5
```

## Evaluation signals

- Reproduced comparison table matches the reported counts: 3 metabolites increased in S. Typhi/S. Paratyphi A in both conditions, 7 increased in controls in both conditions, 16 direction-discordant
- Exactly 5 metabolites are extracted as elevated in the S. Typhi/S. Paratyphi A group in carriage samples only: glutaric acid, hexanoic acid, Unknown_087, Unknown_118, Unknown_399
- Every shared significant metabolite is assigned to exactly one concordance class (concordant-increased-in-carriers, concordant-increased-in-controls, discordant), with no unclassified entries
- Unknown-identity peaks are matched across datasets via mass spectra/retention index, not by name alone
- Output CSV parses and carries both the counts and the carriage-only metabolite list

## Limitations

- Substantial heterogeneity was observed between metabolites associated with acute enteric fever and chronic carriage, so concordant overlap is expected to be limited
- The carriage study's small sample size and lack of an independent validation cohort limit confidence in the significant-metabolite lists being compared
- Unknown-identity metabolites (e.g., Unknown_087, Unknown_118, Unknown_399) can only be matched via mass spectra/retention index, risking mis-assignment across datasets
- All patients presented with gallbladder conditions, so primary cause of cholecystectomy may confound the metabolite signatures being compared

## Evidence

- [abstract] these data revealed substantial heterogeneity between metabolites associated with acute enteric fever and chronic carriage: "these data revealed substantial heterogeneity between metabolites associated with acute enteric fever and chronic carriage"
- [abstract] Comparison of carriage metabolite profiles with profiles generated during acute infection: "We additionally compared carriage metabolite profiles with profiles generated during acute infection"
- [intro] to investigate if the identified metabolite profiles were unique to carriage with respect to those identified during acute infection: "to investigate if the identified metabolite profiles were unique to carriage with respect to those identified during acute infection"
- [supplementary] models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy: "models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy"
- [discussion] as all patients included in this study presented with gallbladder conditions we cannot rule out that the primary cause for the cholecystectomy may impact on the metabolite signatures.: "as all patients included in this study presented with gallbladder conditions we cannot rule out that the primary cause for the cholecystectomy may impact on the metabolite signatures."
