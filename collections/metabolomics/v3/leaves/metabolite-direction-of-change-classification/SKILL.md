---
name: metabolite-direction-of-change-classification
description: Use when comparing metabolite profiles from GCxGC-TOFMS studies of acute
  enteric fever versus chronic carriage to determine which metabolites are unique
  to carriage versus shared with discordant/concordant direction of change.
when_to_use_negative:
- When the goal is to derive a diagnostic classifier (latent biomarker) rather than
  to characterise the overlap and concordance of metabolite changes between acute
  infection and carriage.
- When metabolites cannot be matched across the two studies by identity (putatively
  identified names or unknown peaks matched by mass spectra/retention index), as classification
  of direction-of-change concordance is then undefined.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GCxGC-TOFMS
  role: Instrument that generated the plasma metabolite profiles from both acute enteric
    fever and carriage studies; the resulting metabolite lists (including unknown-identity
    peaks) are the input to the direction-of-change comparison.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:6872eeaa52bd7f97cada38872ff5e80716a350df78e0576483b32f08b755e3ea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# metabolite-direction-of-change-classification

## Summary

Cross-study classification of metabolites by concordance of direction of change between acute enteric fever and chronic Salmonella gallbladder carriage, identifying carriage-only elevated metabolites.

## When to use

Use when comparing metabolite profiles from GCxGC-TOFMS studies of acute enteric fever versus chronic carriage to determine which metabolites are unique to carriage versus shared with discordant/concordant direction of change.

## When NOT to use

- When the goal is to derive a diagnostic classifier (latent biomarker) rather than to characterise the overlap and concordance of metabolite changes between acute infection and carriage.
- When metabolites cannot be matched across the two studies by identity (putatively identified names or unknown peaks matched by mass spectra/retention index), as classification of direction-of-change concordance is then undefined.

## Inputs

- S3 Table supplementary metabolite comparison table (metabolite ID, significance and direction of change in each condition)
- List of significant metabolites from acute enteric fever study
- List of significant metabolites from chronic carriage study (GCxGC-TOFMS)

## Outputs

- Structured comparison table of metabolite concordance counts and direction-of-change categories (metabolite_comparison_acute_vs_carriage.csv)
- List of five carriage-only elevated metabolites (glutaric acid, hexanoic acid, Unknown_087, Unknown_118, Unknown_399)

## How to apply

Load the metabolite comparison data from S3 Table as a table of metabolites annotated with significance and direction of change in each condition. Cross-reference carriage-study significant metabolites against acute enteric fever study significant metabolites by metabolite identity, including putatively identified metabolites (e.g. glutaric acid, hexanoic acid) and unknown-identity peaks matched via mass spectra/retention index. Classify each shared metabolite by concordance of direction of change: metabolites increased in the S. Typhi/S. Paratyphi A group in both conditions (3), increased in the control group in both conditions (7), and direction-discordant between acute infection and carriage (16). Extract metabolites elevated in the S. Typhi/S. Paratyphi A group in carriage samples only (glutaric acid, hexanoic acid, Unknown_087, Unknown_118, Unknown_399). Emit a structured comparison table reproducing Fig 3A's counts and the carriage-only list.

## Related tools

- **GCxGC-TOFMS** (Instrument that generated the plasma metabolite profiles from both acute enteric fever and carriage studies; the resulting metabolite lists (including unknown-identity peaks) are the input to the direction-of-change comparison.)

## Examples

```
python classify_direction_of_change.py --s3_table S3_Table.csv --acute acute_fever_significant.csv --carriage carriage_significant.csv --output metabolite_comparison_acute_vs_carriage.csv
```

## Evaluation signals

- Verify the reproduced table matches the reported counts: 3 concordant-increased-in-carriers, 7 concordant-increased-in-controls, 16 direction-discordant, and 5 carriage-only elevated metabolites, as stated in the Results section and summarized in Fig 3A and S3 Table.
- Confirm the carriage-only list contains exactly five metabolites: glutaric acid, hexanoic acid, and three unknown-identity metabolites (Unknown_087, Unknown_118, Unknown_399).
- Check that every shared significant metabolite is assigned to exactly one concordance category (increased-in-carriers both / increased-in-controls both / discordant).
- Verify that unknown-identity peaks are matched across studies via mass spectra and retention index, not by name alone.
- Check the output CSV preserves metabolite identity (including putative IDs and unknown-peak IDs) alongside the direction-of-change category.

## Limitations

- The comparison is limited to metabolites detected and matched across the two studies; unknown-identity peaks must be matched via mass spectra/retention index, which may miss or mis-assign some peaks.
- The major limitation of the underlying study was the small sample size, which was difficult to overcome given the ethical, surgical, and diagnostic issues of identifying and confirming invasive Salmonella.
- Metabolite profiles from the underlying study should be validated in an additional, independent cohort, which was not possible in this setting.
- The physiological meaning of the detected metabolite profiles is unclear, and the relevance of identified carboxylic acid biomarkers is uncertain.

## Evidence

- [results] five metabolites (glutaric acid, hexanoic acid, and three metabolites with unknown identity) were elevated in the S. Typhi/S. Paratyphi A group in the carriage samples only: [evidence span withheld — anchor status: not_found]
- [abstract] substantial heterogeneity between metabolites associated with acute enteric fever and chronic carriage: "these data revealed substantial heterogeneity between metabolites associated with acute enteric fever and chronic carriage"
- [methods] unknown-identity peaks matched via mass spectra/retention index: [evidence span withheld — anchor status: not_found]
- [discussion] The major limitation of this study was the small sample size: "The major limitation of this study was the small sample size, which was difficult to overcome given the ethical, surgical, and diagnostic issues of identifying and confirming invasive Salmonella"
