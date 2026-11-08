---
name: pathway-reporter-score-aggregation
description: Use when you have a feature abundance table (KO numbers, gene symbols,
  or KEGG C numbers) with sample metadata and need threshold-free, pathway/module-level
  enrichment with directionality across two-group, multi-group, or longitudinal designs.
when_to_use_negative:
- The input abundance table has been prefiltered to significant features — GRSA is
  threshold-free and requires background information
- The analysis requires per-feature significance testing only, without pathway-level
  aggregation (use ko.test alone)
- A hypergeometric-style over-representation analysis of a pre-defined significant
  list is specifically required (GRSA instead considers all KOs in a pathway for increased
  sensitivity)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3174
tools:
- name: ReporterScore
  role: 'R package implementing GRSA: ko.test, reporter_score, get_reporter_score,
    export_report_table, plot_report, plot_enrich_res'
  repo: https://github.com/Asa12138/ReporterScore
- name: pcutils
  role: Dependency installed alongside ReporterScore from GitHub
  repo: github.com/Asa12138/pcutils
- name: devtools
  role: Used to install ReporterScore and pcutils from GitHub
- name: R
  role: Runtime environment for the ReporterScore package and analysis scripts
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae116
    title: Generalized reporter score-based enrichment analysis for omics data
  source_content_hash: sha256:611240017027a09309570f5d116581107dd622c170c6f37ad9831f7a05eba07e
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# pathway-reporter-score-aggregation

## Summary

Aggregate per-feature (KO/gene/compound) differential statistics into pathway- or module-level reporter scores using the Generalized Reporter Score-based Analysis (GRSA) implemented in the R package ReporterScore, a threshold-free method supporting multi-group and longitudinal omics designs.

## When to use

Use when you have a feature abundance table (KO numbers, gene symbols, or KEGG C numbers) with sample metadata and need threshold-free, pathway/module-level enrichment with directionality across two-group, multi-group, or longitudinal designs.

## When NOT to use

- The input abundance table has been prefiltered to significant features — GRSA is threshold-free and requires background information
- The analysis requires per-feature significance testing only, without pathway-level aggregation (use ko.test alone)
- A hypergeometric-style over-representation analysis of a pre-defined significant list is specifically required (GRSA instead considers all KOs in a pathway for increased sensitivity)

## Inputs

- Feature abundance table (data.frame): rownames = feature IDs (KEGG K numbers, gene symbols, or C numbers); colnames = samples; values = counts or normalized abundances (TPM/FPKM/RPKM/relative abundance)
- Sample metadata data.frame: rownames = samples matching abundance colnames; one grouping column (categories, time points, or continuous)
- Pathway database (modulelist): built-in KEGG pathway-KO / module-KO (2023-08) or custom lists via custom_modulelist(), load_KOlist(), update_KO_file()

## Outputs

- reporter_score_res object with per-feature statistics and pathway/module-level reporter scores with adjusted p-values
- Exported enrichment tables via export_report_table (e.g. grsa_mode_comparison.csv)
- Visualization plots from plot_report and plot_enrich_res showing enriched modules and trend directions

## How to apply

Prepare an unfiltered feature abundance table (rownames as feature IDs such as K03169, colnames as samples) plus metadata whose rownames match the abundance table colnames; GRSA is threshold-free, so do not prefilter. Compute per-feature statistics with ko.test (method, p.adjust.method1), then run reporter_score(kodf, group, metadata, ...) with mode='directed' to convert p-values to signed statistics preserving within-pathway direction, or mode='mixed' when direction is not of interest; aggregation to pathway/module reporter scores via get_reporter_score uses filters p.adjust.method2, min_exist_KO, max_exist_KO, and a permutation-based null (perm) against the KEGG modulelist. Rank enriched pathways by reporter score sign and adjusted p-value, and interpret only directed-mode score signs as increasing/decreasing trends — in mixed mode opposing-trend KOs cancel and signs are uninterpretable. Export results with export_report_table and visualize with plot_report (rs_threshold, mode, show_ID, Pathway_description) and plot_enrich_res.

## Related tools

- **ReporterScore** (R package implementing GRSA: ko.test, reporter_score, get_reporter_score, export_report_table, plot_report, plot_enrich_res) — https://github.com/Asa12138/ReporterScore
- **pcutils** (Dependency installed alongside ReporterScore from GitHub) — github.com/Asa12138/pcutils
- **devtools** (Used to install ReporterScore and pcutils from GitHub)
- **R** (Runtime environment for the ReporterScore package and analysis scripts)

## Examples

```
reporter_score(kodf, group="Group", metadata=metadata, mode="directed", p.adjust.method1="BH", p.adjust.method2="BH", perm=1000)
```

## Evaluation signals

- Rownames of metadata match colnames of the abundance table (all(rownames(metadata) %in% colnames(KO_abundance)) is TRUE)
- Directed-mode enriched pathways show concordant within-pathway KO trend directions; mixed-mode reporter score signs are treated as uninterpretable
- Validation against known trends, e.g. on IHSMGC skin microbiota data, lipid A biosynthesis reporter scores increase and d-glucuronate (carbohydrate) degradation scores decrease across young-to-old aging groups
- Exported enrichment table contains both increasing- and decreasing-direction pathways with adjusted p-values below threshold and respects min_exist_KO/max_exist_KO filters
- Pathway scores remain insignificant when within-pathway KOs have opposing trends (signed Z-scores cancel) in directed mode — a sanity check on aggregation behavior

## Limitations

- In mixed mode, the increasing or decreasing trend of enriched pathways cannot be determined from reporter score signs because opposing-trend KOs cancel
- GRSA exhibits slightly lower specificity, underscoring the necessity for additional experimental validations
- No established gold standard exists for comparing enrichment methods; specificity cannot be accurately evaluated because true negatives are hard to define due to incomplete databases
- No version number is stated for the ReporterScore package in the article, limiting reproducibility; use update_KO_file() since using the latest pathway database is important

## Evidence

- [readme] GRSA is a threshold-free method that works well with all types of biomedical features: "'GRSA' is a threshold-free method that works well with all types of biomedical features, such as genes, chemical compounds, and microbial species."
- [methods] install from GitHub: "devtools::install_github("Asa12138/ReporterScore")
library(ReporterScore)"
- [readme] threshold-free input warning: [evidence span withheld — anchor status: external_doc]
- [results] mixed mode sign cancellation: [evidence span withheld — anchor status: not_found]
- [discussion] IHSMGC aging findings: "applying GRSA on the IHSMGC dataset suggested different functional profiles between aging and young skin microbiota."
- [discussion] sensitivity vs hypergeometric: "GRSA considers all KOs involved in the pathway compared to hypergeometric tests that only consider a pre-defined list"
