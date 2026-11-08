---
name: metabolite-name-standardization
description: Use when a metabolomics dataset contains free-text metabolite names that
  must be converted to standard HMDB/SMPDB IDs before pathway mapping or pathway-based
  classification (e.g., Lilikoi PDS modeling).
when_to_use_negative:
- The metabolite features are already standard HMDB/PubChem/METLIN IDs or already
  mapped to pathways (the feature mapping step is redundant)
- The goal is group-level pathway enrichment rather than per-metabolite ID standardization
  feeding sample-level pathway modeling (MetPA/IMPaLA/MPEA-style analyses)
- The dataset has been shrunk for fast testing (e.g., metaboliteMeasurements[, 1:20]),
  so mapping-rate results would not be comparable to the reported 182/227
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: lilikoi
  role: Provides the feature mapping module lilikoi.metab_to_pathway and the internal
    HMDB/SMPDB/MetaboAnalyst database snapshots used for name-based standardization;
    lilikoi.update_database() refreshes those databases
  repo: https://github.com/lanagarmire/lilikoi
- name: R
  role: Runtime environment for loading the CSV, extracting metabolite names, and
    running lilikoi functions
  repo: https://cran.r-project.org/web/packages/lilikoi/index.html
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giy136
    title: 'Lilikoi: an R package for personalized pathway-based classification modeling
      using metabolomics data'
  source_content_hash: sha256:ac6396f0360e3730e2f217ea1c797dc1bfc890b6adda622aba53282008d35ffa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi
  tools:
  - R
  - lilikoi
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# metabolite-name-standardization

## Summary

Standardizes raw, non-standard metabolite names from a user-provided metabolomics measurement file (e.g., the Lilikoi plasma_breast_cancer.csv dataset of 227 metabolites) to standard database IDs such as HMDB, using Lilikoi's feature mapping module (lilikoi.metab_to_pathway) with name-based matching against bundled HMDB/SMPDB/MetaboAnalyst snapshots.

## When to use

Use when a metabolomics dataset contains free-text metabolite names that must be converted to standard HMDB/SMPDB IDs before pathway mapping or pathway-based classification (e.g., Lilikoi PDS modeling).

## When NOT to use

- The metabolite features are already standard HMDB/PubChem/METLIN IDs or already mapped to pathways (the feature mapping step is redundant)
- The goal is group-level pathway enrichment rather than per-metabolite ID standardization feeding sample-level pathway modeling (MetPA/IMPaLA/MPEA-style analyses)
- The dataset has been shrunk for fast testing (e.g., metaboliteMeasurements[, 1:20]), so mapping-rate results would not be comparable to the reported 182/227

## Inputs

- Metabolite measurement CSV (e.g., plasma_breast_cancer.csv from system.file("extdata", package = "lilikoi")) with metabolite names as column names
- Vector of raw metabolite names (e.g., colnames(metaboliteMeasurements))
- Lilikoi's bundled HMDB, SMPDB, and MetaboAnalyst database snapshots (optionally refreshed via lilikoi.update_database())

## Outputs

- metabolitePathwayTable: one row per input metabolite with matched standard HMDB ID and assigned pathway(s)
- Written mapping file, e.g., metabolite_pathway_mapping.csv
- Mapping-rate count (e.g., 182/227 standard HMDB IDs for the plasma breast cancer dataset)

## How to apply

Load the measurement CSV (e.g., plasma_breast_cancer.csv from the lilikoi package extdata with read.csv(row.names = 1, check.names = FALSE)) and extract the 227 metabolite column names. Run lilikoi.metab_to_pathway(metaboliteNames, "name") so each name is matched by Levenshtein-distance similarity against the metabolites stored in Lilikoi's internal HMDB/SMPDB/MetaboAnalyst databases shipped with the package; if databases may be stale, run lilikoi.update_database() first (warning: datasets are >5GB and can take >20 minutes). Write the returned metabolitePathwayTable to a CSV recording each input metabolite's matched standard ID and assigned pathway(s). Count how many metabolites received a standard HMDB ID and compute the mapping rate; for the plasma dataset the expected result is 182 of 227 (~80%), with the ~20% mismatch attributed to the Levenshtein-distance name-similarity measure. The mapping table must contain 227 rows (one per input metabolite) and becomes the input to lilikoi.get_pd_scores for downstream pathway deregulation scoring.

## Related tools

- **lilikoi** (Provides the feature mapping module lilikoi.metab_to_pathway and the internal HMDB/SMPDB/MetaboAnalyst database snapshots used for name-based standardization; lilikoi.update_database() refreshes those databases) — https://github.com/lanagarmire/lilikoi
- **R** (Runtime environment for loading the CSV, extracting metabolite names, and running lilikoi functions) — https://cran.r-project.org/web/packages/lilikoi/index.html

## Examples

```
metaboliteMeasurements <- read.csv(system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi"), check.names = FALSE, row.names = 1); metabolitePathwayTable <- lilikoi.metab_to_pathway(colnames(metaboliteMeasurements)[-1], "name")
```

## Evaluation signals

- The mapping table contains exactly 227 rows, one per input metabolite in plasma_breast_cancer.csv
- Exactly 182 metabolites receive standard HMDB IDs, matching the reported 182/227 (~80%) mapping rate
- Unmapped metabolites (~20%) are consistent with the paper's stated ~20% that cannot be mapped to standard names
- Every mapped metabolite has a resolvable standard ID drawn from Lilikoi's HMDB/SMPDB/MetaboAnalyst snapshots
- The resulting table feeds lilikoi.get_pd_scores without unmatched-ID errors

## Limitations

- About 20% of metabolites cannot be mapped to standard names in the databases
- Levenshtein distance similarity between the user's query metabolites and Lilikoi's stored metabolites is a possible cause of name mismatching
- The package ships with database snapshots as of publication; updating via lilikoi.update_database() involves large datasets (>5GB) and may take over 20 minutes

## Evidence

- [abstract] feature mapping module, which standardizes the metabolite names provided by users and maps them to pathways: "feature mapping module, which standardizes the metabolite names provided by users and maps them to pathways"
- [results] In the first plasma sample set, 182 of 227 metabolites are mapped to standard HMDB IDs: "In the first plasma sample set, 182 of 227 metabolites are mapped to standard HMDB IDs"
- [methods] metabolitePathwayTable <- lilikoi.metab_to_pathway(metaboliteNames, "name"): "metabolitePathwayTable <- lilikoi.metab_to_pathway(metaboliteNames, "name")"
- [discussion] One possible reason for this mismatching is that we used the Levenshtein distance as a measure of the similarity between the user's query metabolites and the metabolites stored in Lilikoi's database: "One possible reason for this mismatching is that we used the Levenshtein distance as a measure of the similarity between the user's query metabolites and the metabolites stored in Lilikoi's database"
- [readme] Warning: the datasets are large (>5GB) and this step may take greater than 20 minutes.: "Warning: the datasets are large (>5GB) and this step may take greater than 20 minutes."
