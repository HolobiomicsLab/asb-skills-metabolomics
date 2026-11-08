---
name: plant-metabolite-database-count-verification
description: 'Use when reconstructing or auditing the composition of the MetMiner
  PM database: you need the exact number of MS1 metabolites retained from each source
  database (KEGG, KNApSAcK, PlantCyc) after plant-specific filtering.'
when_to_use_negative:
- You need counts of MS2 spectra or MS/MS-level entries in the PM database rather
  than MS1 metabolite counts per source database.
- You need the total integrated PM database size after merging/deduplicating the three
  sources — this skill only tabulates per-source retained counts.
- The metabolite counts come from a different pipeline or database build (e.g. HMDB
  or BioCyc) for which the MetMiner article reports no plant-filtered values.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MetMiner
  role: Pipeline whose integrated plant-specific PM database the counts describe;
    source of the reported per-database retained metabolite numbers
  repo: https://github.com/ShawnWx2019/MetMiner
- name: Plant Metabolic Network
  role: Plant-specific metabolic reference resource underpinning the plant-specific
    filtering of MS1-only databases
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:5ef076c085fe30741833aca62273e387f94488a8a82f257c451087568693f163
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/MetMiner
  tools:
  - MetMiner
  - Plant Metabolic Network
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# plant-metabolite-database-count-verification

## Summary

Verifies and tabulates the per-database metabolite counts that survive plant-specific filtering of MS1-only annotation databases (KEGG, KNApSAcK, PlantCyc) used to build the MetMiner PM database, ensuring reported database sizes are traceable and reproducible.

## When to use

Use when reconstructing or auditing the composition of the MetMiner PM database: you need the exact number of MS1 metabolites retained from each source database (KEGG, KNApSAcK, PlantCyc) after plant-specific filtering.

## When NOT to use

- You need counts of MS2 spectra or MS/MS-level entries in the PM database rather than MS1 metabolite counts per source database.
- You need the total integrated PM database size after merging/deduplicating the three sources — this skill only tabulates per-source retained counts.
- The metabolite counts come from a different pipeline or database build (e.g. HMDB or BioCyc) for which the MetMiner article reports no plant-filtered values.

## Inputs

- Reported MS1 metabolite counts per source database from the MetMiner article (Results section): KEGG 3,044; KNApSAcK 1,627; PlantCyc 2,800
- Plant-specific filtered versions of the MS1-only annotation databases KEGG, KNApSAcK, and PlantCyc

## Outputs

- plant_database_ms1_counts.csv — a three-row tidy table with columns (source database, retained MS1 metabolite count)

## How to apply

Load the MS1 metabolite counts reported in the Results section of the MetMiner article for the three source databases (KEGG: 3,044; KNApSAcK: 1,627; PlantCyc: 2,800), which represent entries retained after plant-specific filtering removed non-plant metabolites. Cross-check each count against the paper's reported values for metabolites retained after restricting the annotation databases to plant-specific entries. Assemble the verified counts into a three-row table (source database, retained MS1 metabolite count) using R with the tidyverse readr package, writing the result as plant_database_ms1_counts.csv. Validation requires that the output table contain exactly three rows whose count fields match the reported values 3044, 1627, and 2800.

## Related tools

- **MetMiner** (Pipeline whose integrated plant-specific PM database the counts describe; source of the reported per-database retained metabolite numbers) — https://github.com/ShawnWx2019/MetMiner
- **Plant Metabolic Network** (Plant-specific metabolic reference resource underpinning the plant-specific filtering of MS1-only databases)

## Examples

```
library(readr); plant_database_ms1_counts <- data.frame(source_database = c("KEGG","KNApSAcK","PlantCyc"), retained_ms1_metabolites = c(3044L, 1627L, 2800L)); write_csv(plant_database_ms1_counts, "plant_database_ms1_counts.csv")
```

## Evaluation signals

- Output CSV plant_database_ms1_counts.csv contains exactly three rows.
- Count fields match the reported values exactly: 3044 (KEGG), 1627 (KNApSAcK), 2800 (PlantCyc).
- Each row's count can be traced back to a stated value in the Results section of the MetMiner article (cross-checked against the plant-specific filtering description).
- Table schema is (source database, retained MS1 metabolite count) with no extra columns or missing rows.

## Limitations

- The counts are article-reported values for the version of the PM database described in the 2024 MetMiner paper; upstream KEGG/KNApSAcK/PlantCyc releases change over time, so re-running the filtering pipeline today may yield different numbers.
- The task verifies reported counts only — it does not re-derive the plant-specific filtering itself or explain why specific metabolites were retained or removed.
- Counts cover MS1-only databases only; the PM database also integrates MS2 spectra, which are out of scope here.

## Evidence

- [abstract] we constructed and integrated a plant-specific mass spectrometry database into the MetMiner pipeline to optimize metabolite annotation: "we constructed and integrated a plant-specific mass spectrometry database into the MetMiner pipeline to optimize metabolite annotation"
- [discussion] we also integrated a plant-specific metabolite database in MetMiner, to enhance the accuracy and speed for metabolite annotation in plant metabolomics.: [evidence span withheld — anchor status: not_found]
- [intro] (ii) Plant‐specific databases containing MS1 and MS2 spectra are integrated into MetMiner to enhance metabolite annotation accuracy.: "(ii) Plant‐specific databases containing MS1 and MS2 spectra are integrated into MetMiner to enhance metabolite annotation accuracy."
- [readme] MetMiner, https://github.com/ShawnWx2019/MetMiner), a user-friendly, full-functionality pipeline specifically designed for plant metabolomics data analysis: "we present MetMiner (https://github.com/ShawnWx2019/MetMiner), a user-friendly, full-functionality pipeline"
