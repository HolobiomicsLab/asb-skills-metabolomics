---
name: mapping-rate-qc-comparison
description: Use when running Lilikoi's lilikoi.metab_to_pathway on a metabolomics
  dataset and you need to verify the metabolite-name-to-HMDB mapping rate against
  a reported benchmark (e.g., 182/227 plasma, ~80%) before downstream PDS computation.
when_to_use_negative:
- Input metabolites are already annotated with standard HMDB/PubChem/METLIN IDs, so
  no name standardization or mapping-rate QC is needed
- The goal is pathway-level feature selection or classification (lilikoi.select_pathways,
  lilikoi.machine_learning) rather than verifying the ID-mapping step
- The dataset is shrunk for fast testing (e.g., only the first ~20 metabolite columns
  in the README example), which cannot reproduce the full 227-metabolite benchmark
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: lilikoi
  role: Provides lilikoi.metab_to_pathway for name-based standardization of metabolite
    names to HMDB IDs and pathway assignment, plus the plasma_breast_cancer.csv extdata
    used as the benchmark input
  repo: https://github.com/lanagarmire/lilikoi
- name: R
  role: Scripting environment in which the mapping workflow (read.csv, lilikoi.metab_to_pathway,
    counting mapped IDs) is executed
  repo: https://cran.r-project.org/web/packages/lilikoi/index.html
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giy136
    title: 'Lilikoi: an R package for personalized pathway-based classification modeling
      using metabolomics data'
  source_content_hash: sha256:1789385b977f90ffac12c442cf57d61238d25bcaba874cbb4617f6df19c9e0d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi
  tools:
  - R
  - lilikoi
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mapping-rate-qc-comparison

## Summary

A quality-control skill that reproduces and validates the metabolite-to-standard-ID mapping rate of Lilikoi's feature mapping module, checking how many input metabolite names are standardized to HMDB IDs (e.g., 182/227 in the plasma breast cancer dataset).

## When to use

Use when running Lilikoi's lilikoi.metab_to_pathway on a metabolomics dataset and you need to verify the metabolite-name-to-HMDB mapping rate against a reported benchmark (e.g., 182/227 plasma, ~80%) before downstream PDS computation.

## When NOT to use

- Input metabolites are already annotated with standard HMDB/PubChem/METLIN IDs, so no name standardization or mapping-rate QC is needed
- The goal is pathway-level feature selection or classification (lilikoi.select_pathways, lilikoi.machine_learning) rather than verifying the ID-mapping step
- The dataset is shrunk for fast testing (e.g., only the first ~20 metabolite columns in the README example), which cannot reproduce the full 227-metabolite benchmark

## Inputs

- Metabolite measurement CSV (e.g., plasma_breast_cancer.csv from lilikoi extdata, metabolite names as column headers)
- Lilikoi internal ID-mapping databases (HMDB, SMPDB, MetaboAnalyst snapshots shipped with the package)
- Reported benchmark mapping rate to compare against (182/227 plasma; 120/162 tissue)

## Outputs

- metabolite_pathway_mapping.csv with one row per input metabolite, its matched standard HMDB ID, and assigned pathway(s)
- Mapped-metabolite count and mapping-rate proportion (expected 182/227, ~80%)
- List/description of unmapped metabolites (~20% mismatch rate)

## How to apply

Load the metabolite measurement CSV (e.g., plasma_breast_cancer.csv from the lilikoi package extdata with read.csv using row.names=1 and check.names=FALSE) and extract the metabolite column names — 227 for the plasma dataset. Run lilikoi.metab_to_pathway(metaboliteNames, "name") to standardize each name against Lilikoi's internal HMDB/SMPDB/MetaboAnalyst snapshots via name-based (Levenshtein-distance) matching. Write the resulting metabolitePathwayTable to a mapping CSV, then count metabolites receiving a standard HMDB ID and compute the mapping rate. Compare the achieved count with the reported benchmark (182/227 for plasma; 120/162 for tissue) and treat the ~20% unmapped remainder as expected, attributed to Levenshtein-distance name similarity mismatching rather than a pipeline failure.

## Related tools

- **lilikoi** (Provides lilikoi.metab_to_pathway for name-based standardization of metabolite names to HMDB IDs and pathway assignment, plus the plasma_breast_cancer.csv extdata used as the benchmark input) — https://github.com/lanagarmire/lilikoi
- **R** (Scripting environment in which the mapping workflow (read.csv, lilikoi.metab_to_pathway, counting mapped IDs) is executed) — https://cran.r-project.org/web/packages/lilikoi/index.html

## Examples

```
metaboliteNames <- colnames(read.csv(system.file("extdata","plasma_breast_cancer.csv",package="lilikoi"),check.names=FALSE,row.names=1)); metabolitePathwayTable <- lilikoi.metab_to_pathway(metaboliteNames, "name")
```

## Evaluation signals

- The mapping table contains exactly 227 rows, one per input metabolite from plasma_breast_cancer.csv
- The achieved mapped count equals 182 metabolites with standard HMDB IDs (rate ≈ 80%)
- The unmapped remainder (~20%, i.e., 45 metabolites) is consistent with the reported mismatch rate attributed to Levenshtein-distance name similarity
- For the tissue dataset, the analogous check yields 120/162 metabolites mapped to standard HMDB IDs
- Mapping is reproducible from the shipped HMDB/SMPDB/MetaboAnalyst snapshots without needing lilikoi.update_database (which pulls >5GB of data)

## Limitations

- About 20% of metabolites cannot be mapped to standard names in the databases
- Levenshtein distance used as the name-similarity measure is a possible cause of metabolite name mismatching
- The package ships with database snapshots as of publication date; updating via lilikoi.update_database() downloads >5GB and may take over 20 minutes
- The README example shrinks the dataset (metaboliteMeasurements[, 1:20]) and subsets the SMPDB database (data.smpdb[1:25,]) for speed, which must be reverted to reproduce the full 227-metabolite mapping rate

## Evidence

- [results] plasma mapping benchmark: "In the first plasma sample set, 182 of 227 metabolites are mapped to standard HMDB IDs"
- [results] tissue mapping benchmark: "in the second breast cancer tissue sample set, 120 of 162 metabolites are mapped to standard HMDB IDs"
- [methods] mapping function call: "metabolitePathwayTable <- lilikoi.metab_to_pathway(metaboliteNames, "name")"
- [readme] data loading in README: "metaboliteMeasurements <- read.csv(file = filename, check.names = FALSE, row.names = 1)"
- [discussion] mismatch cause: "a small percentage (20%) of the metabolites still cannot be mapped to the standard names in databases"
