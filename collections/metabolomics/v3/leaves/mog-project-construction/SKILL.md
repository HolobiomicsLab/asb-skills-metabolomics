---
name: mog-project-construction
description: Use when raw combined TCGA/GTEx RNA-Seq expression data plus sample metadata
  must be filtered (organ-type sample counts, duplicate TCGA samples, missing metadata,
  incompletely reported genes) and packaged as a reusable MOG project.
when_to_use_negative:
- The input is already a saved MOG project — reload it directly rather than reconstructing
  the dataset
- The goal is downstream analysis (differential expression, co-expression) rather
  than building the dataset — load the existing MOG project and use MOG's statistical
  analyses
- Working with datasets other than the combined TCGA/GTEx Hu-cancer resource (e.g.
  the AT-microarray or AT-metab datasets), where these organ/sample/gene filters do
  not apply
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MetaOmGraph
  role: Java workbench used to assemble the filtered expression matrix with metadata
    into a savable/shareable MOG project file
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: Java
  role: Runtime environment (Java 8 or higher) required to run the MOG main class
    and save the project
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: R
  role: Optional companion environment MOG can send data subsets to for additional
    analyses during project exploration
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:8f76e744dd2bf9c1b8b7a7e0a49c80754d4d84744a949c26f8f4c4357485cec6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - Java
  - MetaOmGraph
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# mog-project-construction

## Summary

Assemble a filtered combined TCGA/GTEx human cancer RNA-Seq expression matrix (the Hu-cancer-RNASeq-dataset: 18,212 genes × 7,412 samples over 14 cancer types) into a savable MetaOmGraph (MOG) project with integrated gene and sample metadata.

## When to use

Use when raw combined TCGA/GTEx RNA-Seq expression data plus sample metadata must be filtered (organ-type sample counts, duplicate TCGA samples, missing metadata, incompletely reported genes) and packaged as a reusable MOG project.

## When NOT to use

- The input is already a saved MOG project — reload it directly rather than reconstructing the dataset
- The goal is downstream analysis (differential expression, co-expression) rather than building the dataset — load the existing MOG project and use MOG's statistical analyses
- Working with datasets other than the combined TCGA/GTEx Hu-cancer resource (e.g. the AT-microarray or AT-metab datasets), where these organ/sample/gene filters do not apply

## Inputs

- Combined TCGA/GTEx RNA-Seq expression data from the PMR resource
- Sample metadata accompanying the PMR RNA-Seq resource (organ, tumor status, study source)
- Gene metadata for annotating the expression matrix

## Outputs

- MOG project file encoding an 18,212 genes × 7,412 samples Hu-cancer-RNASeq-dataset
- Filtered expression matrix with gene and sample metadata attached (14 cancer types with matched non-tumor tissues)

## How to apply

Retrieve the combined TCGA/GTEx RNA-Seq expression data and sample metadata from PMR. Apply the paper's exclusion filters: drop organ types with fewer than 30 tumor or 30 non-tumor samples, remove TCGA non-tumor samples keeping exactly one tumor sample per patient, drop the 17 samples lacking required metadata, and remove one lung adenocarcinoma (LUAD) sample flagged by pairwise Pearson correlation QC (all same-condition samples should exceed 0.70). Intersect genes across contributing studies and remove the 1,870 genes not reported in every study, yielding the 18,212-gene matrix. Attach gene and sample metadata (organ, tumor status, study source) in MOG's data structure and save the project via the Java-based MetaOmGraph main class; saved MOG projects preserve the analysis state and can be shared. Validate that the final matrix has exactly 18,212 gene rows and 7,412 sample columns.

## Related tools

- **MetaOmGraph** (Java workbench used to assemble the filtered expression matrix with metadata into a savable/shareable MOG project file) — https://github.com/urmi-21/MetaOmGraph/
- **Java** (Runtime environment (Java 8 or higher) required to run the MOG main class and save the project) — https://github.com/urmi-21/MetaOmGraph/
- **R** (Optional companion environment MOG can send data subsets to for additional analyses during project exploration)

## Examples

```
git clone https://github.com/urmi-21/MetaOmGraph.git
```

## Evaluation signals

- Constructed project matrix contains exactly 18,212 gene rows and 7,412 sample columns
- All pairwise Pearson correlations among samples from the same organ and tumor status exceed 0.70 (the single outlier LUAD sample is removed)
- Organ types retained have at least 30 tumor and 30 non-tumor samples in the pooled TCGA/GTEx resource
- Each TCGA patient contributes exactly one tumor sample and no TCGA non-tumor samples remain
- Every retained gene is reported in all contributing studies (the 1,870 incompletely reported genes are absent)

## Limitations

- Filters are specific to the Wang et al. combined TCGA/GTEx RNA-Seq resource; sample and gene counts (17 samples, 1,870 genes) depend on that resource's state
- MOG requires a Java Runtime Environment 8 (or higher); R 3.4+ is optional but needed only for sending subsets to R
- The 18,212-gene matrix loses genes incompletely reported across contributing studies, reducing genomic completeness

## Evidence

- [results] PMR data source: "Data and metadata were retrieved from PMR (61)."
- [results] LUAD sample QC removal: [evidence span withheld — anchor status: not_found]
- [results] Dataset scale: "In each type of cancer in the Hu-cancer-RNASeq-dataset, between 2000–5000 of the 18 212 genes are differentially expressed"
- [readme] MOG project save/share: [evidence span withheld — anchor status: external_doc]
- [methods] Obtain MOG source: "git clone https://github.com/urmi-21/MetaOmGraph.git"
- [supplementary] Supporting data location: "Additional files are available at https://github.com/urmi-21/MetaOmGraph/tree/master/MOG_SupportingData."
