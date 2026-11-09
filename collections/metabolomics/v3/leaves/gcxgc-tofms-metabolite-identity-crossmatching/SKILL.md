---
name: gcxgc-tofms-metabolite-identity-crossmatching
description: Use when two significant metabolite feature lists from GCxGC-TOFMS plasma
  profiling (named metabolites plus unknown peaks annotated by mass spectrum and retention
  index) must be matched across studies to classify overlap, direction concordance,
  and condition-unique features.
when_to_use_negative:
- You have a single study's feature table and no second condition/study to cross-reference
  against — the classification of concordance/discordance requires both significant
  metabolite lists.
- The candidate comparison study's unknown peaks lack mass-spectrum/retention-index
  metadata, so Unknown_### peak labels cannot be matched on identity rather than label.
- The goal is diagnostic classification of new patient samples (OPLS-DA/ROC biomarker
  modeling) rather than reconciling which metabolites are shared between published
  signatures.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GCxGC-TOFMS
  role: Source analytical platform whose plasma metabolite feature lists (195 metabolites,
    annotated by mass spectrum and retention index) supply the significant-metabolite
    sets that are cross-matched between the acute enteric fever and chronic carriage
    studies
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:d7b2331fd5e56386f94814c7c659a05df6e18d7623e71aa80cde909332a6762d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# gcxgc-tofms-metabolite-identity-crossmatching

## Summary

Cross-references significant metabolites (including unknown-identity peaks) between two GCxGC-TOFMS studies by metabolite identity — matching putatively named metabolites such as glutaric acid and hexanoic acid and unknown peaks via mass spectra/retention index — to determine whether a disease-state signature is unique to one condition or shared, as done for chronic Salmonella gallbladder carriage vs acute enteric fever.

## When to use

Use when two significant metabolite feature lists from GCxGC-TOFMS plasma profiling (named metabolites plus unknown peaks annotated by mass spectrum and retention index) must be matched across studies to classify overlap, direction concordance, and condition-unique features.

## When NOT to use

- You have a single study's feature table and no second condition/study to cross-reference against — the classification of concordance/discordance requires both significant metabolite lists.
- The candidate comparison study's unknown peaks lack mass-spectrum/retention-index metadata, so Unknown_### peak labels cannot be matched on identity rather than label.
- The goal is diagnostic classification of new patient samples (OPLS-DA/ROC biomarker modeling) rather than reconciling which metabolites are shared between published signatures.

## Inputs

- Supplementary comparison table (S3 Table) of metabolites annotated with significance and direction of change in acute enteric fever and in chronic carriage (CSV/TSV export of the GCxGC-TOFMS feature table, 195 metabolites)
- Significant metabolite list from the acute enteric fever GCxGC-TOFMS study (with direction of change per metabolite)
- Significant metabolite list from the chronic carriage GCxGC-TOFMS study of Nepali cholecystectomy patients (S. Typhi and S. Paratyphi A carriers vs non-carriage controls)
- Mass spectra and retention-index annotations for unknown-identity peaks (used to match Unknown_### labels across studies)

## Outputs

- metabolite_comparison_acute_vs_carriage.csv — structured table reproducing Fig 3A counts (3 concordant-increased-in-carriers, 7 concordant-increased-in-controls, 16 discordant, 5 carriage-only elevated)
- List of metabolites elevated in the S. Typhi/S. Paratyphi A group in carriage samples only (glutaric acid, hexanoic acid, Unknown_087, Unknown_118, Unknown_399)
- Per-metabolite concordance classification (concordant-up-in-carriers / concordant-up-in-controls / discordant / carriage-only)

## How to apply

Load the comparison table (e.g. S3 Table) where each of the 195 GCxGC-TOFMS metabolites is annotated with significance and direction of change in each condition (carriage vs non-carriage controls; acute enteric fever cases vs controls). Cross-reference metabolites across the two studies by identity: named compounds (e.g. glutaric acid, hexanoic acid) by name, and unknown-identity peaks by matching mass spectra and retention index rather than arbitrary peak labels. For each shared significant metabolite, classify direction concordance: increased in the S. Typhi/S. Paratyphi A group in both conditions, increased in controls in both conditions, or direction-discordant between acute infection and carriage. Then extract features elevated in the S. Typhi/S. Paratyphi A group in the carriage samples only — here glutaric acid, hexanoic acid, and the unknowns Unknown_087, Unknown_118, Unknown_399 — as the carriage-unique signature. Emit a structured comparison table and validate the reproduced counts against the paper's reported values before trusting the derived 'unique to carriage' claim.

## Related tools

- **GCxGC-TOFMS** (Source analytical platform whose plasma metabolite feature lists (195 metabolites, annotated by mass spectrum and retention index) supply the significant-metabolite sets that are cross-matched between the acute enteric fever and chronic carriage studies)

## Examples

```
import pandas as pd; t=pd.read_csv('S3_Table.csv'); print(t.groupby(['direction_acute','direction_carriage']).size()); t[t.direction_acute.isna() & (t.direction_carriage=='up_S_Typhi_ParaA')].to_csv('metabolite_comparison_acute_vs_carriage.csv')
```

## Evaluation signals

- Reproduced counts must match the reported values exactly: 3 metabolites increased in the S. Typhi/S. Paratyphi A group in both conditions, 7 increased in controls in both conditions, 16 direction-discordant, and 5 carriage-only elevated (glutaric acid, hexanoic acid, Unknown_087, Unknown_118, Unknown_399).
- Every unknown-identity metabolite (Unknown_###) in the shared list must be matched across studies via mass spectra/retention index, not by peak label string equality.
- The carriage-only elevated list must contain exactly 5 metabolites, of which only 2 have putative names (glutaric acid, hexanoic acid).
- Sum of classification categories (3 + 7 + 16 + 5) must equal the number of metabolites reported as significant across the comparison.
- Output CSV must carry per-metabolite direction-of-change annotations for both conditions so the concordance calls are auditable against S3 Table.

## Limitations

- The carriage study had a small sample size and no independent validation cohort, so the cross-study concordance classification and the 5-metabolite carriage-only signature require validation before reuse.
- Three of the five carriage-only elevated metabolites have unknown identity, limiting biological interpretation and the reliability of cross-study identity matching.
- All patients presented with gallbladder conditions, so the metabolite signatures may be confounded by the primary cause of cholecystectomy, and controls with other bacterial gallbladder carriage were absent — the comparison cannot demonstrate Salmonella-specificity.

## Evidence

- [abstract] Substantial heterogeneity between acute infection and carriage signatures: "these data revealed substantial heterogeneity between metabolites associated with acute enteric fever and chronic carriage"
- [intro] Motivation for checking uniqueness of carriage profiles: "to investigate if the identified metabolite profiles were unique to carriage with respect to those identified during acute infection"
- [abstract] Explicit comparison workflow step: "We additionally compared carriage metabolite profiles with profiles generated during acute infection"
- [supplementary] Feature basis of the cross-matched metabolite lists: "models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy"
- [abstract] Analytical platform generating the metabolite identities: "using two-dimensional gas chromatography coupled with time-of-flight mass spectrometry (GCxGC-TOFMS)"
