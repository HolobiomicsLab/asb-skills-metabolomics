---
name: tcga-gtex-sample-metadata-integration
description: Use when reproducing or extending a pooled TCGA/GTEx RNA-Seq compendium
  that requires sample-level filtering (organ-size cutoffs, duplicate removal, metadata
  completeness) and integration into MetaOmGraph's metadata-aware project structure.
when_to_use_negative:
- The input dataset is already a filtered MOG project (e.g. the published 18,212 x
  7,412 matrix) and only downstream analysis is needed
- The analysis targets a single-study expression table without pooled TCGA/GTEx structure,
  so organ-size, duplicate-sample, and cross-study gene-intersection filters do not
  apply
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MetaOmGraph
  role: Java workbench used to attach sample/gene metadata to the filtered expression
    matrix and save the shared MOG project
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: Java
  role: Runtime (JRE 8+) required to build and run the MOG .jar that constructs and
    saves the project
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:c93893018695e31a0a1a5e27976a35d3ebaca2b0a86ce09a2aa3ad2c1c86f3e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - Java
  - MetaOmGraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# tcga-gtex-sample-metadata-integration

## Summary

Assembles the Hu-cancer-RNASeq-dataset from the Wang et al. combined TCGA/GTEx RNA-Seq resource (PMR) by applying the paper's sample and gene filters and attaching organ/tumor-status/study metadata, then encoding the result as a savable MetaOmGraph (MOG) project of 18,212 genes x 7,412 samples.

## When to use

Use when reproducing or extending a pooled TCGA/GTEx RNA-Seq compendium that requires sample-level filtering (organ-size cutoffs, duplicate removal, metadata completeness) and integration into MetaOmGraph's metadata-aware project structure.

## When NOT to use

- The input dataset is already a filtered MOG project (e.g. the published 18,212 x 7,412 matrix) and only downstream analysis is needed
- The analysis targets a single-study expression table without pooled TCGA/GTEx structure, so organ-size, duplicate-sample, and cross-study gene-intersection filters do not apply

## Inputs

- Combined TCGA/GTEx RNA-Seq expression matrix from the Wang et al. resource retrieved via PMR
- Sample metadata (organ, tumor status, study source, patient identifiers)
- Gene metadata / annotations

## Outputs

- Filtered expression matrix of 18,212 genes x 7,412 samples covering 14 cancer types with matched non-tumor tissues
- Saved MetaOmGraph (MOG) project file, e.g. Hu-Cancer-18212-7412-RNASeq.mog, with integrated metadata and exploration history

## How to apply

Retrieve the combined TCGA/GTEx RNA-Seq expression matrix and sample metadata from PMR. Exclude organ types with fewer than 30 tumor or 30 non-tumor samples, remove TCGA non-tumor samples, retain exactly one TCGA tumor sample per patient, and drop the 17 samples lacking required metadata; also remove one LUAD sample failing the pairwise Pearson correlation >0.70 batch check. Intersect genes across contributing studies and remove the 1,870 genes not reported in every study, yielding the 18,212-gene matrix. Attach gene metadata and sample metadata (organ, tumor status, study source) in MetaOmGraph's metadata-integrated data structure, then save the MOG project (MOG runs as a Java .jar, with optional R 3.4+ for additional analyses). The rationale is that MOG combines indexing-based big-data handling with metadata so downstream exploration (correlations, differential expression) is possible without coding.

## Related tools

- **MetaOmGraph** (Java workbench used to attach sample/gene metadata to the filtered expression matrix and save the shared MOG project) — https://github.com/urmi-21/MetaOmGraph/
- **Java** (Runtime (JRE 8+) required to build and run the MOG .jar that constructs and saves the project)

## Examples

```
git clone https://github.com/urmi-21/MetaOmGraph.git
```

## Evaluation signals

- Constructed project matrix has exactly 18,212 gene rows and 7,412 sample columns
- Final dataset covers 14 cancer types, each with matched non-tumor tissue samples
- No organ type remains with fewer than 30 tumor or 30 non-tumor samples
- Exactly one TCGA tumor sample retained per patient and the 17 samples lacking metadata are absent
- All retained samples pass the pairwise Pearson correlation >0.70 within-organ/tumor-status check (the single LUAD outlier was removed)

## Limitations

- The 1,870 genes dropped for incomplete cross-study reporting reduce the resource to 18,212 genes, so genes absent in any contributing study cannot be analyzed
- Results depend on the PMR-hosted version of the Wang et al. combined TCGA/GTEx resource; upstream updates would change counts
- MOG requires Java Runtime Environment 8 or higher; R integration needs R 3.4 or higher and is optional

## Evidence

- [results] PMR data source for the cancer compendium: "Data and metadata were retrieved from PMR (61)."
- [results] Final gene count of the assembled dataset: "between 2000–5000 of the 18 212 genes are differentially expressed"
- [readme] MOG is a Java tool with metadata integration and savable projects: [evidence span withheld — anchor status: external_doc]
- [methods] Cloning the MOG repository to build the project: "git clone https://github.com/urmi-21/MetaOmGraph.git"
