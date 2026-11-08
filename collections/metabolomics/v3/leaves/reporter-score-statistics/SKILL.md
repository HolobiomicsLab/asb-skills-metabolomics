---
name: reporter-score-statistics
description: Use when a KO/gene/compound abundance table plus metadata (two-group,
  multi-group, continuous covariate, or longitudinal) needs pathway-level enrichment,
  and threshold-free aggregation with reporter scores is preferred over cutoff-based
  hypergeometric-style tests.
when_to_use_negative:
- The abundance table has already been prefiltered to significant/background features
  — GRSA is threshold-free and requires the full background
- Only a pre-defined hit list without full feature abundances and metadata is available
  — hypergeometric-style KO_enrich/KO_fisher on ko_stat may be the appropriate fallback
  instead of the reporter-score aggregation
- Specificity must be strictly guaranteed — the article notes GRSA exhibits slightly
  lower specificity and that true negatives are difficult to define due to incomplete
  databases
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: ReporterScore
  role: 'R package implementing GRSA: ko.test, pvalue2zs, reporter_score/get_reporter_score,
    KO_* comparator methods, plot_report visualization, and updatable KEGG/GO/organism
    pathway databases'
  repo: https://github.com/Asa12138/ReporterScore
- name: pcutils
  role: GitHub dependency installed alongside ReporterScore for the development version
  repo: https://github.com/Asa12138/pcutils
- name: devtools
  role: Used to install ReporterScore and pcutils from GitHub
- name: R
  role: Runtime environment for the ReporterScore package and ggplot2-based boxplot
    comparisons
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae116
    title: Generalized reporter score-based enrichment analysis for omics data
  source_content_hash: sha256:417675733f3bb9ec47595495169c7b7a0a5c2c3943f00fa1c8489e070ae51476
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Asa12138/ReporterScore
  tools:
  - R
  - ReporterScore
  - devtools
  - pcutils
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# reporter-score-statistics

## Summary

Generalized Reporter Score-based Analysis (GRSA), implemented in the R package ReporterScore, aggregates threshold-free feature-level statistics (KOs, genes, compounds) into pathway/module-level reporter scores for multi-group, longitudinal, and correlation omics designs, and benchmarks sensitivity against KO_enrich, KO_fisher, KO_gsea, KO_gsva, KO_padog, KO_safe, and KO_sea.

## When to use

Use when a KO/gene/compound abundance table plus metadata (two-group, multi-group, continuous covariate, or longitudinal) needs pathway-level enrichment, and threshold-free aggregation with reporter scores is preferred over cutoff-based hypergeometric-style tests.

## When NOT to use

- The abundance table has already been prefiltered to significant/background features — GRSA is threshold-free and requires the full background
- Only a pre-defined hit list without full feature abundances and metadata is available — hypergeometric-style KO_enrich/KO_fisher on ko_stat may be the appropriate fallback instead of the reporter-score aggregation
- Specificity must be strictly guaranteed — the article notes GRSA exhibits slightly lower specificity and that true negatives are difficult to define due to incomplete databases

## Inputs

- Feature abundance table (kodf): rows are K00001-style KO IDs (or gene symbols like PEX11A, or KEGG C-number compounds), columns are samples; counts or normalized values (TPM/FPKM/RPKM/relative abundance)
- Metadata table with rownames matching abundance-table colnames, containing a grouping variable (categorical, multi-time-point, or continuous)
- Pathway database: built-in KEGG pathway-KO/module-KO (2023-08) via load_KOlist, organism-specific (hsa/mmu) via custom_modulelist_from_org, GO via load_GOlist, or custom via custom_modulelist

## Outputs

- Pathway-level reporter score table with adjusted p-values (exportable, e.g. grsa_pregnancy_reporter_scores.csv)
- Per-dataset sensitivity and specificity values per method for benchmark comparison (grouped boxplots, e.g. Figure 2C-D)
- Pathway visualization plots (e.g. pathway_reporter_scores.png via plot_report)

## How to apply

Install ReporterScore via devtools::install_github("Asa12138/pcutils") then devtools::install_github("Asa12138/ReporterScore") (or CRAN). Do not prefilter the abundance table, since GRSA is threshold-free. Run ko.test on the kodf with the grouping/continuous metadata variable to get feature-level p-values, convert them with pvalue2zs, and call reporter_score/get_reporter_score with the KEGG modulelist (or custom_modulelist/custom_modulelist_from_org, e.g. mmu for mouse), applying p.adjust.method2 and min_exist_KO/max_exist_KO pathway-size filters. For benchmarking, define ground truth (e.g. pathways whose KO member list contains the knockout gene, following Nguyen et al.), run the comparator functions KO_enrich, KO_fisher, KO_gsea, KO_gsva, KO_padog, KO_safe, KO_sea with default padj_threshold, logFC_threshold, weight, and perm settings, then compute sensitivity (fraction of true-positive pathways recovered) and specificity per dataset and compare with grouped boxplots. Visualize pathway reporter scores with plot_report (e.g. bar mode showing pathway descriptions). GRSA is expected to be more sensitive than hypergeometric tests because it considers all KOs in a pathway rather than only a pre-defined list, though its specificity is slightly reduced.

## Related tools

- **ReporterScore** (R package implementing GRSA: ko.test, pvalue2zs, reporter_score/get_reporter_score, KO_* comparator methods, plot_report visualization, and updatable KEGG/GO/organism pathway databases) — https://github.com/Asa12138/ReporterScore
- **pcutils** (GitHub dependency installed alongside ReporterScore for the development version) — https://github.com/Asa12138/pcutils
- **devtools** (Used to install ReporterScore and pcutils from GitHub)
- **R** (Runtime environment for the ReporterScore package and ggplot2-based boxplot comparisons)

## Examples

```
devtools::install_github("Asa12138/pcutils"); devtools::install_github("Asa12138/ReporterScore"); res <- reporter_score(kodf, group, metadata, type="pathway", feature="ko", modulelist=load_KOlist()$pathway)
```

## Evaluation signals

- Output table contains one sensitivity and one specificity entry per method per dataset (9 mouse wild-type/knockout GEO datasets) with no missing method-dataset combinations
- GRSA attains the highest median sensitivity across the 9 datasets in the grouped boxplot comparison (Figure 2C-D), accepting slightly reduced specificity
- Metadata rownames match abundance-table colnames: all(rownames(metadata) %in% colnames(KO_abundance)) returns TRUE
- In the pregnancy-cohort application, steroid hormone biosynthesis and cortisol synthesis appear up-regulated and aromatic amino acid metabolism down-regulated with gestational age (Figure 5A-B)
- Differential analysis fits the intended design: two-group, multi-group, multi-time-point (longitudinal), or continuous (correlation) variable handled via ko.test's metadata argument

## Limitations

- GRSA exhibits slightly lower specificity, underscoring the necessity for additional experimental validations
- Specificity of enrichment methods cannot be accurately evaluated because true negatives are hard to define due to the incompleteness of databases; no established gold standard exists
- No version number is stated for the ReporterScore package in the article, limiting reproducibility; databases are versioned (2023-08 built-ins) and should be updated via update_KO_file/update_org_pathway
