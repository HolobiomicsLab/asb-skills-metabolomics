---
name: carriage-only-biomarker-candidate-selection
description: Use when metabolite profiles from a chronic carriage study must be compared
  with those of acute infection to find metabolites elevated in carriage only, e.g.
  GCxGC-TOFMS data of S. Typhi/S. Paratyphi A carriage vs controls, cross-referenced
  against acute enteric fever significant metabolite lists.
when_to_use_negative:
- When the aim is simply to discriminate carriers from non-carriers without any comparison
  to acute infection profiles — the OPLS-DA discrimination workflow already covers
  that goal.
- When the input is a raw GCxGC-TOFMS peak table that has not yet been statistically
  analysed for carriage vs control significance; the cross-reference step requires
  pre-computed significant metabolite lists.
- When comparing carriage profiles to an independent carriage cohort (validation)
  rather than to acute infection profiles — this skill is specifically for acute-vs-carriage
  uniqueness testing.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: GCxGC-TOFMS
  role: Two-dimensional gas chromatography coupled with time-of-flight mass spectrometry
    used to generate the plasma metabolite profiles (195 metabolites) that feed the
    acute-vs-carriage comparison.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:6bfb4899ce63fdc42b784c3a62dfdae127e70827df8ea998a49234a91a817af0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# carriage-only-biomarker-candidate-selection

## Summary

Identifies metabolites that discriminate Salmonella gallbladder carriage and are unique to carriage by cross-referencing significant metabolites from the carriage cohort against significant metabolites from the acute enteric fever cohort, classifying shared metabolites by direction of change.

## When to use

Use when metabolite profiles from a chronic carriage study must be compared with those of acute infection to find metabolites elevated in carriage only, e.g. GCxGC-TOFMS data of S. Typhi/S. Paratyphi A carriage vs controls, cross-referenced against acute enteric fever significant metabolite lists.

## When NOT to use

- When the aim is simply to discriminate carriers from non-carriers without any comparison to acute infection profiles — the OPLS-DA discrimination workflow already covers that goal.
- When the input is a raw GCxGC-TOFMS peak table that has not yet been statistically analysed for carriage vs control significance; the cross-reference step requires pre-computed significant metabolite lists.
- When comparing carriage profiles to an independent carriage cohort (validation) rather than to acute infection profiles — this skill is specifically for acute-vs-carriage uniqueness testing.

## Inputs

- Metabolite comparison table (S3 Table) listing metabolites with significance and direction of change in acute enteric fever and in chronic carriage
- Significant metabolite list from the carriage cohort (GCxGC-TOFMS plasma data, S. Typhi / S. Paratyphi A carriers vs non-carriage controls)
- Significant metabolite list from the acute enteric fever cohort

## Outputs

- Structured comparison table (metabolite_comparison_acute_vs_carriage.csv) reproducing the Fig 3A counts: 3 concordant-increased-in-carriers, 7 concordant-increased-in-controls, 16 discordant, 5 carriage-only elevated
- List of carriage-only elevated metabolites: glutaric acid, hexanoic acid, Unknown_087, Unknown_118, Unknown_399

## How to apply

Load the acute-vs-carriage metabolite comparison table (S3 Table) as a metabolite list annotated with significance and direction of change in each condition. Cross-reference the carriage-study significant metabolites against the acute enteric fever significant metabolites by metabolite identity, including putatively identified metabolites (e.g. glutaric acid, hexanoic acid) and unknown-identity peaks matched via mass spectra/retention index. Classify each shared significant metabolite by concordance of direction of change: 3 metabolites increased in the S. Typhi/S. Paratyphi A group in both conditions, 7 increased in the control group in both conditions, and 16 direction-discordant. Extract the metabolites elevated in the carriage group only (5 total: glutaric acid, hexanoic acid and three unknowns, Unknown_087, Unknown_118, Unknown_399). Validate by confirming the counts match Fig 3A/S3 Table (3/7/16 concordant/discordant and 5 carriage-only elevated).

## Related tools

- **GCxGC-TOFMS** (Two-dimensional gas chromatography coupled with time-of-flight mass spectrometry used to generate the plasma metabolite profiles (195 metabolites) that feed the acute-vs-carriage comparison.)

## Examples

```
python compare_metabolites.py --s3-table S3_Table.csv --carriage-significant carriage_hits.csv --acute-significant acute_hits.csv --output metabolite_comparison_acute_vs_carriage.csv
```

## Evaluation signals

- Reproduced comparison table contains exactly 3 metabolites increased in the S. Typhi/S. Paratyphi A group in both conditions, 7 increased in the control group in both conditions, and 16 direction-discordant metabolites.
- Carriage-only elevated metabolite list contains exactly 5 entries: glutaric acid, hexanoic acid, Unknown_087, Unknown_118, Unknown_399.
- All shared significant metabolites are classified as concordant-increased-in-carriers, concordant-increased-in-controls, or discordant (no unclassified entries).
- Output CSV matches the counts reported in the Results section and summarized in Fig 3A and S3 Table.

## Limitations

- The major limitation of this study was the small sample size, which was difficult to overcome given the ethical, surgical, and diagnostic issues of identifying and confirming invasive Salmonella.
- No independent validation cohort was available, so resulting metabolite profiles should be validated in an additional, independent cohort.
- All patients presented with gallbladder conditions, so the primary cause for the cholecystectomy may impact the metabolite signatures.
- Lack of controls with other bacterial gallbladder carriage means it is not possible to demonstrate that the carriage-only metabolites are truly Salmonella-specific.
- The relevance of the carboxylic acid biomarkers (e.g. hexanoic acid) is uncertain; they may reflect altered gut microbiome substrate fermentation.

## Evidence

- [supplementary] Comparison of acute enteric fever and chronic carriage profiles revealed metabolites unique to carriage: five metabolites were elevated in the S. Typhi/S. Paratyphi A group in the carriage samples only.: [evidence span withheld — anchor status: not_found]
- [abstract] Substantial heterogeneity was observed between metabolites associated with acute enteric fever and chronic carriage.: "these data revealed substantial heterogeneity between metabolites associated with acute enteric fever and chronic carriage"
- [intro] Aim to investigate if the identified metabolite profiles were unique to carriage with respect to those identified during acute infection.: "to investigate if the identified metabolite profiles were unique to carriage with respect to those identified during acute infection"
- [abstract] Salmonella carriers could be significantly distinguished from non-carriage controls using only five metabolites.: "Salmonella carriers could be significantly distinguished from non-carriage controls using only five metabolites"
- [discussion] We were uncertain of the relevance of these carboxylic acids, but these chemicals have antibacterial properties and may be the result of altered gut microbiome substrate fermentation.: "We are uncertain of the relevance of these carboxylic acids, but these chemicals have antibacterial properties [29] and may be the result of altered gut microbiome substrate fermentation."
