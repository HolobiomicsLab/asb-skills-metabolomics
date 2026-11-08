---
name: tabular-result-reproduction
description: Use when a research question asks to reproduce specific reported counts
  from an article — e.g. how many metabolites from each MS1-only database (KEGG, KNApSAcK,
  PlantCyc) were retained in the PM database after plant-specific filtering — into
  a small verified table.
when_to_use_negative:
- Input is raw LC-MS metabolomics data requiring peak picking, data cleaning, or metabolite
  annotation — MetMiner's upstream tidyMass-driven workflow handles that, not this
  skill
- The goal is to independently recompute the retained-metabolite counts by re-running
  plant-specific filtering over KEGG/KNApSAcK/PlantCyc, rather than extracting and
  verifying the article's reported numbers
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetMiner
  role: Pipeline whose integrated plant-specific PM database is the subject of the
    reproduced counts; its article is the source of the reported values
  repo: https://github.com/ShawnWx2019/MetMiner
- name: tidyverse (readr)
  role: R packages used to assemble the three-row count table and write it as plant_database_ms1_counts.csv
- name: Plant Metabolic Network
  role: Plant-specific reference context for the metabolite database filtering being
    reproduced
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:84fbea03d0b7235f2bd0b927d3a6df655c4acb8d23b4539a07460319e9e69a34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/MetMiner
  tools:
  - MetMiner
  - Plant Metabolic Network
  - tidyverse (readr)
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# tabular-result-reproduction

## Summary

Reproduces published summary tables from a scientific article into a small, validated machine-readable table. Here, it reconstructs the per-source MS1 metabolite counts of MetMiner's integrated plant-specific PM database (KEGG, KNApSAcK, PlantCyc) after plant-specific filtering, as reported in the paper's Results.

## When to use

Use when a research question asks to reproduce specific reported counts from an article — e.g. how many metabolites from each MS1-only database (KEGG, KNApSAcK, PlantCyc) were retained in the PM database after plant-specific filtering — into a small verified table.

## When NOT to use

- Input is raw LC-MS metabolomics data requiring peak picking, data cleaning, or metabolite annotation — MetMiner's upstream tidyMass-driven workflow handles that, not this skill
- The goal is to independently recompute the retained-metabolite counts by re-running plant-specific filtering over KEGG/KNApSAcK/PlantCyc, rather than extracting and verifying the article's reported numbers

## Inputs

- Reported per-source MS1 metabolite counts from the MetMiner article (DOI 10.1111/jipb.13774): KEGG = 3044, KNApSAcK = 1627, PlantCyc = 2800
- Reference values for metabolites retained after plant-specific filtering of the MS1-only databases, as stated in the Results section

## Outputs

- plant_database_ms1_counts.csv — a three-row CSV table (source database, retained MS1 metabolite count)

## How to apply

Load the reported per-source MS1 metabolite counts stated in the Results section of the MetMiner article (KEGG: 3,044; KNApSAcK: 1,627; PlantCyc: 2,800), which correspond to metabolites retained after restricting each MS1-only annotation database to plant-specific entries. Cross-check each count against the paper's reported post-filtering values to guard against transcription error. Assemble a three-row table with columns for the source database and the retained MS1 metabolite count using R with the tidyverse readr package, and write it as plant_database_ms1_counts.csv. Validate the output: it must contain exactly three rows whose count fields match 3044, 1627, and 2800. Note this reproduces reported values; it does not re-run MetMiner's plant-specific filtering itself.

## Related tools

- **MetMiner** (Pipeline whose integrated plant-specific PM database is the subject of the reproduced counts; its article is the source of the reported values) — https://github.com/ShawnWx2019/MetMiner
- **tidyverse (readr)** (R packages used to assemble the three-row count table and write it as plant_database_ms1_counts.csv)
- **Plant Metabolic Network** (Plant-specific reference context for the metabolite database filtering being reproduced)

## Examples

```
library(readr); write_csv(data.frame(database=c('KEGG','KNApSAcK','PlantCyc'), ms1_metabolites=c(3044L,1627L,2800L)), 'plant_database_ms1_counts.csv')
```

## Evaluation signals

- Output CSV contains exactly three rows
- Count fields match the reported values 3044 (KEGG), 1627 (KNApSAcK), and 2800 (PlantCyc)
- Each count has been cross-checked against the paper's stated value for metabolites retained after plant-specific filtering of the MS1-only databases
- Table schema includes a source-database column and a retained MS1 metabolite count column, written via tidyverse readr

## Limitations

- Reproduces reported values only; it does not independently rerun the plant-specific filtering of the MS1-only databases, so it cannot detect errors in the original publication
- No version number is stated for MetMiner or its PM database build, so the counts cannot be pinned to a specific database snapshot
- Counts are specific to the article's PM database construction (KEGG/KNApSAcK/PlantCyc plant-specific subsets) and are not transferable to other annotation database versions or non-plant metabolomics

## Evidence

- [abstract] we constructed and integrated a plant-specific mass spectrometry database into the MetMiner pipeline to optimize metabolite annotation: "we constructed and integrated a plant-specific mass spectrometry database into the MetMiner pipeline to optimize metabolite annotation"
- [discussion] we also integrated a plant-specific metabolite database in MetMiner, to enhance the accuracy and speed for metabolite annotation in plant metabolomics.: [evidence span withheld — anchor status: not_found]
- [readme] if (!require('tidyverse')) install.packages('tidyverse');: "if (!require('tidyverse')) install.packages('tidyverse');"
