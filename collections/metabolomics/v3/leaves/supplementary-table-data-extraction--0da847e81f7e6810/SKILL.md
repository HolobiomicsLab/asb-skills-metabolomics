---
name: supplementary-table-data-extraction
description: Use when claims in a paper (ROC AUC benchmarks, metabolite counts, concordance
  categories) must be verified against numeric values locked in supplementary tables
  supplied as .docx/.xlsx, rather than re-derived from raw data.
when_to_use_negative:
- The metabolite data are already available as a machine-readable feature table and
  the goal is de novo modeling (OPLS-DA, PCA, ROC analysis) rather than extraction
  and validation of published values.
- The claim to verify depends only on main-text figures with no corresponding supplementary
  table — there is nothing to parse and reproduce.
- The target metabolites are only reported as unknown-identity peaks with no mass
  spectra/retention index available to match across studies.
edam_operation: http://edamontology.org/operation_2409
edam_topics: []
tools:
- name: GCxGC-TOFMS
  role: Instrument whose metabolite annotations (named and Unknown_### peaks) populate
    the supplementary tables being parsed; identifiers must be preserved exactly as
    reported from the GCxGC-TOFMS analysis
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:d05d3894eda7965e177de1a771243e46e7bbaee67d061373e8587bbb9e4f8e50
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# supplementary-table-data-extraction

## Summary

Extract and structure quantitative results from supplementary tables (e.g., S2_Table.docx individual-metabolite AUCs, S3_Table acute-vs-carriage comparisons) so that reported statistics can be re-ranked, cross-referenced, and validated against figure-based claims.

## When to use

Use when claims in a paper (ROC AUC benchmarks, metabolite counts, concordance categories) must be verified against numeric values locked in supplementary tables supplied as .docx/.xlsx, rather than re-derived from raw data.

## When NOT to use

- The metabolite data are already available as a machine-readable feature table and the goal is de novo modeling (OPLS-DA, PCA, ROC analysis) rather than extraction and validation of published values.
- The claim to verify depends only on main-text figures with no corresponding supplementary table — there is nothing to parse and reproduce.
- The target metabolites are only reported as unknown-identity peaks with no mass spectra/retention index available to match across studies.

## Inputs

- S2_Table.docx — supplementary table of AUC values and 95% CIs for 195 metabolites from the OPLS-DA model
- S3_Table — supplementary comparison of acute enteric fever vs chronic carriage metabolites with significance and direction-of-change annotations
- Reported benchmark values from main-text figures (Fig 2A pattern AUC, Fig 3A counts, S2 Fig five-metabolite ROC curves)

## Outputs

- ranked_metabolite_auc_table.csv — 195 metabolites ranked by descending AUC plus benchmark rows and verification flag
- metabolite_comparison_acute_vs_carriage.csv — concordance-classified metabolite comparison table (3/7/16) and carriage-only elevated list (glutaric acid, hexanoic acid, Unknown_087, Unknown_118, Unknown_399)

## How to apply

Locate the supplementary file containing the values of interest (e.g., S2_Table.docx for per-metabolite AUCs of the 195 metabolites in the OPLS-DA model, S3_Table for the acute-enteric-fever-vs-chronic-carriage comparison). Parse the document programmatically, preserving metabolite identifiers (including unknown-identity peaks such as Unknown_087, Unknown_118, Unknown_399 matched via mass spectra/retention index), AUC values, 95% confidence intervals, and direction-of-change annotations. For S3 Table, cross-reference carriage-significant metabolites against acute-infection-significant ones by identity and classify concordance: 3 increased in S. Typhi/S. Paratyphi A in both conditions, 7 increased in controls in both, 16 discordant, and 5 elevated in carriage only (glutaric acid, hexanoic acid, and three unknowns). For S2 Table, rank all 195 metabolites by descending AUC and append benchmark rows from Fig 2A (pattern AUC >= 0.95) and the five-metabolite panel (AUC 0.935). Emit structured CSVs (e.g., metabolite_comparison_acute_vs_carriage.csv, ranked_metabolite_auc_table.csv) whose counts and thresholds are validated verbatim against the Results text and figures.

## Related tools

- **GCxGC-TOFMS** (Instrument whose metabolite annotations (named and Unknown_### peaks) populate the supplementary tables being parsed; identifiers must be preserved exactly as reported from the GCxGC-TOFMS analysis)

## Examples

```
python scripts/parse_supp_table.py --input S2_Table.docx --rank --benchmarks pattern_auc=0.95,panel_auc=0.935 --output ranked_metabolite_auc_table.csv
```

## Evaluation signals

- The reproduced comparison table reproduces exactly the Fig 3A / S3 Table counts: 3 concordant-increased-in-carriers, 7 concordant-increased-in-controls, 16 discordant, and 5 carriage-only elevated metabolites.
- The maximum individual-metabolite AUC among the 195 ranked rows is strictly less than 0.935 and less than the Fig 2A multivariate pattern AUC (>= 0.95), with a True verification flag recorded.
- The five named panel metabolites parse with their expected AUC values: hexanoic acid 0.841, Unknown_087 0.697, Unknown_118 0.756, glutaric acid 0.774, Unknown_399 0.876.
- The parsed S2 Table contains exactly 195 metabolite rows, matching the OPLS-DA model's metabolite count.
- Output CSVs carry metabolite identifiers verbatim, including Unknown_### peaks annotated via mass spectra/retention index.

## Limitations

- The study's small sample size limits the reliability of the supplementary-table statistics being extracted and validated.
- Several key metabolites have unknown identity (Unknown_087, Unknown_118, Unknown_399) and can only be matched across tables via mass spectra/retention index, risking annotation ambiguity.
- The published values require further epidemiological validation in alternative endemic enteric fever settings before reuse as biomarker benchmarks.

## Evidence

- [supplementary] models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy: "models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy"
- [abstract] We were able to significantly discriminate Salmonella carriage samples from non-carriage control samples.: "We were able to significantly discriminate Salmonella carriage samples from non-carriage control samples."
- [supplementary] Panels A-E are showing ROC curves with false positive rates (i.e. 1-specificity) and true positive rates (i.e. sensitivity) on the x- and y-axes: "Panels A-E are showing ROC curves with false positive rates (i.e. 1-specificity) and true positive rates (i.e. sensitivity) on the x- and y-axes respectively"
- [abstract] these data revealed substantial heterogeneity between metabolites associated with acute enteric fever and chronic carriage: "these data revealed substantial heterogeneity between metabolites associated with acute enteric fever and chronic carriage"
