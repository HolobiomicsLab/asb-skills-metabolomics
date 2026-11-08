---
name: web-tool-metabolite-list-submission
description: Use when you have a metabolite name/ID list from a metabolomics study
  and need group-level pathway enrichment results (top pathways + P values) to compare
  against published benchmark values.
when_to_use_negative:
- You need individual patient-level pathway matrices or pathway features for classification
  — MetPA, IMPaLA, and MPEA 'are not capable of generating such individual patient-level
  pathway matrices'; use lilikoi's PDS-based modules instead
- Your metabolite IDs are non-standard names that have not yet been standardized —
  map them first (e.g., to HMDB IDs) since ~20% of metabolites fail to map and cannot
  be enriched
- The analysis goal is sample-level pathway deregulation scoring rather than group-level
  enrichment P values
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: MetPA
  role: web tool combining pathway enrichment analysis with pathway topological characteristics;
    used to obtain the top significant pathway (aminoacyl-tRNA biosynthesis) and its
    P value
- name: IMPaLA
  role: web tool performing joint over-representation/enrichment pathway analysis;
    used to obtain the top significant pathway (biochemical pathways part I) and its
    P value
- name: MPEA
  role: GSEA-based pathway analysis tool handling many-to-many compound-pathway relationships;
    used to obtain the top significant pathway (protein digestion and absorption)
    and its P value
- name: lilikoi
  role: R package supplying the source dataset (plasma_breast_cancer.csv in extdata)
    and HMDB-standardized metabolite names submitted to the web tools
  repo: https://github.com/lanagarmire/lilikoi
- name: R
  role: scripting language used to load the metabolite measurements and extract the
    227 metabolite names from the lilikoi extdata file
  repo: https://cran.r-project.org/web/packages/lilikoi/index.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy136
    title: 'Lilikoi: an R package for personalized pathway-based classification modeling
      using metabolomics data'
  source_content_hash: sha256:09975b43e91d76f29e629a05ca1035e22c8d887bcc831ec148758cbd1d6c7c17
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi
  tools:
  - IMPaLA
  - MPEA
  - MetPA
  - R
  - lilikoi
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# web-tool-metabolite-list-submission

## Summary

Submits a curated metabolite list (e.g., 227 metabolites from plasma_breast_cancer.csv, 182 mapping to standard HMDB IDs) to the group-level enrichment web tools MetPA, IMPaLA, and MPEA to record each tool's top significant pathway and its enrichment P value, benchmarking against the paper's Supplementary Table S1.

## When to use

Use when you have a metabolite name/ID list from a metabolomics study and need group-level pathway enrichment results (top pathways + P values) to compare against published benchmark values.

## When NOT to use

- You need individual patient-level pathway matrices or pathway features for classification — MetPA, IMPaLA, and MPEA 'are not capable of generating such individual patient-level pathway matrices'; use lilikoi's PDS-based modules instead
- Your metabolite IDs are non-standard names that have not yet been standardized — map them first (e.g., to HMDB IDs) since ~20% of metabolites fail to map and cannot be enriched
- The analysis goal is sample-level pathway deregulation scoring rather than group-level enrichment P values

## Inputs

- metabolite name list extracted from plasma_breast_cancer.csv (227 metabolites, 182 mapped to standard HMDB IDs)
- standard metabolite identifiers (HMDB IDs) accepted by the MetPA, IMPaLA, and MPEA web interfaces

## Outputs

- top significant pathway name and enrichment P value per tool (MetPA: aminoacyl-tRNA biosynthesis P = 5.8e-09; IMPaLA: biochemical pathways part I P = 4.4e-34; MPEA: protein digestion and absorption P = 3.15e-20)
- compiled comparison table written to table_s1_enrichment_comparison.csv reproducing Supplementary Table S1

## How to apply

Load the metabolite measurements from the lilikoi package extdata file plasma_breast_cancer.csv and extract the 227 metabolite names, of which 182 map to standard HMDB IDs. Submit the same metabolite list to three web tools in turn: MetPA, which combines pathway enrichment analysis with pathway topological characteristics; IMPaLA, which performs joint over-representation/enrichment pathway analysis; and MPEA, a GSEA-based tool handling many-to-many compound-pathway relationships. Record each tool's top significant pathway and P value: aminoacyl-tRNA biosynthesis (P = 5.8e-09) for MetPA, biochemical pathways part I (P = 4.4e-34) for IMPaLA, and protein digestion and absorption (P = 3.15e-20) for MPEA. Compile the top-ranked pathways and P values side by side into a comparison CSV for validation against Table S1. Rationale: these tools operate at group level, contrasting with lilikoi's personalized pathway deregulation score approach.

## Related tools

- **MetPA** (web tool combining pathway enrichment analysis with pathway topological characteristics; used to obtain the top significant pathway (aminoacyl-tRNA biosynthesis) and its P value)
- **IMPaLA** (web tool performing joint over-representation/enrichment pathway analysis; used to obtain the top significant pathway (biochemical pathways part I) and its P value)
- **MPEA** (GSEA-based pathway analysis tool handling many-to-many compound-pathway relationships; used to obtain the top significant pathway (protein digestion and absorption) and its P value)
- **lilikoi** (R package supplying the source dataset (plasma_breast_cancer.csv in extdata) and HMDB-standardized metabolite names submitted to the web tools) — https://github.com/lanagarmire/lilikoi
- **R** (scripting language used to load the metabolite measurements and extract the 227 metabolite names from the lilikoi extdata file) — https://cran.r-project.org/web/packages/lilikoi/index.html

## Examples

```
filename <- system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi"); metaboliteMeasurements <- read.csv(filename, check.names = FALSE, row.names = 1); metaboliteNames <- colnames(metaboliteMeasurements)[-1]
```

## Evaluation signals

- Compiled table_s1_enrichment_comparison.csv exists as a valid CSV file
- MetPA top hit reproduces aminoacyl-tRNA biosynthesis with P = 5.8e-09
- IMPaLA top hit reproduces biochemical pathways part I with P = 4.4e-34
- MPEA top hit reproduces protein digestion and absorption with P = 3.15e-20
- Input metabolite list contains 227 names, of which 182 map to standard HMDB IDs, matching the paper's mapping rate

## Limitations

- MetPA, IMPaLA, and MPEA are group-level methods and cannot produce individual patient-level pathway matrices or pathway features for building classification models
- Approximately 20% of metabolites cannot be mapped to standard names in databases, partly because lilikoi uses Levenshtein distance as the similarity measure for name matching, causing mismatches
- Results depend on the current versions of external web tools and databases (HMDB, SMPDB, MetaboAnalyst), so P values may drift from the published Table S1 values over time

## Evidence

- [intro] MetPA is a web tool that combines pathway enrichment analysis with pathway topological characteristics to help identify the most relevant metabolic pathways.: "MetPA is a web tool that combines pathway enrichment analysis with pathway topological characteristics to help identify the most relevant metabolic pathways."
- [intro] IMPaLA is a web tool that performs joint pathway analysis of transcriptomics or proteomics and metabolomics data through over-representation or enrichment analysis.: "IMPaLA is a web tool that performs joint pathway analysis of transcriptomics or proteomics and metabolomics data through over-representation or enrichment analysis."
- [intro] MPEA is another pathway analysis tool based on GSEA principles, designed specifically to handle many-to-many relationships that may occur between the query compounds and metabolite annotations.: "MPEA is another pathway analysis tool based on GSEA principles, designed specifically to handle many-to-many relationships that may occur between the query compounds and metabolite annotations."
- [results] other pathway-based methods, such as MetPA [11], IMPaLA [12], and MPEA [13], are not capable of generating such individual patient-level pathway matrices: "other pathway-based methods, such as MetPA [11], IMPaLA [12], and MPEA [13], are not capable of generating such individual patient-level pathway matrices"
- [results] In the first plasma sample set, 182 of 227 metabolites are mapped to standard HMDB IDs: "In the first plasma sample set, 182 of 227 metabolites are mapped to standard HMDB IDs"
- [readme] filename <- system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi"): "filename <- system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi")
metaboliteMeasurements <- read.csv(file = filename, check.names = FALSE, row.names = 1)"
