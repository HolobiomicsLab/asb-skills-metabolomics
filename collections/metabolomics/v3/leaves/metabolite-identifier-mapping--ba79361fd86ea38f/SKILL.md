---
name: metabolite-identifier-mapping
description: Use when the input is a metabolomics feature matrix whose columns (or
  rows) are raw metabolite names (e.g. 227 plasma or 162 tissue metabolites) and you
  need standard HMDB IDs plus pathway assignments before computing pathway deregulation
  scores.
when_to_use_negative:
- Input identifiers are already standard HMDB IDs or the analysis already provides
  a pathway/feature table — mapping would be redundant
- You need patient-level pathway matrices from transcriptomics/proteomics rather than
  metabolite name-to-pathway mapping (use Lilikoi's downstream dimension transformation
  module instead)
- You require group-level enrichment P values only (e.g. MetPA, IMPaLA, MPEA outputs)
  without needing individual sample-level pathway transformation
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: lilikoi
  role: Provides lilikoi.metab_to_pathway for exact, synonym, and Levenshtein fuzzy
    matching of metabolite names to HMDB IDs and pathways; lilikoi.update_database
    refreshes HMDB/SMPDB/MetaboAnalyst data
  repo: https://github.com/lanagarmire/lilikoi
- name: R
  role: Runtime environment for loading the CSV via system.file/read.csv and executing
    lilikoi mapping functions
  repo: https://cran.r-project.org/web/packages/lilikoi/index.html
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_007
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy136
    title: 'Lilikoi: an R package for personalized pathway-based classification modeling
      using metabolomics data'
  source_content_hash: sha256:1746b263d443cce63f72574a22c574d1e37b21aa792415d9e15de591fc7ad793
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi
  tools:
  - IMPaLA
  - MPEA
  - MetPA
  - R
  - R/Bioconductor
  - lilikoi
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# metabolite-identifier-mapping

## Summary

Standardizes user-provided metabolite names to standard HMDB IDs and maps them to pathways using Lilikoi's feature mapping module (lilikoi.metab_to_pathway), the first step of its metabolomics classification pipeline.

## When to use

Use when the input is a metabolomics feature matrix whose columns (or rows) are raw metabolite names (e.g. 227 plasma or 162 tissue metabolites) and you need standard HMDB IDs plus pathway assignments before computing pathway deregulation scores.

## When NOT to use

- Input identifiers are already standard HMDB IDs or the analysis already provides a pathway/feature table — mapping would be redundant
- You need patient-level pathway matrices from transcriptomics/proteomics rather than metabolite name-to-pathway mapping (use Lilikoi's downstream dimension transformation module instead)
- You require group-level enrichment P values only (e.g. MetPA, IMPaLA, MPEA outputs) without needing individual sample-level pathway transformation

## Inputs

- Metabolite measurement CSV (e.g. plasma_breast_cancer.csv, 227 metabolites × 207 patients) with metabolite names as column headers
- Character vector of user-provided metabolite names
- Lilikoi's bundled annotation databases (data.hmdb, data.metaboAnalyst, data.smpdb)

## Outputs

- metabolitePathwayTable: one row per input metabolite with matched standard HMDB ID and assigned pathway(s), written e.g. to metabolite_hmdb_mapping.csv
- Mapping rate statistic (fraction of metabolites assigned a standard HMDB ID)

## How to apply

Load the metabolite measurement CSV (e.g. plasma_breast_cancer.csv from lilikoi's extdata) and extract metabolite names via colnames(metaboliteMeasurements)[-1]. Run lilikoi.metab_to_pathway(metaboliteNames, 'name'), which attempts an exact match against Lilikoi's bundled HMDB entries, falls back to a synonym table, and finally applies Levenshtein edit-distance fuzzy matching for remaining unmatched names — Lilikoi depends on data from HMDB, SMPDB, and MetaboAnalyst snapshots shipped with the package. Write the resulting metabolitePathwayTable to CSV, retaining each input metabolite name alongside its matched standard HMDB ID and assigned pathway(s). Compute the mapping rate as the fraction of input metabolites that received a standard HMDB ID (expected ~80%; about 20% may remain unmapped because of Levenshtein-distance name similarity mismatches). Use lilikoi.update_database() if newer HMDB/SMPDB/MetaboAnalyst data are needed, noting this ships >5GB datasets and can take >20 minutes.

## Related tools

- **lilikoi** (Provides lilikoi.metab_to_pathway for exact, synonym, and Levenshtein fuzzy matching of metabolite names to HMDB IDs and pathways; lilikoi.update_database refreshes HMDB/SMPDB/MetaboAnalyst data) — https://github.com/lanagarmire/lilikoi
- **R** (Runtime environment for loading the CSV via system.file/read.csv and executing lilikoi mapping functions) — https://cran.r-project.org/web/packages/lilikoi/index.html

## Examples

```
metabolitePathwayTable <- lilikoi.metab_to_pathway(metaboliteNames, "name")
```

## Evaluation signals

- Mapping table is a well-formed CSV with one row per input metabolite (e.g. 227 rows for the plasma dataset, 162 for the tissue dataset)
- Mapping rate matches reported values: 182/227 (~80%) for plasma and 120/162 for the tissue dataset
- Unmapped remainder (~20%) is consistent with Levenshtein-distance fuzzy matching being the fallback step
- Each successfully mapped metabolite carries both a standard HMDB ID and at least one assigned pathway
- lilikoi.update_database() run reports the expected >5GB data download and completes without corrupting the mapping tables

## Limitations

- About 20% of metabolites cannot be mapped to standard names in the databases
- Levenshtein distance as the string-similarity measure is a possible cause of metabolite name mismatching
- Bundled HMDB/SMPDB/MetaboAnalyst data are snapshots as of the publication date; updating via lilikoi.update_database() involves >5GB datasets and may take over 20 minutes

## Evidence

- [abstract] feature mapping module: "feature mapping module, which standardizes the metabolite names provided by users and maps them to pathways"
- [methods] metab_to_pathway call: "metabolitePathwayTable <- lilikoi.metab_to_pathway(metaboliteNames, "name")"
- [results] mapping rate results: "In the first plasma sample set, 182 of 227 metabolites are mapped to standard HMDB IDs"
- [readme] database dependency: "Lilikoi depends on data from HMDB, SMPDB, and MetaboAnalyst. This library ships with the latest data as of the date of publication."
- [discussion] mismatch limitation: "a small percentage (20%) of the metabolites still cannot be mapped to the standard names in databases"
