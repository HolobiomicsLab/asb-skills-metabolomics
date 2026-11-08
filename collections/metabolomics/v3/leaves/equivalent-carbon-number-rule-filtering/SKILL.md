---
name: equivalent-carbon-number-rule-filtering
description: Use when a lipid MS2 annotation table contains putative lipid identities
  and inconsistent annotations (violating the ECN rule or feature-peak criteria) must
  be removed before computing false discovery rates.
when_to_use_negative:
- The annotation candidates have not yet passed EQ MS2 library search or the LCI retention-time
  rules — filtering first would discard valid candidates out of order
- The goal is double-bond positional isomer localization via Paternò–Büchi derivatization,
  which uses the PB spectral library rather than ECN-rule filtering
- Input is already a fully curated, manually reviewed lipid identification list with
  no unresolved inconsistent annotations
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: LipidIN
  role: Framework providing the EQ/LCI modules and within which the strict ECN-rule
    and feature-peak criteria filtering is enforced
  repo: github.com/LinShuhaiLAB/LipidIN
- name: LipidIN EQ module
  role: Produces the upstream candidate MS2 annotations (against the 24.6-million-spectrum
    theoretical library) that are then filtered by the ECN rule
- name: LipidIN LCI module
  role: Applies the three relative retention time rules with a best-first search heuristic
    before the strict ECN/feature-peak filtering step
- name: LipidIN WMYn module
  role: Regenerates the 5th-level fingerprint library used after filtering to boost
    recall and confirm low-confidence lipids
- name: MS-DIAL
  role: Peak-picking/annotation software whose outputs can feed the LipidIN annotation
    and filtering pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:3710be71edbee57a52d6ec554e219a4925b1dbb2dc0d5bb533e38d4e65a17233
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidIN
  - LipidIN EQ module
  - LipidIN LCI module
  - LipidIN WMYn module
  - MS-DIAL
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# equivalent-carbon-number-rule-filtering

## Summary

A strict filtering step from the LipidIN workflow that enforces the equivalent carbon number (ECN) rule and feature-peak criteria to remove inconsistent lipid annotations, contributing to the reported 5.69% average false discovery rate across four datasets.

## When to use

Use when a lipid MS2 annotation table contains putative lipid identities and inconsistent annotations (violating the ECN rule or feature-peak criteria) must be removed before computing false discovery rates.

## When NOT to use

- The annotation candidates have not yet passed EQ MS2 library search or the LCI retention-time rules — filtering first would discard valid candidates out of order
- The goal is double-bond positional isomer localization via Paternò–Büchi derivatization, which uses the PB spectral library rather than ECN-rule filtering
- Input is already a fully curated, manually reviewed lipid identification list with no unresolved inconsistent annotations

## Inputs

- Candidate lipid annotation table produced by the LipidIN EQ module MS2 library search
- LCI-filtered annotation results (subclass/retention-time consistent candidates)
- Raw MS/MS feature tables from the datasets being annotated (RBL-2H3 cells, mixture, human sera, zebrafish tissues)

## Outputs

- Filtered per-dataset annotation tables with ECN-rule and feature-peak-inconsistent annotations removed
- Per-dataset and aggregated false discovery rate summary (annotation_report.csv)

## How to apply

After running the LipidIN EQ module MS2 library search (chain compositions + C=C double-bond positions) against the LipidIN theoretical library, apply the ECN-rule (equivalent carbon number) and feature-peak criteria as a strict filter on the candidate annotations. This filtering is step 4 of the task workflow and is performed after the EQ library search and after the LCI module's three relative retention time rules (executed with a best-first search heuristic) have already removed retention-time-inconsistent false positives. Retain only annotations whose inferred lipid subclass properties are consistent under the ECN rule and whose expected diagnostic feature peaks are present in the MS/MS spectrum. Enforce the strict-criteria mode to reproduce the reported benchmark rather than relaxing thresholds. Combine with the LCI retention-time filtering and optionally confirm low-confidence lipids with the WMYn regenerated 5th-level fingerprint library.

## Related tools

- **LipidIN** (Framework providing the EQ/LCI modules and within which the strict ECN-rule and feature-peak criteria filtering is enforced) — github.com/LinShuhaiLAB/LipidIN
- **LipidIN EQ module** (Produces the upstream candidate MS2 annotations (against the 24.6-million-spectrum theoretical library) that are then filtered by the ECN rule)
- **LipidIN LCI module** (Applies the three relative retention time rules with a best-first search heuristic before the strict ECN/feature-peak filtering step)
- **LipidIN WMYn module** (Regenerates the 5th-level fingerprint library used after filtering to boost recall and confirm low-confidence lipids)
- **MS-DIAL** (Peak-picking/annotation software whose outputs can feed the LipidIN annotation and filtering pipeline)

## Examples

```
python lipidin_pipeline.py --datasets RBL-2H3,mixture,human_sera,zebrafish --library pos_ALL.rda neg_ALL.rda --mode strict --criteria ecn,feature_peaks --output annotation_report.csv
```

## Evaluation signals

- Average estimated FDR across the four datasets (RBL-2H3, mixture, human sera, zebrafish) should be ~5.69% at a cutoff threshold of 2.4 under strict criteria
- Total annotated lipids across the four datasets should reach ~8923, with ~471 unique lipids recovered in the mixture dataset (Fig. 3e-h, Supplementary Data 6-7)
- annotation_report.csv should contain per-dataset annotation tables plus an aggregated FDR summary consistent with the reported values
- Every retained annotation must satisfy both the ECN-rule and feature-peak criteria simultaneously; violations should be absent from the output
- Filtering must follow the LCI relative-retention-time step, and downstream FDR should decrease relative to EQ-search-only annotations

## Limitations

- The strict ECN-rule and feature-peak criteria trade coverage for accuracy — recall is only partially recovered afterwards via the WMYn 20% estimated recall boosting step
- Estimated FDR (~5.7%) is an estimate, not a decoy-based ground truth, and depends on dataset-specific conditions
- Filtering is designed for LC-MS/MS lipidomics data processed with the EQ+LCI pipeline; applied to arbitrary similarity-search outputs (e.g., MS-DIAL, LipidSearch, flash entropy search alone), it may not transfer since those tools are reported to cause feature redundancy

## Evidence

- [other] Enforce the strict ECN-rule (equivalent carbon number) and feature-peak criteria to filter out inconsistent annotations.: [evidence span withheld — anchor status: not_found]
- [other] EQ plus LCI achieved an average FDR of 5.69% at a cutoff threshold of 2.4 under strict criteria, annotating 8923 lipids in four datasets: [evidence span withheld — anchor status: not_found]
- [abstract] developed using three relative retention time rules, reducing false positive annotations and predicting unannotated lipids with a 5.7% estimated false discovery rate: "developed using three relative retention time rules, reducing false positive annotations and predicting unannotated lipids with a 5.7% estimated false discovery rate"
- [methods] Annotation (MS2 library search against theoretical library): "**Annotation** — MS2 library search against the theoretical library (chain compositions + C=C double-bond positions)"
