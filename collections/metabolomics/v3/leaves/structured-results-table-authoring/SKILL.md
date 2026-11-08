---
name: structured-results-table-authoring
description: Use when a research question requires comparing multiple methods or configurations
  reported in an article (e.g., reagent gases or ionization techniques) and the findings
  are scattered across Results text, figures, and supplementary figures.
when_to_use_negative:
- The source article reports only a single method with no comparison rows to assemble
  (a one-row table adds no structure)
- The comparison involves quantitative datasets to be re-analyzed rather than reported
  findings to be tabulated — use the analytical pipeline (e.g., isoSCAN on GC-CI-MS
  raw data) instead
- Claims in the article lack explicit supporting statements or figures, so rows cannot
  be traced to the text
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: R
  role: Used to assemble the method-comparison rows and write the resulting table
    to CSV
- name: isoSCAN
  role: Downstream R-based tool that consumes the quantified isotopologue context
    from GC-CI-MS studies whose method-comparison results this table summarizes
  repo: https://github.com/jcapelladesto/isoSCAN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:b0672cc5037dd5a6ca5e26d85d50c2f7dceed0951926894e0164a7b2e7b4ab25
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/isoSCAN
  tools:
  - R
  - isoSCAN
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# structured-results-table-authoring

## Summary

Assembles a machine-readable comparison table (e.g., ionization_method_comparison.csv) in which each row is a method variant (CI-isobutane, CI-methane, EI 70 eV, EI 15 eV) and each cell traces to an explicit statement or figure in a source article. Ensures extracted findings are organized into consistent columns rather than free-text notes.

## When to use

Use when a research question requires comparing multiple methods or configurations reported in an article (e.g., reagent gases or ionization techniques) and the findings are scattered across Results text, figures, and supplementary figures.

## When NOT to use

- The source article reports only a single method with no comparison rows to assemble (a one-row table adds no structure)
- The comparison involves quantitative datasets to be re-analyzed rather than reported findings to be tabulated — use the analytical pipeline (e.g., isoSCAN on GC-CI-MS raw data) instead
- Claims in the article lack explicit supporting statements or figures, so rows cannot be traced to the text

## Inputs

- Article Results-section passages reporting the method comparison (e.g., Figure 1A–C and Supplementary Figure S1 text)
- Extracted findings with section attributions (e.g., '>5-fold more intense using isobutane than methane (Figure 1C)')
- Target output schema (columns such as reagent/technique, envelope behavior, fragmentation assessment, supporting metric/figure)

## Outputs

- ionization_method_comparison.csv with one row per method (methane CI, isobutane CI, EI 70 eV, EI 15 eV)
- Columns: reagent/technique, isotopic envelope behavior, fragmentation assessment, reported supporting metric/figure

## How to apply

Read the Results passages reporting the comparison (here: lactate as reference metabolite in 13C-U-glucose labeled and non-labeled cultures, Figure 1A–C, Supplementary Figure S1). Extract per-method findings: CI-isobutane concentrating abundance in the [M+H]+ isotopic envelope vs methane-induced fragmentation of [M+H]+ (with the >5-fold intensity difference across glycolytic and TCA cycle intermediates, Figure 1C), EI at 70 eV causing extensive fragmentation, and low-energy EI at 15 eV vs CI-isobutane (Suppl. Figure S1). Define columns before writing rows — for this comparison: reagent/technique, isotopic envelope behavior, fragmentation assessment, and the reported supporting metric/figure. Record the key qualitative outcome (suitable spectra for isotopic enrichment detection or not) per method and the quantitative evidence only where the article reports it. Write the table to CSV and verify every row traces to an explicit statement in the article text before finalizing.

## Related tools

- **R** (Used to assemble the method-comparison rows and write the resulting table to CSV)
- **isoSCAN** (Downstream R-based tool that consumes the quantified isotopologue context from GC-CI-MS studies whose method-comparison results this table summarizes) — https://github.com/jcapelladesto/isoSCAN

## Examples

```
write.csv(ionization_method_comparison, "ionization_method_comparison.csv", row.names = FALSE)
```

## Evaluation signals

- Every row in the CSV traces to an explicit statement or figure in the article text (e.g., the >5-fold [M+H]+ intensity claim must cite Figure 1C)
- All four method variants reported in the article are present as rows: CI-methane, CI-isobutane, EI 70 eV, and EI 15 eV
- Each row contains non-empty values for all defined columns: reagent/technique, envelope behavior, fragmentation assessment, and supporting metric/figure
- Quantitative evidence appears only where the article reports it — CI-isobutane vs methane carries the >5-fold metric; the EI 15 eV comparison remains qualitative with reference to Suppl. Figure S1
- Qualitative suitability outcomes are mutually consistent with the article's conclusion that CI with isobutane yielded the most suitable spectra for detecting isotopic enrichments

## Limitations

- Only methods and metrics explicitly reported in the source article can populate rows; e.g., the low-energy EI (15 eV) comparison is supported by Supplementary Figure S1 without a reported fold-change value
- Quantitative support is uneven across methods — CI-isobutane vs methane has a reported metric (>5-fold, Figure 1C), while EI rows rely on qualitative fragmentation assessments
- Table rows are specific to the article's experimental context (lactate reference metabolite, 13C-U-glucose labeled and non-labeled cell cultures, glycolytic pathway and TCA cycle intermediates) and may not generalize to other metabolite classes

## Evidence

- [results] CI with isobutane yielded the most suitable spectra to detect isotopic enrichments compared with EI, using lactate as reference metabolite in 13C-U-glucose labeled and non-labeled cell cultures: [evidence span withheld — anchor status: not_found]
- [results] Isobutane concentrates abundance in [M+H]+ envelope; methane fragments it, with >5-fold intensity difference: [evidence span withheld — anchor status: not_found]
- [results] Quantitative supporting metric for the methane-vs-isobutane row: [evidence span withheld — anchor status: not_found]
- [results] EI fragmentation assessments supporting the EI 70 eV and EI 15 eV rows: [evidence span withheld — anchor status: not_found]
- [other] Verification requirement that every tabulated claim traces to the article text: [evidence span withheld — anchor status: not_found]
