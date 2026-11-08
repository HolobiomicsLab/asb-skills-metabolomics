---
name: qualitative-finding-tabulation-from-article-text
description: Use when an article's Results section compares multiple methods or conditions
  (e.g. CI-isobutane vs CI-methane vs EI at 70/15 eV) and you need a structured per-method
  table with quantitative metrics and figure provenance.
when_to_use_negative:
- The article reports only aggregated conclusions without per-condition statements,
  figures, or quantitative metrics to trace rows against
- The comparison data are already available as a structured table or supplementary
  file that can be used directly instead of re-extracted from prose
- The task requires quantitative re-analysis of raw spectra (e.g. isotopologue quantification
  with isoSCAN) rather than tabulating reported findings
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Scripting environment used to assemble the comparison rows and write ionization_method_comparison.csv
  repo: https://github.com/jcapelladesto/isoSCAN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:849c63d96da6642cdebdacd3afeca065e8e30ae6da77cc71e8aa0bf9c826fff7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/isoSCAN
  tools:
  - R
  - isoSCAN
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# qualitative-finding-tabulation-from-article-text

## Summary

Extracts per-condition qualitative and quantitative findings from an article's Results text and assembles them into a traced comparison table (e.g. ionization_method_comparison.csv) where every row cites an explicit statement, figure, or supplementary figure.

## When to use

Use when an article's Results section compares multiple methods or conditions (e.g. CI-isobutane vs CI-methane vs EI at 70/15 eV) and you need a structured per-method table with quantitative metrics and figure provenance.

## When NOT to use

- The article reports only aggregated conclusions without per-condition statements, figures, or quantitative metrics to trace rows against
- The comparison data are already available as a structured table or supplementary file that can be used directly instead of re-extracted from prose
- The task requires quantitative re-analysis of raw spectra (e.g. isotopologue quantification with isoSCAN) rather than tabulating reported findings

## Inputs

- Article Results-section text reporting the method comparison (Figure 1A–C and Supplementary Figure S1 passages)
- Named reference condition or metabolite anchoring the comparison (lactate in 13C-U-glucose labeled and non-labeled cultures)
- List of methods/conditions to tabulate (CI-methane, CI-isobutane, EI 70 eV, EI 15 eV)

## Outputs

- ionization_method_comparison.csv with rows per method and columns for reagent/technique, isotopic-envelope behavior, fragmentation assessment, and supporting metric/figure

## How to apply

1) Read the article's Results passages covering the comparison of interest — here the ionization-method comparison using lactate as the reference metabolite in 13C-U-glucose labeled and non-labeled cultures (Figure 1A–C, Supplementary Figure S1). 2) Extract per-method findings: isobutane CI concentrating metabolite abundance in the [M+H]+ isotopic envelope; methane CI inducing fragmentation of [M+H]+; the >5-fold higher [M+H]+ intensity with isobutane across glycolytic and TCA intermediates (Figure 1C); EI at 70 eV causing extensive fragmentation; and low-energy EI at 15 eV fragmenting more than CI-isobutane (Suppl. Figure S1). 3) Record each method's key qualitative outcome (suitable spectra for isotopic enrichment detection or not) plus the quantitative evidence where reported. 4) Assemble one row per method (methane CI, isobutane CI, EI 70 eV, EI 15 eV) with columns for reagent/technique, envelope behavior, fragmentation assessment, and the supporting metric/figure. 5) Write the table to ionization_method_comparison.csv using R and verify every row traces to an explicit statement in the article text before finalizing.

## Related tools

- **R** (Scripting environment used to assemble the comparison rows and write ionization_method_comparison.csv) — https://github.com/jcapelladesto/isoSCAN

## Examples

```
write.csv(method_comparison, "ionization_method_comparison.csv", row.names = FALSE)
```

## Evaluation signals

- Every row in ionization_method_comparison.csv cites an explicit statement, figure, or supplementary figure from the article text
- All four methods (CI-methane, CI-isobutane, EI 70 eV, EI 15 eV) appear as distinct rows with non-empty fragmentation assessments
- The isobutane-vs-methane quantitative claim ('>5-fold' [M+H]+ intensity, Figure 1C) is recorded exactly as stated, not rounded or paraphrased
- Qualitative suitability verdicts match the article: CI-isobutane 'most suitable spectra to detect isotopic enrichments'; EI at 70 eV 'extensive fragmentation'
- CSV loads cleanly in R with one header row and no orphan columns or misaligned fields

## Limitations

- Tabulation inherits the article's scope: findings cover glycolytic and TCA cycle intermediates with lactate as the reference metabolite; other metabolite classes (e.g. polyamines with their distinct GC oven gradient) may not be covered by the cited passages
- Some comparisons rely on supplementary material (Suppl. Figure S1 for EI 15 eV) whose full underlying data are not in the main text
- The workflow captures qualitative outcomes and reported metrics only — it cannot recover per-sample intensities unless stated in the text

## Evidence

- [results] isobutane [M+H]+ advantage: [evidence span withheld — anchor status: not_found]
- [results] EI 70 eV fragmentation: [evidence span withheld — anchor status: not_found]
- [results] EI 15 eV vs CI-isobutane: [evidence span withheld — anchor status: not_found]
- [abstract] isobutane best performance: [evidence span withheld — anchor status: not_found]
- [results] lactate reference comparison: [evidence span withheld — anchor status: not_found]
