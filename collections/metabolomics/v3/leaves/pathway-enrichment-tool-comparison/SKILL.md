---
name: pathway-enrichment-tool-comparison
description: Use when you have a standardized metabolite list (e.g. HMDB-mapped names)
  and need to benchmark which pathways different group-level enrichment methods rank
  as most significant.
when_to_use_negative:
- You need patient-level (personalized) pathway features for classification — the
  paper states MetPA, IMPaLA, and MPEA are 'not capable of generating such individual
  patient-level pathway matrices' (use lilikoi.get_pd_scores instead)
- Your metabolites are not yet standardized to IDs accepted by the tools (e.g. ~20%
  of metabolites fail HMDB mapping via Levenshtein-distance matching) — run the feature
  mapping module first
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: lilikoi
  role: Source of the benchmark metabolite list and the plasma_breast_cancer.csv extdata
    dataset used to query all three enrichment tools
  repo: https://github.com/lanagarmire/lilikoi
- name: MetPA
  role: Web tool combining pathway enrichment analysis with pathway topological characteristics;
    supplies the top significant pathway and P value column
- name: IMPaLA
  role: Web tool performing joint over-representation/enrichment pathway analysis;
    supplies the top significant pathway and P value column
- name: MPEA
  role: GSEA-based pathway tool handling many-to-many compound-pathway relationships;
    supplies the top significant pathway and P value column
- name: R
  role: Used to load the lilikoi package dataset and extract the metabolite names
    submitted to the three tools
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy136
    title: 'Lilikoi: an R package for personalized pathway-based classification modeling
      using metabolomics data'
  source_content_hash: sha256:76e29e82d7fdc9032534c22626ba12fed970b76025715980328ca03b0bc5edec
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
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# pathway-enrichment-tool-comparison

## Summary

A benchmarking skill that submits a curated metabolite list to multiple group-level pathway enrichment tools (MetPA, IMPaLA, MPEA) and compiles each tool's top-ranked pathway and P value into a side-by-side comparison table, as done in the Lilikoi paper's Supplementary Table S1.

## When to use

Use when you have a standardized metabolite list (e.g. HMDB-mapped names) and need to benchmark which pathways different group-level enrichment methods rank as most significant.

## When NOT to use

- You need patient-level (personalized) pathway features for classification — the paper states MetPA, IMPaLA, and MPEA are 'not capable of generating such individual patient-level pathway matrices' (use lilikoi.get_pd_scores instead)
- Your metabolites are not yet standardized to IDs accepted by the tools (e.g. ~20% of metabolites fail HMDB mapping via Levenshtein-distance matching) — run the feature mapping module first

## Inputs

- Metabolite name list extracted from a metabolomics dataset (e.g. colnames of plasma_breast_cancer.csv, 227 metabolites)
- HMDB-standardized metabolite identifiers (182 of 227 metabolites mapped for dataset 1)
- MetPA web tool session
- IMPaLA web tool session
- MPEA web tool session

## Outputs

- Comparison table of top-ranked pathways per tool with enrichment P values (table_s1_enrichment_comparison.csv)
- Per-tool top significant pathway and P value (MetPA: aminoacyl-tRNA biosynthesis P = 5.8e-09; IMPaLA: biochemical pathways part I P = 4.4e-34; MPEA: protein digestion and absorption P = 3.15e-20)

## How to apply

Extract the metabolite names from the input dataset — in the Lilikoi paper, the 227 metabolites from the plasma_breast_cancer.csv file in the lilikoi package extdata, of which 182 map to standard HMDB IDs. Submit the same metabolite list separately to three tools: MetPA (web tool combining pathway enrichment analysis with pathway topological characteristics), IMPaLA (web tool performing joint over-representation/enrichment pathway analysis of metabolomics data), and MPEA (a GSEA-based tool designed for many-to-many compound-pathway relationships). Record each tool's most significant pathway and its enrichment P value: for the plasma breast cancer dataset these are aminoacyl-tRNA biosynthesis (P = 5.8e-09) for MetPA, biochemical pathways part I (P = 4.4e-34) for IMPaLA, and protein digestion and absorption (P = 3.15e-20) for MPEA. Compile the top-ranked pathways and P values side by side in a CSV table and check that the recorded entries reproduce the reference values before treating the comparison as valid.

## Related tools

- **lilikoi** (Source of the benchmark metabolite list and the plasma_breast_cancer.csv extdata dataset used to query all three enrichment tools) — https://github.com/lanagarmire/lilikoi
- **MetPA** (Web tool combining pathway enrichment analysis with pathway topological characteristics; supplies the top significant pathway and P value column)
- **IMPaLA** (Web tool performing joint over-representation/enrichment pathway analysis; supplies the top significant pathway and P value column)
- **MPEA** (GSEA-based pathway tool handling many-to-many compound-pathway relationships; supplies the top significant pathway and P value column)
- **R** (Used to load the lilikoi package dataset and extract the metabolite names submitted to the three tools)

## Examples

```
filename <- system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi"); metaboliteMeasurements <- read.csv(filename, check.names = FALSE, row.names = 1); metaboliteNames <- colnames(metaboliteMeasurements)[-1]
```

## Evaluation signals

- The compiled CSV table exists and contains one row per tool (MetPA, IMPaLA, MPEA) with a pathway name and a P value
- MetPA top hit is aminoacyl-tRNA biosynthesis with P = 5.8e-09
- IMPaLA top hit is biochemical pathways part I with P = 4.4e-34
- MPEA top hit is protein digestion and absorption with P = 3.15e-20
- The metabolite list submitted to all three tools is identical (the same HMDB-mapped metabolite names), confirming a fair benchmark

## Limitations

- MetPA, IMPaLA, and MPEA operate at group level and cannot generate individual patient-level pathway matrices, so their results are not usable for personalized classification features
- Only ~182 of 227 plasma metabolites map to standard HMDB IDs; unmapped metabolites (~20%) are excluded from enrichment input
- Metabolite-to-standard-name mapping relies on Levenshtein distance similarity, which can cause name mismatches and affect the submitted list

## Evidence

- [intro] MetPA is a web tool that combines pathway enrichment analysis with pathway topological characteristics to help identify the most relevant metabolic pathways.: "MetPA is a web tool that combines pathway enrichment analysis with pathway topological characteristics to help identify the most relevant metabolic pathways."
- [intro] IMPaLA is a web tool that performs joint pathway analysis of transcriptomics or proteomics and metabolomics data through over-representation or enrichment analysis.: "IMPaLA is a web tool that performs joint pathway analysis of transcriptomics or proteomics and metabolomics data through over-representation or enrichment analysis."
- [intro] MPEA is another pathway analysis tool based on GSEA principles, designed specifically to handle many-to-many relationships that may occur between the query compounds and metabolite annotations.: "MPEA is another pathway analysis tool based on GSEA principles, designed specifically to handle many-to-many relationships that may occur between the query compounds and metabolite annotations."
- [results] other pathway-based methods, such as MetPA [11], IMPaLA [12], and MPEA [13], are not capable of generating such individual patient-level pathway matrices: "other pathway-based methods, such as MetPA [11], IMPaLA [12], and MPEA [13], are not capable of generating such individual patient-level pathway matrices"
- [methods] filename <- system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi"): "filename <- system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi")"
- [readme] Lilikoi depends on data from HMDB, SMPDB, and MetaboAnalyst.: "Lilikoi depends on data from HMDB, SMPDB, and MetaboAnalyst."
