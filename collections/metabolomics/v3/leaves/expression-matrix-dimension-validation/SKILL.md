---
name: expression-matrix-dimension-validation
description: Use when a filtered/assembled expression dataset must be checked against
  its published dimensions (e.g. 18,212 genes x 7,412 samples for the Hu-cancer-RNASeq-dataset)
  before MOG analyses.
when_to_use_negative:
- Input is an unrelated expression dataset (e.g. the AT-microarray-dataset or AT-metab-dataset)
  whose expected dimensions are not 18,212 x 7,412
- The raw pooled TCGA/GTEx resource has not yet passed through the article's filter
  steps (organ-type sample minimums, duplicate-sample removal, gene intersection),
  so dimension mismatches would be expected and uninformative
- The goal is statistical analysis of the matrix rather than structural validation
  of the assembled MOG project
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MetaOmGraph
  role: Java workbench used to load the filtered expression matrix with metadata,
    save the MOG project, and confirm the 18,212-gene x 7,412-sample structure
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: Java
  role: Runtime environment required to build and run the MetaOmGraph main class that
    constructs and validates the MOG project
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:98e8595d5b4a7b82dd85ce7c6cc557cd56081a5ed2c37db98976f074a3269445
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - Java
  - MetaOmGraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# expression-matrix-dimension-validation

## Summary

Validate that a constructed MetaOmGraph (MOG) project expression matrix has the expected reported dimensions — for the Hu-cancer-RNASeq-dataset, exactly 18,212 gene rows by 7,412 sample columns — so downstream MOG analyses operate on a correctly assembled dataset.

## When to use

Use when a filtered/assembled expression dataset must be checked against its published dimensions (e.g. 18,212 genes x 7,412 samples for the Hu-cancer-RNASeq-dataset) before MOG analyses.

## When NOT to use

- Input is an unrelated expression dataset (e.g. the AT-microarray-dataset or AT-metab-dataset) whose expected dimensions are not 18,212 x 7,412
- The raw pooled TCGA/GTEx resource has not yet passed through the article's filter steps (organ-type sample minimums, duplicate-sample removal, gene intersection), so dimension mismatches would be expected and uninformative
- The goal is statistical analysis of the matrix rather than structural validation of the assembled MOG project

## Inputs

- Filtered expression matrix built from the combined TCGA/GTEx RNA-Seq data retrieved from PMR
- Sample metadata (organ, tumor status, study source) and gene metadata attached to the matrix
- Expected dataset dimensions reported in the article (18,212 genes x 7,412 samples)

## Outputs

- Confirmed 18,212 x 7,412 gene-by-sample matrix within a saved MOG project file
- Validation report confirming 14 cancer types with matched non-tumor tissues and metadata completeness

## How to apply

After assembling the Hu-cancer-RNASeq-dataset in MOG — retrieving pooled TCGA/GTEx RNA-Seq data from PMR, excluding organ types with fewer than 30 tumor or 30 non-tumor samples, removing duplicate TCGA samples, dropping the 17 samples lacking metadata, and intersecting genes to remove the 1,870 incompletely reported genes — check that the final matrix contains exactly 18,212 gene rows and 7,412 sample columns spanning 14 cancer types with matched non-tumor tissues. Attach gene metadata and sample metadata (organ, tumor status, study source) before validation so the MOG project encodes the full dataset. Save the project via the Java-based MetaOmGraph main class and re-open it to confirm the dimensions persist in the saved MOG project file. As a biological sanity check, MOG can compute pairwise Pearson correlations among samples from the same organ and tumor status; the paper found all samples exceeded 0.70 except one LUAD sample that was removed.

## Related tools

- **MetaOmGraph** (Java workbench used to load the filtered expression matrix with metadata, save the MOG project, and confirm the 18,212-gene x 7,412-sample structure) — https://github.com/urmi-21/MetaOmGraph/
- **Java** (Runtime environment required to build and run the MetaOmGraph main class that constructs and validates the MOG project)

## Examples

```
git clone https://github.com/urmi-21/MetaOmGraph.git
```

## Evaluation signals

- The constructed project matrix has exactly 18,212 gene rows and 7,412 sample columns, matching the reported Hu-cancer-RNASeq-dataset dimensions
- Gene count reflects removal of the 1,870 genes not reported in every contributing study after intersecting genes across all studies
- Sample count reflects removal of organ types with <30 tumor or <30 non-tumor samples, duplicate TCGA samples, and the 17 samples lacking required metadata
- The dataset spans 14 cancer types, each with matched non-tumor tissues
- Pairwise Pearson correlations among same-organ, same-tumor-status samples exceed 0.70 (with the one LUAD outlier sample already removed)

## Limitations

- Dimension validation alone does not detect content errors — a correctly sized matrix could still contain mis-assigned metadata, so organ/tumor-status metadata should also be checked
- The Pearson correlation screen (>0.70) will flag but not diagnose batch effects; the article used it only to identify and remove a single LUAD outlier sample
- Validation depends on MOG's indexing and buffering; projects must be saved and reopened to confirm the structure persists in the shared MOG project file

## Evidence

- [other] workflow card validation step: [evidence span withheld — anchor status: not_found]
- [results] dataset dimensions in results: "In each type of cancer in the Hu-cancer-RNASeq-dataset, between 2000–5000 of the 18 212 genes are differentially expressed"
- [results] sample outlier removal: [evidence span withheld — anchor status: not_found]
- [abstract] MOG project saving: "MOG projects, with history of explorations, can be saved and shared."
- [readme] MOG efficient data handling: "MOG overcomes the challenges posed by big size and complexity of big datasets by efficient handling of data files by using a combination of data indexing and buffering schemes."
