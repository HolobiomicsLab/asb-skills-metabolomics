---
name: ko-abundance-statistical-testing
description: Use when input is an unfiltered KO abundance table (rownames = K numbers,
  colnames = samples) plus metadata with at least two groups, and per-KO statistics
  are needed before pathway/module enrichment.
when_to_use_negative:
- The input abundance table has already been prefiltered, since GRSA requires background
  feature information and prefiltering biases the threshold-free statistics.
- The input features are not K numbers, gene symbols, or KEGG C numbers (e.g. unstandardized
  metabolite IDs), because ID-to-feature matching against the modulelist will fail.
- The goal is a single-feature differential expression report rather than pathway/module-level
  enrichment; ko.test alone provides no pathway aggregation.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3174
tools:
- name: ReporterScore (R package)
  role: Provides ko.test for per-KO testing and reporter_score/get_reporter_score
    for signed pathway aggregation
  repo: https://github.com/Asa12138/ReporterScore
- name: devtools
  role: Installs the development version of ReporterScore and its pcutils dependency
    from GitHub
  repo: https://github.com/Asa12138/pcutils
- name: R
  role: Runtime environment in which the KO testing and reporter scoring pipeline
    is executed
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae116
    title: Generalized reporter score-based enrichment analysis for omics data
  source_content_hash: sha256:07a10d3e46b1d4f9964e788d56c95e91385ada0595b13f7372111767991c8471
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Asa12138/ReporterScore
  tools:
  - R
  - ReporterScore
  - ReporterScore (R package)
  - devtools
  - pcutils
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# ko-abundance-statistical-testing

## Summary

Per-KO differential statistical testing of a KEGG KO abundance table against sample metadata grouping, using ko.test from the ReporterScore (GRSA) package, producing per-KO statistics that feed pathway-level reporter scoring.

## When to use

Use when input is an unfiltered KO abundance table (rownames = K numbers, colnames = samples) plus metadata with at least two groups, and per-KO statistics are needed before pathway/module enrichment.

## When NOT to use

- The input abundance table has already been prefiltered, since GRSA requires background feature information and prefiltering biases the threshold-free statistics.
- The input features are not K numbers, gene symbols, or KEGG C numbers (e.g. unstandardized metabolite IDs), because ID-to-feature matching against the modulelist will fail.
- The goal is a single-feature differential expression report rather than pathway/module-level enrichment; ko.test alone provides no pathway aggregation.

## Inputs

- KO abundance table (rownames = KEGG K numbers such as K03169; colnames = samples; read counts or normalized values like TPM/relative abundance)
- Sample metadata table (rownames = samples, columns = grouping variables; rownames must match abundance table colnames)
- KEGG pathway/module KOlist (built-in 2023-08 version or updated via KEGG API)

## Outputs

- Per-KO statistics table (ko_stat) with p-values and signed statistics per KO
- Pathway-level reporter score table with reporter score signs and adjusted p-values (exportable via export_report_table, e.g. grsa_mode_comparison.csv)
- Reporter_score_res object inspectable via plot_report / plot_enrich_res

## How to apply

Load the KO abundance table (e.g. the packaged KO_abundance) and metadata whose rownames match the table's colnames, and do NOT prefilter the abundance table because GRSA is a threshold-free method. Call ko.test(kodf, group, metadata, method, pattern, p.adjust.method1, threads, verbose) to compute per-KO differential statistics against the chosen grouping variable (categorical, multi-timepoint longitudinal, or continuous). Then run reporter_score with mode='directed' (or call get_reporter_score on the ko_stat) so pvalue2zs converts KO p-values into signed statistics preserving within-pathway direction, and aggregate to pathway reporter scores against the built-in KEGG pathway/module KOlist. Optionally re-run with mode='mixed' to obtain the classic unsigned reporter scores, where opposing-trend KOs cancel and score signs are uninterpretable for pathway trends. Tune the built-in filters (p.adjust.method1, p.adjust.method2, min_exist_KO, max_exist_KO, perm) and optionally update the KOlist via update_KO_file() so the pathway database is current.

## Related tools

- **ReporterScore (R package)** (Provides ko.test for per-KO testing and reporter_score/get_reporter_score for signed pathway aggregation) — https://github.com/Asa12138/ReporterScore
- **devtools** (Installs the development version of ReporterScore and its pcutils dependency from GitHub) — https://github.com/Asa12138/pcutils
- **R** (Runtime environment in which the KO testing and reporter scoring pipeline is executed)

## Examples

```
ko.test(kodf, group="Group", metadata, method="wilcox.test"); reporter_score(kodf, group="Group", metadata, mode="directed")
```

## Evaluation signals

- Sample compatibility check passes: all(rownames(metadata) %in% colnames(KO_abundance)) returns TRUE.
- The comparison of directed vs mixed modes yields enriched pathways for both modes, and directed-mode enriched pathways show concordant within-pathway KO directions while mixed-mode signs cannot be interpreted due to cancellation of opposing-trend KOs.
- ko_stat contains statistics for all (unfiltered) input KOs, and downstream get_reporter_score applies min_exist_KO/max_exist_KO existence filters against the modulelist.
- Validation against published trends, e.g. lipid A biosynthesis reporter scores increasing and d-glucuronate degradation scores decreasing across young-to-old aging groups, recovers the expected directionality.
- Exported enrichment tables (export_report_table) include pathway IDs, reporter score signs, and adjusted p-values for each mode/group.

## Limitations

- GRSA exhibits slightly lower specificity, so additional experimental validation of enriched pathways is recommended.
- No version number is stated for the ReporterScore package, limiting reproducibility; pathway databases should be updated via update_KO_file() as the built-in set is a 2023-08 version.
- In mixed mode, the reporter score sign cannot indicate whether an enriched pathway is increasing or decreasing because opposing-trend KOs cancel each other's signed Z-scores.

## Evidence

- [methods] ko.test per-KO testing: "- `ko.test(kodf, group, metadata, method, pattern, p.adjust.method1, threads, verbose)`"
- [methods] reporter_score signature: "- `reporter_score(kodf, group, metadata, method, pattern, p.adjust.method1, mode, verbose, feature, type, p.adjust.method2, modulelist, threads, perm, min_exist_KO, max_exist_KO)`"
- [readme] unfiltered abundance table requirement: [evidence span withheld — anchor status: external_doc]
- [readme] metadata matching requirement: [evidence span withheld — anchor status: external_doc]
- [results] directed vs mixed mode behavior: [evidence span withheld — anchor status: not_found]
