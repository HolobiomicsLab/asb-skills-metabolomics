---
name: lipid-class-regulation-pattern-interpretation
description: Use when a lipidomics dataset with sample group annotations (e.g. cancer
  vs benign, cancer vs metastasis) has been differential-analyzed with lipidr and
  you need to summarize which lipid classes show consistent regulation.
when_to_use_negative:
- The dataset has more than two groups or confounding variables to adjust without
  defined pairwise contrasts — use lipidr's multi-group comparison and confounder-adjustment
  support instead of manual class-pattern reading
- The input is raw instrument output (e.g. an unprocessed Skyline export without sample
  annotation) rather than a normalized, annotated lipidr experiment object
- The goal is quantitative enrichment testing of lipid classes rather than descriptive
  pattern interpretation — use lipidr's LSEA instead
edam_operation: http://edamontology.org/operation_3662
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: lipidr
  role: R/Bioconductor package providing de_analysis (limma-based linear models on
    loaded clinical variables), volcano plot visualization, and LSEA for detecting
    class-level lipid regulation
  repo: https://github.com/ahmohamed/lipidr
- name: limma
  role: Underlying linear-modeling framework invoked by lipidr's de_analysis to fit
    two-group contrasts on lipid abundances
- name: Metabolomics Workbench API
  role: Source of public lipidomics datasets (e.g. study ST001111) that lipidr can
    download and reanalyze directly
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:c5093322c7c52cac408da3164aa29540ea1f447a177207ff245322c2ce32ad5f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ahmohamed/lipidr
  tools:
  - Metabolomics Workbench API
  - limma
  - lipidr
  edam_operation: http://edamontology.org/operation_3662
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- lipid-class-regulation-interpretation
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# lipid-class-regulation-pattern-interpretation

## Summary

Interprets lipid-class regulation patterns from limma-based two-group differential analysis in lipidr by reading per-lipid logFC/p-values and volcano plots to identify which lipid classes (e.g. PCs, PGs, CLs, TGs) are consistently up- or down-regulated between sample groups.

## When to use

Use when a lipidomics dataset with sample group annotations (e.g. cancer vs benign, cancer vs metastasis) has been differential-analyzed with lipidr and you need to summarize which lipid classes show consistent regulation.

## When NOT to use

- The dataset has more than two groups or confounding variables to adjust without defined pairwise contrasts — use lipidr's multi-group comparison and confounder-adjustment support instead of manual class-pattern reading
- The input is raw instrument output (e.g. an unprocessed Skyline export without sample annotation) rather than a normalized, annotated lipidr experiment object
- The goal is quantitative enrichment testing of lipid classes rather than descriptive pattern interpretation — use lipidr's LSEA instead

## Inputs

- LipidomicsExperiment object (lipidr, extends SummarizedExperiment) with outlier-removed lipid abundance matrix
- Sample annotation/clinical variables table (CSV, sample names in first column, matching the matrix columns)
- Contrast definitions as strings (e.g. 'Cancer-Benign', 'Cancer-Metastasis')

## Outputs

- CSV results table of logFC and p-values per lipid for each contrast
- Volcano plots per contrast (interactive lipidr visualizations)
- Interpreted lipid-class regulation pattern (which classes are up- or down-regulated per contrast)

## How to apply

Load the outlier-removed dataset as a LipidomicsExperiment object (extended from SummarizedExperiment) with sample annotations, then run lipidr's de_analysis with explicit contrasts (e.g. Cancer-Benign, Cancer-Metastasis), which fits limma-based linear models on the loaded clinical variables. Export the per-lipid logFC and p-values as a CSV results table and generate volcano plots for each contrast to visualize significance against fold change. At the class level, aggregate the direction of regulation per lipid class (e.g. PCs, PGs, CLs, TGs) to determine which classes are up- vs down-regulated in the condition of interest; in the ST001111 breast cancer example this yields PCs and PGs up-regulated and CLs and TGs down-regulated in cancer vs benign, with only minute differences between cancer and metastatic samples. For formal class-level enrichment rather than manual pattern reading, apply lipidr's lipid set enrichment analysis (LSEA), which detects preferential enrichment of lipid classes, total chain lengths, or unsaturation patterns.

## Related tools

- **lipidr** (R/Bioconductor package providing de_analysis (limma-based linear models on loaded clinical variables), volcano plot visualization, and LSEA for detecting class-level lipid regulation) — https://github.com/ahmohamed/lipidr
- **limma** (Underlying linear-modeling framework invoked by lipidr's de_analysis to fit two-group contrasts on lipid abundances)
- **Metabolomics Workbench API** (Source of public lipidomics datasets (e.g. study ST001111) that lipidr can download and reanalyze directly)

## Examples

```
d <- add_sample_annotation(as_lipidomics_experiment(read.csv("data_matrix.csv")), "data_clin.csv"); de_res <- de_analysis(d, Cancer - Benign, Cancer - Metastasis)
```

## Evaluation signals

- Exported CSV contains logFC and p-value columns for every lipid and for each requested contrast (Cancer-Benign and Cancer-Metastasis)
- Volcano plots render per contrast with lipids positioned by logFC vs significance, and are consistent with the CSV values
- Class-level direction check: PCs and PGs trend up-regulated and CLs and TGs trend down-regulated in the cancer-vs-benign contrast
- Cancer-vs-metastasis contrast shows only minute differences relative to cancer-vs-benign — a large inter-contrast difference is a red flag
- Sample names in the annotation table match the sample columns of the abundance matrix exactly (a hard requirement in lipidr's add_sample_annotation)

## Limitations

- Class-level interpretation from volcano plots is descriptive; preferential class enrichment should be confirmed with lipidr's LSEA
- Only minute differences may exist between closely related groups (e.g. cancer vs metastasis), so absent regulation signals are expected and not necessarily a failure
- The analysis depends on clinical/group variables being loaded into the experiment object beforehand; contrasts cannot be formed without them
- The dataset should be outlier-removed before de_analysis; unfiltered outliers can distort limma logFC estimates

## Evidence

- [readme] lipidr represents datasets as LipidomicsExperiment extending SummarizedExperiment: [evidence span withheld — anchor status: not_found]
- [readme] lipidr supports multi-group comparisons and confounder adjustment in univariate analysis: [evidence span withheld — anchor status: external_doc]
