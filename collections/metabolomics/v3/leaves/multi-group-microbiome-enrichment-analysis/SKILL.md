---
name: multi-group-microbiome-enrichment-analysis
description: Use when input is an unfiltered KO (or gene/compound) abundance table
  with metadata containing ≥2 groups, time points, or a continuous variable, and pathway/module-level
  enrichment is the goal.
when_to_use_negative:
- The abundance table has been prefiltered to a background gene list — GRSA is threshold-free
  and requires all background features retained
- The design is a simple two-condition set-based test where a standard hypergeometric/GSEA
  tool is adequate and multi-group compatibility is not needed
- The features are not mappable to the built-in KEGG pathway/module, compound, or
  GO databases and no custom modulelist can be constructed
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: ReporterScore
  role: 'Implements GRSA: reporter_score() computes per-KO differential statistics
    and aggregates them to pathway-level reporter scores against the KEGG modulelist;
    includes export and visualization functions'
  repo: https://github.com/Asa12138/ReporterScore
- name: devtools
  role: Used to install the ReporterScore package and its pcutils dependency from
    GitHub
- name: pcutils
  role: Dependency installed from GitHub before installing the ReporterScore development
    version
  repo: github.com/Asa12138/pcutils
- name: R
  role: Runtime environment in which the ReporterScore package and its workflow functions
    are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae116
    title: Generalized reporter score-based enrichment analysis for omics data
  source_content_hash: sha256:d539a7ae7dcba462c0cf47bf76c85ed06ee4ff23313d2eaad67eb1c386ef5791
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
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# multi-group-microbiome-enrichment-analysis

## Summary

Applies Generalized Reporter Score-based Analysis (GRSA) via the R package ReporterScore to detect KEGG pathway/module enrichment in KO abundance tables under multi-group, longitudinal, or correlation designs, e.g. re-analyzing skin microbiota (IHSMGC) functional shifts across five aging groups.

## When to use

Use when input is an unfiltered KO (or gene/compound) abundance table with metadata containing ≥2 groups, time points, or a continuous variable, and pathway/module-level enrichment is the goal.

## When NOT to use

- The abundance table has been prefiltered to a background gene list — GRSA is threshold-free and requires all background features retained
- The design is a simple two-condition set-based test where a standard hypergeometric/GSEA tool is adequate and multi-group compatibility is not needed
- The features are not mappable to the built-in KEGG pathway/module, compound, or GO databases and no custom modulelist can be constructed

## Inputs

- Unfiltered KO abundance table (kodf) with KEGG K numbers as rownames and samples as colnames (read counts or normalized values such as TPM/FPKM/relative abundance)
- Sample metadata table with rownames matching abundance-table colnames and a grouping column (categories, time points, or continuous)
- Built-in KEGG pathway-KO and module-KO databases (2023-08 version), optionally updated via update_KO_file()

## Outputs

- reporter_score_res object with pathway/module-level reporter scores and adjusted p-values per group
- Exported report tables from export_report_table (per-KO statistics and pathway-level results)
- Enrichment visualizations from plot_report and plot_enrich_res

## How to apply

Load the KO abundance table (kodf, rows = K numbers like K03169, columns = samples) and metadata whose rownames match the table colnames; do NOT prefilter the abundance table since GRSA is threshold-free. Install via install.packages('ReporterScore') from CRAN or devtools::install_github('Asa12138/pcutils') then devtools::install_github('Asa12138/ReporterScore'). Run reporter_score(kodf, group, metadata) with its filters (p.adjust.method1, p.adjust.method2, min_exist_KO, max_exist_KO, perm) against the built-in KEGG modulelist to compute per-KO statistics (ko.test) and aggregate them to pathway-level reporter scores (get_reporter_score). Rank enriched modules by reporter score sign and adjusted p-value per group, export with export_report_table(reporter_res, dir_name, overwrite), and visualize with plot_report (rs_threshold, mode, show_ID, Pathway_description) and plot_enrich_res. Optionally update pathway databases with update_KO_file() via the KEGG API before running. Validate by confirming recovered directionality, e.g. lipid A biosynthesis scores increasing and d-glucuronate degradation scores decreasing across young-to-old aging groups in the IHSMGC skin microbiota data.

## Related tools

- **ReporterScore** (Implements GRSA: reporter_score() computes per-KO differential statistics and aggregates them to pathway-level reporter scores against the KEGG modulelist; includes export and visualization functions) — https://github.com/Asa12138/ReporterScore
- **devtools** (Used to install the ReporterScore package and its pcutils dependency from GitHub)
- **pcutils** (Dependency installed from GitHub before installing the ReporterScore development version) — github.com/Asa12138/pcutils
- **R** (Runtime environment in which the ReporterScore package and its workflow functions are executed)

## Examples

```
devtools::install_github("Asa12138/pcutils"); devtools::install_github("Asa12138/ReporterScore"); reporter_score(kodf, group, metadata, p.adjust.method1="BH", p.adjust.method2="BH")
```

## Evaluation signals

- rownames(metadata) match (at least partially) colnames(kodf); intersect() > 0
- The abundance table was NOT prefiltered — GRSA requires the full background feature set
- Reproduced IHSMGC run recovers reported directionality: lipid A biosynthesis reporter scores increase and d-glucuronate/carbohydrate degradation scores decrease across young-to-old aging groups; absence flags the run for review
- Reporter scores and adjusted p-values (p.adjust.method2 with min_exist_KO/max_exist_KO filters) are present for all pathways/modules in the exported report table
- Database freshness: KOlist was checked with load_KO_list() and updated via update_KO_file() if the analysis requires the latest KEGG

## Limitations

- GRSA exhibits slightly lower specificity, requiring additional experimental validation of enrichment results
- There is no established gold standard for comparing enrichment analysis methods, and specificity cannot be accurately evaluated because true negatives are hard to define due to incomplete databases
- No version number is stated for the ReporterScore package in the paper, limiting reproducibility; underlying mechanisms of aging-related skin microbiota functional changes remain to be studied

## Evidence

- [abstract] GRSA is threshold-free and multi-group compatible: "Here, we propose the Generalized Reporter Score-based Analysis (GRSA) method for multi-group and longitudinal omics data."
- [methods] Installation from GitHub: "devtools::install_github("Asa12138/ReporterScore")"
- [methods] reporter_score signature and filters: "reporter_score(kodf, group, metadata, method, pattern, p.adjust.method1, mode, verbose, feature, type, p.adjust.method2, modulelist, threads, perm, min_exist_KO, max_exist_KO)"
- [readme] Do not prefilter input: [evidence span withheld — anchor status: external_doc]
- [readme] Sample matching requirement: [evidence span withheld — anchor status: external_doc]
- [discussion] IHSMGC aging application: "applying GRSA on the IHSMGC dataset suggested different functional profiles between aging and young skin microbiota."
- [discussion] Lower specificity caveat: "GRSA exhibits a slightly lower specificity, underscoring the necessity for additional experimental validations."
