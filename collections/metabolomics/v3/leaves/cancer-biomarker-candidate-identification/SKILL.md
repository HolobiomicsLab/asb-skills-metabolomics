---
name: cancer-biomarker-candidate-identification
description: Use when tumor vs non-tumor RNA-Seq expression data with sample metadata
  (tumor status, organ) must be mined for differentially expressed genes, and candidates
  should be triaged against The Human Protein Atlas prognostic-marker flags.
when_to_use_negative:
- The input is metabolomics or microarray data outside the curated RNA-Seq expression
  matrix with tumor/non-tumor metadata (e.g. the AT-metab-dataset case study, which
  uses Spearman co-expression instead)
- The goal is stage-stratified analysis (early vs late stage) rather than tumor vs
  non-tumor differential expression
- The dataset lacks sample metadata to define tumor and non-tumor groups, making group
  separation in MOG impossible
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3053
tools:
- name: MOG
  role: 'Interactive differential expression analysis: metadata-driven sample grouping,
    log2 transformation, Mann–Whitney U test with BH correction, fold-change filtering
    and visualization of tumor vs non-tumor expression'
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: MetaOmGraph
  role: Same workbench as MOG; projects with saved exploration history can be shared
    so the biomarker search is reproducible
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: R
  role: Optional recipient of data subsets from MOG for additional analyses
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:0b8974128eb243139b025276328fbd6788c8ceb9844137df99b643da61cddac2
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
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cancer-biomarker-candidate-identification

## Summary

Nominates putative cancer biomarker genes by identifying differentially expressed genes between tumor and non-tumor groups in large RNA-Seq expression data (e.g. the Hu-cancer-RNASeq-dataset) using MetaOmGraph (MOG), then cross-referencing candidates against experimentally evaluated prognostic markers from The Human Protein Atlas (THPA).

## When to use

Use when tumor vs non-tumor RNA-Seq expression data with sample metadata (tumor status, organ) must be mined for differentially expressed genes, and candidates should be triaged against The Human Protein Atlas prognostic-marker flags.

## When NOT to use

- The input is metabolomics or microarray data outside the curated RNA-Seq expression matrix with tumor/non-tumor metadata (e.g. the AT-metab-dataset case study, which uses Spearman co-expression instead)
- The goal is stage-stratified analysis (early vs late stage) rather than tumor vs non-tumor differential expression
- The dataset lacks sample metadata to define tumor and non-tumor groups, making group separation in MOG impossible

## Inputs

- Hu-cancer RNA-Seq expression dataset (18,212 genes across multiple cancer types)
- Sample metadata table with tumor status and organ fields
- The Human Protein Atlas (THPA) prognostic-marker flag list

## Outputs

- List of differentially expressed candidate biomarker genes (BH-corrected P < 10^-3, fold-change >= 2)
- CSV table of candidate genes with mean tumor expression, mean non-tumor expression, fold-change and BH-corrected P-value
- Sub-list of candidates flagged as prognostic markers by THPA

## How to apply

Load the RNA-Seq expression matrix and its sample metadata (tumor status, organ) into MetaOmGraph as a project, then use metadata filters to select the cancer type of interest (e.g. LIHC samples) and split them into tumor and non-tumor groups. Log2-transform expression values and run a Mann–Whitney U test between the two groups in MOG, applying Benjamini–Hochberg correction with a significance threshold of BH-corrected P-value < 10^-3. Retain only genes with fold-change >= 2 between the groups as differentially expressed candidates; in the paper's Hu-cancer dataset this identified 2000–5000 differentially expressed genes out of 18,212 per cancer type. Finally, intersect downregulated (or upregulated) candidates with genes flagged as prognostic markers by The Human Protein Atlas to nominate experimentally supported biomarker candidates (e.g. ARG1, CYP2C8, CYP3A4, CYP3A7 and CYP4A11 decreasing with LIHC progression). Export the selected genes' mean tumor/non-tumor expression, fold-change and corrected P-values to a CSV table for downstream validation.

## Related tools

- **MOG** (Interactive differential expression analysis: metadata-driven sample grouping, log2 transformation, Mann–Whitney U test with BH correction, fold-change filtering and visualization of tumor vs non-tumor expression) — https://github.com/urmi-21/MetaOmGraph/
- **MetaOmGraph** (Same workbench as MOG; projects with saved exploration history can be shared so the biomarker search is reproducible) — https://github.com/urmi-21/MetaOmGraph/
- **R** (Optional recipient of data subsets from MOG for additional analyses)

## Examples

```
java -jar MetaOmGraph.jar  # launch MOG, load Hu-cancer RNA-Seq project, select LIHC tumor/non-tumor groups, run Mann–Whitney U test (BH P<10^-3, FC>=2)
```

## Evaluation signals

- Output CSV contains fields for gene identifier, tumor expression, non-tumor expression, fold-change and BH-corrected P-value
- Every candidate marker satisfies fold-change >= 2 and BH-corrected P-value < 10^-3
- Differentially expressed gene count per cancer type falls within the paper's reported 2000–5000 range of the 18,212 genes
- Downregulated LIHC candidates recover ARG1, CYP2C8, CYP3A4, CYP3A7 and CYP4A11, each flagged as a prognostic marker in THPA
- Sample-level QC passed: pairwise Pearson correlations > 0.70 within each organ/tumor-status group after outlier removal

## Limitations

- One anomalous sample (a lung adenocarcinoma LUAD sample with low Pearson correlation) had to be removed before analysis; unfiltered batch effects can distort group comparisons
- Thresholds (fold-change >= 2, BH-corrected P < 10^-3) are specific to this paper's Mann–Whitney U test on log2-transformed data; other test choices or cutoffs will change the candidate list
- THPA prognostic flags cover only a subset of genes (122 of the paper's candidates were flagged), so many differentially expressed genes remain unevaluated prognostically

## Evidence

- [results] Differential expression test thresholds: "Mann–Whitney U test, on the log2 transformed data, is significant between the two groups (BH corrected P-value <10−3)"
- [results] Dataset scale of differential expression: "In each type of cancer in the Hu-cancer-RNASeq-dataset, between 2000–5000 of the 18 212 genes are differentially expressed"
- [results] Prognostic marker cross-reference: "122 are flagged as prognostic markers by The Human Protein Atlas (THPA)"
- [abstract] Metadata-interactive workflow: "Researchers, without coding, can interactively visualize and evaluate data in the context of its metadata"
- [results] Sample QC outlier removal: [evidence span withheld — anchor status: not_found]
