---
name: directed-vs-mixed-enrichment-mode-interpretation
description: Use when interpreting ReporterScore/GRSA enrichment results on KO abundance
  data and you need to decide whether reporter score signs can be read as pathway-level
  increasing or decreasing trends.
when_to_use_negative:
- The KO abundance table has been prefiltered to background/significant features only
  — GRSA is a threshold-free method and requires the unfiltered abundance table
- The goal is a standard over-representation (hypergeometric/Fisher) analysis of a
  pre-defined KO list rather than signed, threshold-free reporter scoring
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3174
tools:
- name: ReporterScore (R package)
  role: 'Implements GRSA: ko.test, reporter_score (mode = directed/mixed), get_reporter_score,
    plot_report, plot_enrich_res, export_report_table'
  repo: https://github.com/Asa12138/ReporterScore
- name: R
  role: Runtime environment for running the ReporterScore workflow
  repo: https://github.com/Asa12138/ReporterScore
- name: pcutils
  role: Dependency installed alongside ReporterScore for visualization utilities
  repo: https://github.com/Asa12138/pcutils
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae116
    title: Generalized reporter score-based enrichment analysis for omics data
  source_content_hash: sha256:1639b8ab014197e29a2fb2bf0cca24b6ff16a223d1a0ce519b8d28c1bfeb4d6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Asa12138/ReporterScore
  tools:
  - R
  - ReporterScore
  - ReporterScore (R package)
  - pcutils
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# directed-vs-mixed-enrichment-mode-interpretation

## Summary

Reproduces the GRSA directed-vs-mixed mode comparison on the packaged ex_KO_profile metagenomic data to demonstrate that directed-mode reporter score signs preserve within-pathway KO trends while mixed-mode signs are uninterpretable due to cancellation of opposing-trend KOs.

## When to use

Use when interpreting ReporterScore/GRSA enrichment results on KO abundance data and you need to decide whether reporter score signs can be read as pathway-level increasing or decreasing trends.

## When NOT to use

- The KO abundance table has been prefiltered to background/significant features only — GRSA is a threshold-free method and requires the unfiltered abundance table
- The goal is a standard over-representation (hypergeometric/Fisher) analysis of a pre-defined KO list rather than signed, threshold-free reporter scoring

## Inputs

- KO abundance table (rownames = K numbers, colnames = samples), e.g. the packaged KO_abundance data from ReporterScore
- Sample metadata table (rownames = samples matching abundance colnames, columns = grouping variables such as Group)
- Built-in KEGG pathway-KO / module-KO databases (load_KOlist, 2023-08 version)

## Outputs

- Per-KO statistics table from ko.test
- Directed-mode reporter score enrichment table with signed pathway-level scores
- Mixed-mode reporter score enrichment table
- grsa_mode_comparison.csv containing directed- and mixed-mode enrichment tables side by side
- Within-pathway KO trend plots (plot_report / plot_enrich_res)

## How to apply

Load the packaged KO abundance table and metadata from the ReporterScore R package and run ko.test on the KO abundances against the metadata grouping to obtain per-KO statistics. Run reporter_score twice on ko.test output, once with mode set to the directed option so pvalue2zs converts KO p-values into signed statistics preserving within-pathway direction, and once with mode=mixed, then aggregate pathway scores with get_reporter_score. Inspect within-pathway KO trends for enriched pathways via plot_report or plot_enrich_res: directed-mode enriched pathways should show concordant KO directions, while in mixed mode opposing-trend KOs cancel so the sign cannot be interpreted. Export the directed- and mixed-mode enrichment tables side by side (e.g. grsa_mode_comparison.csv) using export_report_table-style output.

## Related tools

- **ReporterScore (R package)** (Implements GRSA: ko.test, reporter_score (mode = directed/mixed), get_reporter_score, plot_report, plot_enrich_res, export_report_table) — https://github.com/Asa12138/ReporterScore
- **R** (Runtime environment for running the ReporterScore workflow) — https://github.com/Asa12138/ReporterScore
- **pcutils** (Dependency installed alongside ReporterScore for visualization utilities) — https://github.com/Asa12138/pcutils

## Examples

```
data("KO_abundance_test"); ko_stat <- ko.test(KO_abundance, "Group", metadata); rs_dir <- reporter_score(ko_stat, mode="directed"); rs_mixed <- reporter_score(ko_stat, mode="mixed")
```

## Evaluation signals

- The comparison table grsa_mode_comparison.csv contains enriched pathways for BOTH directed and mixed modes
- Directed-mode enriched pathways exhibit consistent within-pathway KO trend directions (all KOs moving up, or all down, relative to background)
- Mixed-mode reporter score signs cannot be interpreted as pathway trends because opposing-trend KOs cancel in the signed Z-score aggregation
- All sample names in the abundance table match (or partially match) the metadata rownames before running ko.test
- Pathway-level reporter scores are aggregated by get_reporter_score from per-KO signed statistics, not recomputed independently

## Limitations

- RSA reporter score signs are often misinterpreted — in mixed mode, the increasing or decreasing trend of an enriched pathway cannot be determined from the sign
- If KOs within a pathway have opposing trends, signed Z-scores cancel each other, leading to insignificant results even when the pathway is perturbed
- GRSA exhibits slightly lower specificity, requiring additional experimental validation

## Evidence

- [methods] Directed mode preserves within-pathway direction: "reporter_score(kodf, group, metadata, method, pattern, p.adjust.method1, mode, verbose, feature, type, p.adjust.method2, modulelist, threads, perm, min_exist_KO, max_exist_KO)"
- [other] Directed-mode pathways have concordant KO directions: [evidence span withheld — anchor status: not_found]
- [other] Mixed-mode sign cancellation: [evidence span withheld — anchor status: not_found]
- [intro] Misinterpretation of reporter score signs: [evidence span withheld — anchor status: not_found]
- [readme] Threshold-free, unfiltered input requirement: [evidence span withheld — anchor status: external_doc]
