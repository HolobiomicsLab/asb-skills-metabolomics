---
name: cross-study-gene-intersection-filtering
description: Use when pooling RNA-Seq/microarray data from multiple studies (e.g.
  TCGA + GTEx via PMR) and you must restrict the feature matrix to genes reported
  in every contributing study, plus drop undersampled organ types and samples lacking
  metadata.
when_to_use_negative:
- The input is a single-study dataset with no cross-study gene reporting discrepancies,
  so an intersection step would remove nothing informative
- Samples are already deduplicated and every gene is annotated in all contributing
  studies — rebuilding the intersection would be redundant
- The analysis goal is method development on raw sequencing data rather than curated
  expression matrices
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MetaOmGraph
  role: Java workbench in which the filtered 18,212-gene x 7,412-sample matrix and
    its metadata are assembled and saved as a shareable MOG project
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: Java
  role: Runtime environment (JRE 8+) required to run the MetaOmGraph main class and
    build the MOG project
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: R
  role: Optional companion for additional analyses of data subsets sent from MOG
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:e71b3415ba1574741ece3eb7c91e5eff23c50a204f4963eedcef9154fbaabf92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - Java
  - MetaOmGraph
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Cross-study gene intersection filtering

## Summary

Assemble a pooled multi-study expression dataset (e.g. the Hu-cancer-RNASeq-dataset from the Wang et al. combined TCGA/GTEx RNA-Seq resource) by intersecting genes across all contributing studies and applying sample-level filters, then encode the result as a MetaOmGraph (MOG) project for interactive exploratory analysis.

## When to use

Use when pooling RNA-Seq/microarray data from multiple studies (e.g. TCGA + GTEx via PMR) and you must restrict the feature matrix to genes reported in every contributing study, plus drop undersampled organ types and samples lacking metadata.

## When NOT to use

- The input is a single-study dataset with no cross-study gene reporting discrepancies, so an intersection step would remove nothing informative
- Samples are already deduplicated and every gene is annotated in all contributing studies — rebuilding the intersection would be redundant
- The analysis goal is method development on raw sequencing data rather than curated expression matrices

## Inputs

- Combined multi-study RNA-Seq expression matrix from the TCGA/GTEx resource (PMR)
- Sample metadata table (organ, tumor status, study source; 17 samples lack required metadata)
- Per-study gene reporting lists used to define the cross-study gene intersection

## Outputs

- Filtered expression matrix of 18,212 genes x 7,412 samples spanning 14 cancer types with matched non-tumor tissues
- Saved MetaOmGraph (.mog) project encoding the filtered matrix with attached gene and sample metadata

## How to apply

Retrieve the combined expression matrix and sample metadata from the PMR resource, then apply the paper's filters in order: exclude organ types with fewer than 30 tumor or 30 non-tumor samples; remove duplicate TCGA samples by retaining exactly one tumor sample per patient; drop the 17 samples lacking required metadata; intersect genes across all contributing studies and remove the 1,870 genes not reported in every study. The rationale is to guarantee a complete, comparable feature space and balanced organ representation before building the MOG project. Attach gene metadata and sample metadata (organ, tumor status, study source) to the filtered expression matrix within MetaOmGraph's data structure, then save the MOG project (an 18,212 genes x 7,412 samples matrix covering 14 cancer types with matched non-tumor tissues). Validate by confirming the constructed project has exactly 18,212 gene rows and 7,412 sample columns; the paper additionally removed one LUAD outlier sample whose Pearson correlation fell below the >0.70 within-condition threshold.

## Related tools

- **MetaOmGraph** (Java workbench in which the filtered 18,212-gene x 7,412-sample matrix and its metadata are assembled and saved as a shareable MOG project) — https://github.com/urmi-21/MetaOmGraph/
- **Java** (Runtime environment (JRE 8+) required to run the MetaOmGraph main class and build the MOG project) — https://github.com/urmi-21/MetaOmGraph/
- **R** (Optional companion for additional analyses of data subsets sent from MOG)

## Examples

```
git clone https://github.com/urmi-21/MetaOmGraph.git
```

## Evaluation signals

- The constructed project matrix contains exactly 18,212 gene rows, matching the reported Hu-cancer-RNASeq-dataset dimension
- The constructed project matrix contains exactly 7,412 sample columns, matching the reported dimension
- All retained organ types have at least 30 tumor and 30 non-tumor samples in the pooled TCGA/GTEx resource
- Exactly one TCGA tumor sample per patient is retained and the 17 samples lacking required metadata are excluded
- Every remaining gene is reported in all contributing studies (the 1,870 incompletely reported genes are absent)

## Limitations

- One LUAD sample fell below the >0.70 within-organ/tumor-status Pearson correlation threshold and had to be removed, indicating residual batch/outlier effects even after filtering
- The strict intersection criterion removes 1,870 genes that are incompletely reported in even a single contributing study, sacrificing coverage for comparability
- Organ types with fewer than 30 tumor or 30 non-tumor samples are excluded entirely, limiting cancer-type coverage to the 14 types retained

## Evidence

- [results] PMR source and dataset scale: "Data and metadata were retrieved from PMR (61)."
- [results] Final matrix dimension: "In each type of cancer in the Hu-cancer-RNASeq-dataset, between 2000–5000 of the 18 212 genes are differentially expressed"
- [results] LUAD outlier removal: [evidence span withheld — anchor status: not_found]
- [readme] MOG purpose and project saving: "MOG is a Java software to interactively explore and visualize large datasets."
- [supplementary] Source code availability: "MOG’s source code and user guide is available at https://github.com/urmi-21/MetaOmGraph/."
