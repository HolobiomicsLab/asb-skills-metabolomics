---
name: rna-seq-dataset-curation
description: Use when building a curated tumor/non-tumor RNA-Seq expression matrix
  from the combined Wang et al. TCGA/GTEx resource and packaging it as a shareable
  MOG project with metadata.
when_to_use_negative:
- The expression data are already a curated MOG project or have already passed the
  sample/organ/gene filtering described here.
- The goal is downstream statistical analysis (differential expression, co-expression)
  on an existing matrix — use MOG's analysis features instead of re-curation.
- The source resource is not the pooled TCGA/GTEx PMR compilation, so the per-study
  gene intersection and one-sample-per-patient deduplication rules do not apply.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MetaOmGraph
  role: Java workbench used to load the filtered expression matrix, attach gene/sample
    metadata, and save the shareable MOG project file
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: Java
  role: Runtime (JRE 8+) required to build and run the MetaOmGraph project from the
    cloned repository
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: PMR
  role: Source resource from which the combined TCGA/GTEx RNA-Seq data and metadata
    were retrieved
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:37b8097d3c43870be147408d927a38a044b10552565f6646fe26199324c80618
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - Java
  - MetaOmGraph
  - PMR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# rna-seq-dataset-curation

## Summary

Assemble a curated multi-cancer human RNA-Seq expression dataset by applying the paper's reproducible filtering pipeline to the pooled TCGA/GTEx resource from PMR, then package it as an 18,212 genes x 7,412 samples MetaOmGraph (MOG) project.

## When to use

Use when building a curated tumor/non-tumor RNA-Seq expression matrix from the combined Wang et al. TCGA/GTEx resource and packaging it as a shareable MOG project with metadata.

## When NOT to use

- The expression data are already a curated MOG project or have already passed the sample/organ/gene filtering described here.
- The goal is downstream statistical analysis (differential expression, co-expression) on an existing matrix — use MOG's analysis features instead of re-curation.
- The source resource is not the pooled TCGA/GTEx PMR compilation, so the per-study gene intersection and one-sample-per-patient deduplication rules do not apply.

## Inputs

- Combined TCGA/GTEx RNA-Seq expression matrix (Wang et al. resource) from PMR
- Sample metadata from PMR (organ, tumor status, study source, patient ID)
- Per-study gene reporting lists (to intersect genes across contributing studies)

## Outputs

- Filtered expression matrix of 18,212 genes x 7,412 samples covering 14 cancer types
- MetaOmGraph (MOG) project file encoding the expression matrix with gene and sample metadata
- Curation log of excluded entities (organ types with <30 samples, duplicate TCGA samples, 17 samples without metadata, 1,870 incompletely reported genes, 1 LUAD outlier sample)

## How to apply

Retrieve the combined TCGA/GTEx RNA-Seq expression data and sample metadata from the PMR resource, then exclude organ types with fewer than 30 tumor or 30 non-tumor samples. Remove TCGA non-tumor samples, retain exactly one tumor sample per TCGA patient, and drop the 17 samples lacking required metadata; additionally remove the single LUAD sample flagged by low pairwise Pearson correlation (samples in the same organ/tumor group otherwise correlate >0.70). Intersect genes across contributing studies and discard the 1,870 genes not reported in every study, yielding the 18,212-gene matrix. Attach gene metadata and sample metadata (organ, tumor status, study source) in MetaOmGraph's data structure and save the project via the Java-based MetaOmGraph main class; validate that the final matrix has exactly 18,212 gene rows and 7,412 sample columns spanning 14 cancer types with matched non-tumor tissues.

## Related tools

- **MetaOmGraph** (Java workbench used to load the filtered expression matrix, attach gene/sample metadata, and save the shareable MOG project file) — https://github.com/urmi-21/MetaOmGraph/
- **Java** (Runtime (JRE 8+) required to build and run the MetaOmGraph project from the cloned repository) — https://github.com/urmi-21/MetaOmGraph/
- **PMR** (Source resource from which the combined TCGA/GTEx RNA-Seq data and metadata were retrieved)

## Examples

```
git clone https://github.com/urmi-21/MetaOmGraph.git
```

## Evaluation signals

- The constructed project matrix has exactly 18,212 gene rows and 7,412 sample columns.
- The dataset spans 14 cancer types, each with matched non-tumor tissues, after excluding organ types with fewer than 30 tumor or 30 non-tumor samples.
- Exactly one TCGA tumor sample per patient is retained and the 17 samples lacking required metadata are absent.
- 1,870 genes not reported in every contributing study are removed by the gene intersection step.
- Within-group pairwise Pearson correlations among samples of the same organ and tumor status exceed 0.70, with the one outlier LUAD sample removed.

## Limitations

- The curation rules are tied to the specific Wang et al. combined TCGA/GTEx PMR resource; thresholds (e.g., 30-sample organ minimum, 1,870 dropped genes) may not transfer to other resources.
- Genes incompletely reported across contributing studies (1,870) are discarded, so the matrix cannot be reused where those genes matter.
- MOG requires Java Runtime Environment 8 (or higher); R 3.4+ is optional, and saved projects depend on MOG's project-file format.

## Evidence

- [results] Data and metadata were retrieved from PMR (61).: "Data and metadata were retrieved from PMR (61)."
- [results] Luad outlier removal: [evidence span withheld — anchor status: not_found]
- [results] Dataset dimensions context: "In each type of cancer in the Hu-cancer-RNASeq-dataset, between 2000–5000 of the 18 212 genes are differentially expressed"
- [readme] MOG project saving: [evidence span withheld — anchor status: external_doc]
- [methods] Retrieving the tool: "git clone https://github.com/urmi-21/MetaOmGraph.git"
