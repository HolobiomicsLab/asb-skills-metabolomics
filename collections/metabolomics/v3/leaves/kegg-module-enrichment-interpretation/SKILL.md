---
name: kegg-module-enrichment-interpretation
description: Use when you have a full (unprefiltered) KO/compound abundance table
  plus sample metadata with grouping, and need threshold-free, multi-group or longitudinal
  KEGG pathway/module enrichment rather than single-list over-representation.
when_to_use_negative:
- The feature abundance table has already been prefiltered to significant/background-filtered
  features — GRSA is threshold-free and requires the full background to remain valid
- You need organism-specific pathway-gene mapping for a non-human/mouse organism without
  building a custom modulelist (built-in org pathway databases cover hsa and mmu only)
- Your question is a classic two-list over-representation test on a pre-defined KO
  list rather than threshold-free aggregation of genome-wide differential statistics
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: ReporterScore
  role: 'Core R package implementing GRSA: ko.test, get_reporter_score, reporter_score,
    export_report_table, plot_report, plot_enrich_res'
  repo: https://github.com/Asa12138/ReporterScore
- name: devtools
  role: Used to install the development versions of ReporterScore and its pcutils
    dependency from GitHub
- name: pcutils
  role: R dependency installed alongside ReporterScore when installing from GitHub
  repo: https://github.com/Asa12138/pcutils
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae116
    title: Generalized reporter score-based enrichment analysis for omics data
  source_content_hash: sha256:d3c7dd76b33aa960eef662bb7f21c8718c3f4ae333fbf914930d5b9350e90733
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Asa12138/ReporterScore
  tools:
  - R
  - ReporterScore
  - devtools
  - pcutils
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# kegg-module-enrichment-interpretation

## Summary

Aggregates per-KO (or per-compound) differential statistics from omics feature tables to KEGG pathway/module-level reporter scores using the Generalized Reporter Score-based Analysis (GRSA) implemented in the R package ReporterScore, then interprets which KEGG modules are enriched per group or time point.

## When to use

Use when you have a full (unprefiltered) KO/compound abundance table plus sample metadata with grouping, and need threshold-free, multi-group or longitudinal KEGG pathway/module enrichment rather than single-list over-representation.

## When NOT to use

- The feature abundance table has already been prefiltered to significant/background-filtered features — GRSA is threshold-free and requires the full background to remain valid
- You need organism-specific pathway-gene mapping for a non-human/mouse organism without building a custom modulelist (built-in org pathway databases cover hsa and mmu only)
- Your question is a classic two-list over-representation test on a pre-defined KO list rather than threshold-free aggregation of genome-wide differential statistics

## Inputs

- KO (KEGG K number) abundance table: rows = K numbers, cols = samples; counts or normalized values (TPM/FPKM/RPKM/relative abundance), unprefiltered
- Sample metadata table: rownames = samples, columns = grouping variables (categorical ≥2 levels, multiple time points, or continuous)
- KEGG pathway-KO or module-KO database (built-in 2023-08 KOlist, or custom_modulelist / load_GOlist / CPDlist alternatives)
- Compound (KEGG C number) abundance table with standardized IDs, for metabolomics input

## Outputs

- reporter_score_res object: per-KO statistics plus pathway/module-level reporter scores with adjusted p-values per group
- Exported pathway-level result tables via export_report_table
- Enrichment plots via plot_report and plot_enrich_res showing enriched KEGG modules and directional trends

## How to apply

Install ReporterScore (CRAN, or devtools::install_github("Asa12138/ReporterScore") with the pcutils dependency). Provide a KO abundance table (rownames are K numbers like K03169, colnames are samples) that is NOT prefiltered, since GRSA is threshold-free, and metadata whose rownames match the abundance table colnames. Run reporter_score(kodf, group, metadata) — this calls ko.test for per-KO differential statistics (p.adjust.method1) and get_reporter_score to aggregate to pathway/module reporter scores against the built-in KEGG KOlist (2023-08 version, updatable via update_KO_file()), with filters p.adjust.method2, min_exist_KO, max_exist_KO and permutation-based p-values (perm). Choose type='pathway' or 'module' and optionally custom_modulelist databases; rank enriched modules by reporter score sign and adjusted p-value per group. Export with export_report_table and visualize with plot_report (rs_threshold, mode, show_ID, Pathway_description) and plot_enrich_res. Interpret score sign carefully: reporter score signs indicate direction of enrichment and are frequently misinterpreted in classic RSA usage.

## Related tools

- **ReporterScore** (Core R package implementing GRSA: ko.test, get_reporter_score, reporter_score, export_report_table, plot_report, plot_enrich_res) — https://github.com/Asa12138/ReporterScore
- **devtools** (Used to install the development versions of ReporterScore and its pcutils dependency from GitHub)
- **pcutils** (R dependency installed alongside ReporterScore when installing from GitHub) — https://github.com/Asa12138/pcutils

## Examples

```
reporter_score(kodf, Group, metadata, type="module", perm=1000)
```

## Evaluation signals

- Reporter scores reproduce expected directionality: e.g. in the IHSMGC skin microbiota re-analysis, lipid A biosynthesis reporter scores increase and d-glucuronate (carbohydrate) degradation scores decrease across young-to-old aging groups
- Metadata rownames match abundance-table colnames (all(rownames(metadata) %in% colnames(KO_abundance)) is TRUE, or at least partial matching via intersect > 0)
- Input table retains background KOs (not prefiltered), consistent with GRSA's threshold-free design
- Per-module results include both a reporter score sign and an adjusted p-value (p.adjust.method2) enabling ranking of enriched pathways per aging group
- Exported tables from export_report_table and figures from plot_report are internally consistent (plot IDs/descriptions correspond to ranked modules in the result table)

## Limitations

- GRSA exhibits slightly lower specificity than alternatives, so enrichment results require additional experimental validation
- There is no established gold standard for comparing enrichment methods, and specificity cannot be accurately evaluated because true negatives are hard to define due to incomplete databases
- No version number is stated for the ReporterScore package in the article, limiting reproducibility
- Built-in KEGG databases are from the 2023-08 version; using update_KO_file() to refresh via the KEGG API is recommended, so results depend on database vintage
- Mechanisms behind aging-related skin microbiota functional changes (e.g. IHSMGC findings) require further study — reporter scores show association, not causation

## Evidence

- [methods] reporter_score(kodf, group, metadata, method, pattern, p.adjust.method1, mode, verbose, feature, type, p.adjust.method2, modulelist, threads, perm, min_exist_KO, max_exist_KO): "reporter_score(kodf, group, metadata, method, pattern, p.adjust.method1, mode, verbose, feature, type, p.adjust.method2, modulelist, threads, perm, min_exist_KO, max_exist_KO)"
- [readme] the input abundance table should not be prefiltered to retain the background information, as the 'GRSA' is a threshold-free method: [evidence span withheld — anchor status: external_doc]
- [readme] the 'GRSA' supports **multi-group and longitudinal experimental designs**, because of the included multi-group-compatible statistical methods: [evidence span withheld — anchor status: external_doc]
- [discussion] applying GRSA on the IHSMGC dataset suggested different functional profiles between aging and young skin microbiota: "applying GRSA on the IHSMGC dataset suggested different functional profiles between aging and young skin microbiota."
- [intro] RSA is often misused due to a lack of specific tools and a systematic understanding of the algorithm: "However, RSA is often misused due to a lack of specific tools and a systematic understanding of the algorithm"
- [readme] ReporterScore has built-in KEGG pathway-KO and module-KO databases (2023-08 version) for KO abundance table: [evidence span withheld — anchor status: external_doc]
