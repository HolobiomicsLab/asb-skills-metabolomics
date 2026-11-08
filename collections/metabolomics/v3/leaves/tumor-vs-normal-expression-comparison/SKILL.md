---
name: tumor-vs-normal-expression-comparison
description: Use when you have an RNA-Seq expression dataset with sample metadata
  encoding tumor status and organ, and the goal is to identify genes differentially
  expressed between tumor and matched non-tumor groups within one cancer type.
when_to_use_negative:
- The comparison is between cancer stages rather than tumor vs. non-tumor status —
  MOG's stage analysis instead separates samples by organ and early (stage I/II) vs.
  late (stage III+) stages
- The dataset lacks sample metadata encoding tumor status and organ, so groups cannot
  be defined in MOG
- The goal is cross-study integration of multiple omics datasets rather than a single-dataset
  two-group differential expression test
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: MetaOmGraph
  role: Standalone Java workbench used to load the Hu-cancer RNA-Seq project, filter
    samples by metadata (tumor status, organ), log2-transform values and run the Mann–Whitney
    U differential expression test
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: MOG
  role: MOG performed the statistical analyses and visualizations for the tumor vs.
    non-tumor comparison exclusively
  repo: https://github.com/urmi-21/MetaOmGraph
- name: R
  role: 'Optional: data subsets can be sent to R for additional analyses beyond MOG''s
    built-in tests'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:2c6b7269c2816a209bca068e3054dd0a105274b9ca8b10c4f38a5be216a5cec0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - MOG
  - MetaOmGraph
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# tumor-vs-normal-expression-comparison

## Summary

Differential expression analysis between tumor and non-tumor samples within a cancer type (e.g., LIHC in the Hu-cancer-RNASeq-dataset), performed in MetaOmGraph (MOG) via Mann–Whitney U tests on log2-transformed values with Benjamini–Hochberg correction, to identify putative biomarker genes such as ARG1, CYP2C8, CYP3A4, CYP3A7, CYP4A11 and GPC3.

## When to use

Use when you have an RNA-Seq expression dataset with sample metadata encoding tumor status and organ, and the goal is to identify genes differentially expressed between tumor and matched non-tumor groups within one cancer type.

## When NOT to use

- The comparison is between cancer stages rather than tumor vs. non-tumor status — MOG's stage analysis instead separates samples by organ and early (stage I/II) vs. late (stage III+) stages
- The dataset lacks sample metadata encoding tumor status and organ, so groups cannot be defined in MOG
- The goal is cross-study integration of multiple omics datasets rather than a single-dataset two-group differential expression test

## Inputs

- Hu-cancer-RNASeq-dataset expression matrix (18,212 genes) with sample metadata (tumor status, organ)
- MOG project file built from the expression data and metadata
- Optional: gene list of putative biomarker candidates (e.g., GPC3, ARG1, CYP2A6, CYP2C8)

## Outputs

- List of differentially expressed genes (BH-corrected Mann–Whitney U P < 10^-3, fold-change >= 2) between tumor and non-tumor groups
- CSV table (e.g., lihc_biomarker_expression_table.csv) with gene identifier, tumor expression, non-tumor expression, fold-change and BH-corrected P-value

## How to apply

Load the Hu-cancer RNA-Seq expression data and its sample metadata (tumor status, organ) into MetaOmGraph as a MOG project, then use metadata filtering to select the samples of one cancer type (e.g., LIHC) and split them into tumor and non-tumor groups. Log2 transform the expression values, then run MOG's differential expression analysis using a Mann–Whitney U test between the two groups, applying Benjamini–Hochberg correction with significance threshold BH-corrected P-value < 10^-3. Filter the resulting differentially expressed genes to those with fold-change >= 2 between tumor and non-tumor groups; MOG reported between 2000–5000 differentially expressed genes of 18,212 per cancer type with these criteria. For biomarker prioritization, additionally check flagged prognostic markers in The Human Protein Atlas (THPA) and extract rows of interest (e.g., GPC3, ARG1, CYP2A6, CYP2C8) with mean tumor/non-tumor expression, fold-change and corrected P-values into a CSV table.

## Related tools

- **MetaOmGraph** (Standalone Java workbench used to load the Hu-cancer RNA-Seq project, filter samples by metadata (tumor status, organ), log2-transform values and run the Mann–Whitney U differential expression test) — https://github.com/urmi-21/MetaOmGraph/
- **MOG** (MOG performed the statistical analyses and visualizations for the tumor vs. non-tumor comparison exclusively) — https://github.com/urmi-21/MetaOmGraph
- **R** (Optional: data subsets can be sent to R for additional analyses beyond MOG's built-in tests)

## Examples

```
git clone https://github.com/urmi-21/MetaOmGraph.git && java -jar MetaOmGraph.jar  # load Hu-cancer project, filter LIHC tumor/non-tumor, Mann–Whitney U, BH < 10^-3
```

## Evaluation signals

- Output CSV exists and contains fields for gene identifier, tumor expression, non-tumor expression, fold-change and BH-corrected P-value
- Every reported biomarker satisfies fold-change >= 2 and BH-corrected P-value < 10^-3
- Differentially expressed gene counts fall in the reported range of 2000–5000 out of 18,212 genes for the cancer type analyzed
- Group assignment matches sample metadata: tumor vs. non-tumor labels derived from the MOG project metadata for the selected organ (LIHC)
- Direction of change is consistent with prior findings, e.g., ARG1, CYP2C8, CYP3A4, CYP3A7 and CYP4A11 decreasing with LIHC progression

## Limitations

- Batch/sample-quality effects must be checked beforehand: the paper removed one LUAD sample after finding pairwise Pearson correlations below the >0.70 within-condition expectation
- The Mann–Whitney U test is run on log2-transformed data; applying it to raw counts is out of scope of the reported workflow
- Identifying a gene as differentially expressed does not establish prognostic value — separate validation against THPA-flagged prognostic markers was needed

## Evidence

- [results] Differential expression significance criterion: "Mann–Whitney U test, on the log2 transformed data, is significant between the two groups (BH corrected P-value <10−3)"
- [results] MOG used for the statistical analyses: "The statistical analyses and visualizations shown were generated exclusively using MOG."
- [abstract] Interactivity and statistical scope of MOG: "Statistical analyses include co-expression analysis, differential expression analysis and differential correlation analysis, with significance tests."
- [readme] MOG prerequisites and executable: "* Java Runtime Environment 8 (or higher)
* R 3.4 (or higher) [optional]"
- [results] Sample-quality check before analysis: "we used MOG to compute pairwise Pearson correlations among samples from the same biological condition"
