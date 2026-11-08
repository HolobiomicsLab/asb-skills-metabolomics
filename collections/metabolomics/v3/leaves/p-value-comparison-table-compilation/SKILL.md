---
name: p-value-comparison-table-compilation
description: Use when benchmarking group-level metabolite pathway enrichment tools
  on a shared metabolite list and you need a side-by-side comparison of each tool's
  top significant pathway and its enrichment P value.
when_to_use_negative:
- You need patient-level pathway features for classification modeling — MetPA, IMPaLA,
  and MPEA operate at group level and are not capable of generating individual patient-level
  pathway matrices (use Lilikoi's dimension transformation module instead)
- Your input is already a PDS score matrix or pathway-level feature table rather than
  a raw metabolite list requiring enrichment testing
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: MetPA
  role: Web tool that combines pathway enrichment analysis with pathway topological
    characteristics; supplies the top significant pathway (aminoacyl-tRNA biosynthesis)
    and its P value for the comparison table
- name: IMPaLA
  role: Web tool performing joint over-representation/enrichment pathway analysis;
    supplies its top significant pathway (biochemical pathways part I) and P value
- name: MPEA
  role: GSEA-based pathway analysis tool handling many-to-many compound-pathway relationships;
    supplies its top significant pathway (protein digestion and absorption) and P
    value
- name: lilikoi
  role: R package whose extdata file plasma_breast_cancer.csv provides the 227-metabolite
    input list submitted to the three enrichment tools
  repo: https://github.com/lanagarmire/lilikoi
- name: R
  role: Scripting environment used to load the lilikoi extdata metabolite measurements
    and compile the results into CSV
  repo: https://cran.r-project.org/web/packages/lilikoi/index.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy136
    title: 'Lilikoi: an R package for personalized pathway-based classification modeling
      using metabolomics data'
  source_content_hash: sha256:1c0ec6c08cc71db339537d1da0da178fc8a41bbe3fdcc3a40b189283c91fa9d2
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
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# p-value-comparison-table-compilation

## Summary

Compiles the top-ranked pathway enrichment hits and P values from group-level enrichment tools (MetPA, IMPaLA, MPEA) into a side-by-side CSV table that reproduces Supplementary Table S1 of the Lilikoi paper on the plasma breast cancer metabolomics dataset.

## When to use

Use when benchmarking group-level metabolite pathway enrichment tools on a shared metabolite list and you need a side-by-side comparison of each tool's top significant pathway and its enrichment P value.

## When NOT to use

- You need patient-level pathway features for classification modeling — MetPA, IMPaLA, and MPEA operate at group level and are not capable of generating individual patient-level pathway matrices (use Lilikoi's dimension transformation module instead)
- Your input is already a PDS score matrix or pathway-level feature table rather than a raw metabolite list requiring enrichment testing

## Inputs

- plasma_breast_cancer.csv metabolite measurement matrix from the lilikoi package extdata (227 metabolites; 182 mapped to standard HMDB IDs)
- Metabolite name list suitable for submission to MetPA, IMPaLA, and MPEA web tools

## Outputs

- table_s1_enrichment_comparison.csv: side-by-side table of top-ranked pathway and enrichment P value for MetPA (aminoacyl-tRNA biosynthesis, P = 5.8e-09), IMPaLA (biochemical pathways part I, P = 4.4e-34), and MPEA (protein digestion and absorption, P = 3.15e-20)

## How to apply

Load the plasma breast-cancer metabolite measurements from the lilikoi package extdata file plasma_breast_cancer.csv and extract the 227 metabolite names, 182 of which map to standard HMDB IDs. Submit the same metabolite list to each of three web tools in turn: MetPA (which combines pathway enrichment analysis with pathway topological characteristics), IMPaLA (which performs joint over-representation/enrichment pathway analysis), and MPEA (a GSEA-based tool handling many-to-many compound-pathway relationships), recording each tool's top significant pathway and P value. Compile the three top-ranked pathway/p-value pairs side by side and write them to table_s1_enrichment_comparison.csv. Validate by checking the CSV exists and that its recorded entries reproduce the Table S1 values: aminoacyl-tRNA biosynthesis (P = 5.8e-09) for MetPA, biochemical pathways part I (P = 4.4e-34) for IMPaLA, and protein digestion and absorption (P = 3.15e-20) for MPEA.

## Related tools

- **MetPA** (Web tool that combines pathway enrichment analysis with pathway topological characteristics; supplies the top significant pathway (aminoacyl-tRNA biosynthesis) and its P value for the comparison table)
- **IMPaLA** (Web tool performing joint over-representation/enrichment pathway analysis; supplies its top significant pathway (biochemical pathways part I) and P value)
- **MPEA** (GSEA-based pathway analysis tool handling many-to-many compound-pathway relationships; supplies its top significant pathway (protein digestion and absorption) and P value)
- **lilikoi** (R package whose extdata file plasma_breast_cancer.csv provides the 227-metabolite input list submitted to the three enrichment tools) — https://github.com/lanagarmire/lilikoi
- **R** (Scripting environment used to load the lilikoi extdata metabolite measurements and compile the results into CSV) — https://cran.r-project.org/web/packages/lilikoi/index.html

## Examples

```
filename <- system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi"); metaboliteMeasurements <- read.csv(file = filename, check.names = FALSE, row.names = 1); write.csv(table_s1, "table_s1_enrichment_comparison.csv")
```

## Evaluation signals

- table_s1_enrichment_comparison.csv exists as a valid CSV file
- MetPA row records aminoacyl-tRNA biosynthesis with P = 5.8e-09
- IMPaLA row records biochemical pathways part I with P = 4.4e-34
- MPEA row records protein digestion and absorption with P = 3.15e-20
- The compiled top-hit entries reproduce the Supplementary Table S1 values exactly

## Limitations

- These group-level enrichment methods cannot generate individual patient-level pathway matrices, so the table benchmarks only group-level significance, not personalized pathway deregulation
- MetPA, IMPaLA, and MPEA are external web tools, so results depend on current tool versions and database backends at submission time

## Evidence

- [intro] MetPA combines enrichment and topology on the plasma metabolite list: "MetPA is a web tool that combines pathway enrichment analysis with pathway topological characteristics to help identify the most relevant metabolic pathways."
- [intro] IMPaLA joint pathway analysis: "IMPaLA is a web tool that performs joint pathway analysis of transcriptomics or proteomics and metabolomics data through over-representation or enrichment analysis."
- [intro] MPEA GSEA-based many-to-many handling: "MPEA is another pathway analysis tool based on GSEA principles, designed specifically to handle many-to-many relationships that may occur between the query compounds and metabolite annotations."
- [results] 182 of 227 plasma metabolites map to HMDB IDs: "In the first plasma sample set, 182 of 227 metabolites are mapped to standard HMDB IDs"
- [methods] plasma_breast_cancer.csv loading from lilikoi extdata: "filename <- system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi")"
- [results] group-level tools cannot produce patient-level pathway matrices: "other pathway-based methods, such as MetPA [11], IMPaLA [12], and MPEA [13], are not capable of generating such individual patient-level pathway matrices"
