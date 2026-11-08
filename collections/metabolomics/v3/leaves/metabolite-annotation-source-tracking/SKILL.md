---
name: metabolite-annotation-source-tracking
description: Use when building or auditing a plant-specific MS1 annotation database
  from multiple source libraries (KEGG, KNApSAcK, PlantCyc) and you need per-source
  retained counts after filtering out non-plant metabolites.
when_to_use_negative:
- The goal is MS2 spectral matching or annotation scoring rather than tracking per-source
  MS1 database composition
- Input is a single metabolite feature table with annotations already assigned and
  provenance already recorded
- The organism is non-plant; the PM database is explicitly plant-specific and would
  wrongly remove non-plant metabolites
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetMiner
  role: Pipeline that integrates the plant-specific MS1/MS2 database (PM database)
    and provides annotation and annotation filtering interfaces for per-source metabolite
    annotation
  repo: https://github.com/ShawnWx2019/MetMiner
- name: MDAtoolkits
  role: Provides filter_annotations_massdataset and related annotation-filtering functions
    used to apply and track annotation filtering within MetMiner
  repo: https://github.com/ShawnWx2019/MDAtoolkits
- name: tidyverse
  role: readr package used to write the verified per-source counts as plant_database_ms1_counts.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:b5262b45d0059debe499c6eca1d0daf05fa580773512edc8716926104efd929c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/MetMiner
  tools:
  - MDAtoolkits
  - MetMiner
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# metabolite-annotation-source-tracking

## Summary

Tracks how many metabolite entries from each MS1-only annotation database (KEGG, KNApSAcK, PlantCyc) survive plant-specific filtering into MetMiner's integrated PM database, so downstream annotation can be attributed per source.

## When to use

Use when building or auditing a plant-specific MS1 annotation database from multiple source libraries (KEGG, KNApSAcK, PlantCyc) and you need per-source retained counts after filtering out non-plant metabolites.

## When NOT to use

- The goal is MS2 spectral matching or annotation scoring rather than tracking per-source MS1 database composition
- Input is a single metabolite feature table with annotations already assigned and provenance already recorded
- The organism is non-plant; the PM database is explicitly plant-specific and would wrongly remove non-plant metabolites

## Inputs

- MS1-only annotation databases (KEGG, KNApSAcK, PlantCyc) metabolite entry tables
- Plant-specific filtering criteria used by MetMiner to restrict annotation databases to plant-specific entries
- Reported per-source retained counts from the MetMiner article (3044, 1627, 2800)

## Outputs

- plant_database_ms1_counts.csv — three-row table (source database, retained MS1 metabolite count)
- Per-source provenance for metabolite annotations in the PM database

## How to apply

Load the reported MS1 metabolite counts for each source database (KEGG: 3,044; KNApSAcK: 1,627; PlantCyc: 2,800) as stated in the Results of the MetMiner article, which are the entries retained after plant-specific filtering of the MS1-only databases into the PM database. Cross-check each per-source count against the paper's reported retained values before reuse. Assemble the verified counts into a three-row table (source database, retained MS1 metabolite count) using R with the tidyverse readr package and write it as plant_database_ms1_counts.csv. When applying the analogous filter at runtime, use MetMiner/MDAtoolkits annotation filtering functions (e.g. filter_annotations_massdataset) so each annotated feature retains its source-database provenance. The rationale is that integrating a plant-specific metabolite database enhances annotation accuracy and speed by removing non-plant entries before matching.

## Related tools

- **MetMiner** (Pipeline that integrates the plant-specific MS1/MS2 database (PM database) and provides annotation and annotation filtering interfaces for per-source metabolite annotation) — https://github.com/ShawnWx2019/MetMiner
- **MDAtoolkits** (Provides filter_annotations_massdataset and related annotation-filtering functions used to apply and track annotation filtering within MetMiner) — https://github.com/ShawnWx2019/MDAtoolkits
- **tidyverse** (readr package used to write the verified per-source counts as plant_database_ms1_counts.csv)

## Examples

```
filter_annotations_massdataset(object, annotate_tbl, method)
```

## Evaluation signals

- Output table contains exactly three rows, one per source database (KEGG, KNApSAcK, PlantCyc)
- Count fields match the reported retained values exactly: KEGG = 3044, KNApSAcK = 1627, PlantCyc = 2800
- Each retained count is smaller than (or consistent with) the corresponding pre-filter MS1 database size, reflecting removal of non-plant metabolites
- plant_database_ms1_counts.csv is written with the expected columns (source database, retained MS1 metabolite count) and parses cleanly with readr

## Limitations

- The PM database is plant-specific, so non-plant metabolomics studies cannot reuse these filtered counts
- In-source fragmentation products can still be misannotated as genuine metabolites; MetMiner's authors state there is no satisfactory solution to detect and remove such redundant annotations yet
- No version number is stated for MetMiner, tidyMass, or MDAtoolkits, so per-source counts may shift across tool versions

## Evidence

- [intro] Plant‐specific databases containing MS1 and MS2 spectra are integrated into MetMiner to enhance metabolite annotation accuracy.: "Plant‐specific databases containing MS1 and MS2 spectra are integrated into MetMiner to enhance metabolite annotation accuracy."
- [discussion] we also integrated a plant-specific metabolite database in MetMiner, to enhance the accuracy and speed for metabolite annotation in plant metabolomics.: [evidence span withheld — anchor status: not_found]
- [methods] - `filter_annotations_massdataset(object, annotate_tbl, method)`: "- `filter_annotations_massdataset(object, annotate_tbl, method)`"
- [discussion] Currently, there is no satisfactory solution to detect and remove such redundant annotations: "Currently, there is no satisfactory solution to detect and remove such redundant annotations, and addressing this issue would be a major focus for the next version of MetMiner."
- [readme] ##> start metMiner shinyapp
run_metMiner(maxRequestSize = 300): "##> start metMiner shinyapp
run_metMiner(maxRequestSize = 300)"
