---
name: multi-method-enrichment-comparison
description: Use when evaluating or comparing pathway enrichment methods on gene/KO-level
  omics data with known ground-truth target pathways (knockout genes or disease pathways),
  e.g. reproducing Figure 2C-D sensitivity/specificity or ranking benchmarks.
when_to_use_negative:
- Input abundance table has been prefiltered to a short feature list, since GRSA is
  threshold-free and the README warns the table should retain background information
- The goal is a single-method enrichment run rather than a method comparison with
  ground-truth target pathways
- True negatives cannot be approximated (no knockout-gene or disease-pathway ground
  truth), since specificity of enrichment methods cannot be accurately evaluated
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3569
tools:
- name: ReporterScore
  role: Implements GRSA (reporter_score, get_reporter_score, ko.test) and comparator
    wrappers KO_enrich/KO_fisher/KO_gsea/KO_gsva/KO_padog/KO_safe/KO_sea used in the
    benchmark
  repo: https://github.com/Asa12138/ReporterScore
- name: pcutils
  role: Dependency installed before ReporterScore when using the GitHub development
    version
  repo: https://github.com/Asa12138/pcutils
- name: devtools
  role: Used to install ReporterScore and pcutils from GitHub
- name: clusterProfiler
  role: Suggested package used by KO_enrich/KO_fisher for Fisher over-representation
    (CP)
- name: GSVA
  role: Engine behind the KO_gsva comparator
- name: PADOG
  role: Permutation-based comparator wrapped by KO_padog
- name: safe
  role: Permutation-based comparator wrapped by KO_safe
- name: GSA
  role: Comparator wrapped by KO_gsa in the ranking benchmark
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae116
    title: Generalized reporter score-based enrichment analysis for omics data
  source_content_hash: sha256:2c31470623db40f8e1bce707631ddd8d06ae0cade4d56749ec50a5d18bfcc70a
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3569
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45
n_claims: 20
---

# multi-method-enrichment-comparison

## Summary

Benchmark the GRSA (Generalized Reporter Score-based Analysis) implemented in the ReporterScore R package against popular enrichment methods (Fisher/KO_enrich, KO_fisher, GSEA, GSVA, GSA, PADOG, SAFE, SEA) on mouse knockout and human disease gene expression datasets, measuring sensitivity, specificity, and target-pathway rank.

## When to use

Use when evaluating or comparing pathway enrichment methods on gene/KO-level omics data with known ground-truth target pathways (knockout genes or disease pathways), e.g. reproducing Figure 2C-D sensitivity/specificity or ranking benchmarks.

## When NOT to use

- Input abundance table has been prefiltered to a short feature list, since GRSA is threshold-free and the README warns the table should retain background information
- The goal is a single-method enrichment run rather than a method comparison with ground-truth target pathways
- True negatives cannot be approximated (no knockout-gene or disease-pathway ground truth), since specificity of enrichment methods cannot be accurately evaluated

## Inputs

- 9 wild-type/knockout mouse GEO gene expression datasets (Supplementary Table S3)
- 24 human disease gene expression datasets (Supplementary Table S2) and the GSEABenchmarkeR GEO list
- KO-level abundance table (rownames = K numbers, colnames = samples)
- Metadata table with sample group labels (WT vs KO)
- Pathway module lists from load_org_pathway/custom_modulelist (mmu pathway-KO/gene databases)

## Outputs

- Per-method sensitivity and specificity per dataset (grouped boxplots reproducing Figure 2C-D)
- benchmark_results.csv with one row per method and median_rank and median_padj columns
- Pathway-level reporter scores and adjusted P-values from reporter_score/get_reporter_score

## How to apply

Install ReporterScore (and pcutils) via devtools::install_github. For the knockout benchmark, preprocess 9 wild-type/knockout mouse GEO datasets (Supplementary Table S3), convert gene-level to KO-level abundance with gene2ko, run ko.test per dataset, and label pathways containing the knockout gene as true positives using load_org_pathway/custom_modulelist lists. Run GRSA via reporter_score/get_reporter_score plus comparators KO_enrich, KO_fisher, KO_gsea, KO_gsva, KO_padog, KO_safe, and KO_sea with default thresholds (padj_threshold, logFC_threshold, weight, perm), then compute sensitivity and specificity per method per dataset and draw grouped ggplot2 boxplots (Figure 2C-D). For the ranking benchmark, run GRSA and comparators (KO_gsa included) on 24 human disease datasets (Supplementary Table S2), compute median target-pathway rank and median adjusted P-value per method, and verify GRSA attains the highest median sensitivity while ranking within the top 20% like PADOG, GSA, GSEA and SAFE.

## Related tools

- **ReporterScore** (Implements GRSA (reporter_score, get_reporter_score, ko.test) and comparator wrappers KO_enrich/KO_fisher/KO_gsea/KO_gsva/KO_padog/KO_safe/KO_sea used in the benchmark) — https://github.com/Asa12138/ReporterScore
- **pcutils** (Dependency installed before ReporterScore when using the GitHub development version) — https://github.com/Asa12138/pcutils
- **devtools** (Used to install ReporterScore and pcutils from GitHub)
- **clusterProfiler** (Suggested package used by KO_enrich/KO_fisher for Fisher over-representation (CP))
- **GSVA** (Engine behind the KO_gsva comparator)
- **PADOG** (Permutation-based comparator wrapped by KO_padog)
- **safe** (Permutation-based comparator wrapped by KO_safe)
- **GSA** (Comparator wrapped by KO_gsa in the ranking benchmark)

## Examples

```
devtools::install_github("Asa12138/pcutils"); devtools::install_github("Asa12138/ReporterScore"); res <- reporter_score(kodf, group=metadata$Group); KO_padog(res)
```

## Evaluation signals

- Output table contains one sensitivity and one specificity entry per method per dataset across the 9 knockout datasets
- GRSA attains the highest median sensitivity among all methods in the Figure 2C-D grouped boxplot comparison
- benchmark_results.csv exists with one row per method and field_present median_rank and median_padj columns
- Median target-pathway ranks for GRSA, PADOG, GSA, GSEA and SAFE all fall within the top 20%, and GRSA has the lowest median adjusted P-values

## Limitations

- GRSA exhibits a slightly lower specificity, underscoring the necessity for additional experimental validations
- Specificity cannot be accurately evaluated because true negatives are hard to define due to incomplete pathway databases
- No established gold standard exists for comparing enrichment analysis methods

## Evidence

- [results] GRSA showed the highest median sensitivity among the methods considered, although its specificity is slightly reduced compared to others: "GRSA showed the highest median sensitivity among the methods considered, although its specificity is slightly reduced compared to others (Figure 2C, D)."
- [methods] install ReporterScore from GitHub: "devtools::install_github("Asa12138/ReporterScore")"
- [discussion] benchmark sensitivity discussion: "GRSA exhibits a slightly lower specificity, underscoring the necessity for additional experimental validations."
- [readme] threshold-free input warning: [evidence span withheld — anchor status: external_doc]
- [abstract] sensitivity abstract finding: "A comparison with other popular enrichment analysis methods demonstrated that GRSA had increased sensitivity across multiple benchmark datasets."
