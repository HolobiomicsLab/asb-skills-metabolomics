---
name: gene-expression-dataset-benchmarking
description: Use when you need to compare pathway enrichment methods on human disease
  gene expression datasets by measuring how highly each method ranks known target
  disease pathways.
when_to_use_negative:
- Input is not human disease gene expression data (e.g. metabolomic or microbiome
  KO/compound tables) — apply the GRSA application skill instead
- The goal is to assess specificity or true-negative rates of enrichment methods —
  the article states specificity cannot be accurately evaluated because true negatives
  are hard to define
- Only one dataset is available and cross-dataset median aggregation is not meaningful
  — single-dataset ranking lacks the benchmark's aggregate medians
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_2269
tools:
- name: ReporterScore
  role: Implements GRSA; reporter_score() produces reporter scores and adjusted P-values
    used as the method under benchmark
  repo: https://github.com/Asa12138/ReporterScore
- name: devtools
  role: Installs ReporterScore and pcutils from GitHub
- name: pcutils
  role: Dependency installed before ReporterScore
  repo: github.com/Asa12138/pcutils
- name: clusterProfiler
  role: Back-end for KO_fisher/KO_enrich Fisher over-representation comparison method
- name: GSVA
  role: Back-end for KO_gsva comparison method
- name: GSA
  role: Back-end for KO_gsa comparison method
- name: PADOG
  role: Permutation-based comparison method via KO_padog
- name: safe
  role: Permutation-based comparison method via KO_safe
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae116
    title: Generalized reporter score-based enrichment analysis for omics data
  source_content_hash: sha256:25f3f600768cff00e8bb8c88674b4b13b5243e37a7bfd1b3fe07cba253655b90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Asa12138/ReporterScore
  tools:
  - GSA
  - GSVA
  - KO_enrich
  - KO_gsa
  - KO_gsea
  - KO_gsva
  - KO_padog
  - KO_safe
  - PADOG
  - R
  - ReporterScore
  - clusterProfiler
  - devtools
  - pcutils
  - safe
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 20
---

# gene-expression-dataset-benchmarking

## Summary

Benchmarks the generalized reporter score-based analysis (GRSA, R package ReporterScore) against six enrichment methods (Fisher/KO_enrich, GSEA, GSVA, GSA, PADOG, SAFE) on the 24 human disease gene expression benchmark datasets, ranking target disease pathways by median rank and median adjusted P-value.

## When to use

Use when you need to compare pathway enrichment methods on human disease gene expression datasets by measuring how highly each method ranks known target disease pathways.

## When NOT to use

- Input is not human disease gene expression data (e.g. metabolomic or microbiome KO/compound tables) — apply the GRSA application skill instead
- The goal is to assess specificity or true-negative rates of enrichment methods — the article states specificity cannot be accurately evaluated because true negatives are hard to define
- Only one dataset is available and cross-dataset median aggregation is not meaningful — single-dataset ranking lacks the benchmark's aggregate medians

## Inputs

- 24 human disease gene expression benchmark datasets (Supplementary Table S2) and the GSEABenchmarkeR GEO list
- Known target disease pathway annotations per dataset
- R environment with devtools, pcutils, ReporterScore, clusterProfiler, GSA, GSVA, PADOG, safe

## Outputs

- benchmark_results.csv with one row per method and median_rank and median_padj columns
- Per-dataset target-pathway ranks and adjusted P-values for GRSA and the six competing methods

## How to apply

Load the 24 human disease gene expression datasets from Supplementary Table S2 and the GSEABenchmarkeR GEO list in R. Install ReporterScore via devtools::install_github("Asa12138/ReporterScore") (after pcutils) plus the suggested packages clusterProfiler, GSA, GSVA, PADOG and safe. For each dataset, run GRSA with reporter_score() to obtain reporter scores and adjusted P-values for all pathways, then run the competing methods on the same pathway-level inputs via KO_fisher/KO_enrich, KO_gsea, KO_gsva, KO_gsa, KO_padog and KO_safe. The rationale is that sensitivity can be evaluated by the rank of known target pathways (equation 12), whereas specificity cannot be accurately assessed because true negatives are hard to define. For every method-dataset pair, rank the known target pathways, compute the median target-pathway rank and median adjusted P-value, aggregate medians across the 24 datasets, and write the per-method comparison to benchmark_results.csv.

## Related tools

- **ReporterScore** (Implements GRSA; reporter_score() produces reporter scores and adjusted P-values used as the method under benchmark) — https://github.com/Asa12138/ReporterScore
- **devtools** (Installs ReporterScore and pcutils from GitHub)
- **pcutils** (Dependency installed before ReporterScore) — github.com/Asa12138/pcutils
- **clusterProfiler** (Back-end for KO_fisher/KO_enrich Fisher over-representation comparison method)
- **GSVA** (Back-end for KO_gsva comparison method)
- **GSA** (Back-end for KO_gsa comparison method)
- **PADOG** (Permutation-based comparison method via KO_padog)
- **safe** (Permutation-based comparison method via KO_safe)

## Examples

```
library(ReporterScore); reporter_score(kodf, group, metadata, method="wilcox.test", type="pathway")
```

## Evaluation signals

- benchmark_results.csv exists and contains one row per method with field_present median_rank and median_padj columns
- GRSA and PADOG, GSA, GSEA and SAFE all achieve median target-pathway ranks within the top 20%
- GRSA achieves the lowest median adjusted P-values of the target pathways across the 24 datasets
- Reproduced median ranks and adjusted P-values match the values reported in the article within tolerance (output_matches_reference)
- All seven methods are run on the same pathway-level inputs derived from the same dataset

## Limitations

- Specificity of enrichment methods cannot be accurately evaluated because true negatives are hard to define due to incompleteness of databases
- There is no established gold standard in the field for comparing enrichment analysis methods
- GRSA exhibits slightly lower specificity, requiring additional experimental validation

## Evidence

- [results] GRSA performed comparably to PADOG, GSA, GSEA and SAFE in ranking target pathways: [evidence span withheld — anchor status: not_found]
- [intro] benchmark against other popular enrichment methods: "We benchmarked GRSA against other popular enrichment methods across multiple datasets"
- [discussion] sensitivity evaluation rationale: "the sensitivity can be effectively evaluated (equation (12)). In contrast, it is difficult to define the true negatives due to the complexity of biological systems"
- [methods] KO_padog function signature: "KO_padog(reporter_res, verbose, perm, p.adjust.method, ...)"
- [readme] installation instructions: "devtools::install_github("Asa12138/pcutils")
devtools::install_github("Asa12138/ReporterScore")"
