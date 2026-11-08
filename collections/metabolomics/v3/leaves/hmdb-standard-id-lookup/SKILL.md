---
name: hmdb-standard-id-lookup
description: Use when a metabolomics matrix's columns are raw metabolite names (e.g.,
  plasma_breast_cancer.csv) that must be converted to standard HMDB IDs before pathway
  mapping or PDS-score computation in Lilikoi.
when_to_use_negative:
- Input metabolites are already identified by standard HMDB/PubChem/METLIN IDs and
  only need pathway assignment, not name standardization
- You need group-level pathway enrichment (MetPA, IMPaLA, MPEA style) rather than
  sample-level ID standardization feeding a personalized PDS pipeline
- A very small fraction of query names will survive matching and the mapping step
  would add noise rather than value (expect ~20% of names to remain unmapped)
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: lilikoi
  role: Provides lilikoi.metab_to_pathway() plus the packaged HMDB/MetaboAnalyst synonym
    databases that perform exact, synonym, and Levenshtein fuzzy name-to-HMDB-ID matching
  repo: https://github.com/lanagarmire/lilikoi
- name: R
  role: Runtime environment in which the lilikoi package is installed (devtools::install_github)
    and the mapping CSV is read/written
  repo: https://cran.r-project.org/web/packages/lilikoi/index.html
- name: HMDB
  role: Source of the standard IDs that metabolite names are mapped to; Lilikoi depends
    on HMDB data shipped with the package or refreshed via lilikoi.update_database()
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giy136
    title: 'Lilikoi: an R package for personalized pathway-based classification modeling
      using metabolomics data'
  source_content_hash: sha256:ddfd73d4f0e8323d14e3e5f4e03cbd0a35c1c3d5aee96917617afd0fe9efb546
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi
  tools:
  - HMDB
  - R
  - lilikoi
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# hmdb-standard-id-lookup

## Summary

Standardizes user-provided metabolite names to standard HMDB IDs using Lilikoi's feature-pathway mapper, which chains exact matching, synonym-database lookup, and Levenshtein-distance fuzzy matching, and then maps IDs to pathways via lilikoi.metab_to_pathway. This is the first step of the Lilikoi metabolomics pipeline and gates all downstream pathway-based modeling.

## When to use

Use when a metabolomics matrix's columns are raw metabolite names (e.g., plasma_breast_cancer.csv) that must be converted to standard HMDB IDs before pathway mapping or PDS-score computation in Lilikoi.

## When NOT to use

- Input metabolites are already identified by standard HMDB/PubChem/METLIN IDs and only need pathway assignment, not name standardization
- You need group-level pathway enrichment (MetPA, IMPaLA, MPEA style) rather than sample-level ID standardization feeding a personalized PDS pipeline
- A very small fraction of query names will survive matching and the mapping step would add noise rather than value (expect ~20% of names to remain unmapped)

## Inputs

- CSV metabolite measurement matrix (e.g., plasma_breast_cancer.csv) with metabolite names as column headers
- Metabolite name vector (colnames(metaboliteMeasurements)[-1]) supplied as query strings
- Lilikoi bundled annotation databases (data.hmdb, data.metaboAnalyst, data.smpdb)

## Outputs

- metabolitePathwayTable: one row per input metabolite with matched standard HMDB ID and assigned pathway(s)
- metabolite-to-HMDB mapping rate (fraction of input metabolites assigned a standard ID)

## How to apply

Load the input CSV with system.file('extdata', 'plasma_breast_cancer.csv', package='lilikoi') and extract metabolite names via colnames(metaboliteMeasurements)[-1]. Call lilikoi.metab_to_pathway(metaboliteNames, 'name'), which attempts an exact match against Lilikoi's bundled HMDB-derived database, falls back to the synonym table (data.hmdb, data.metaboAnalyst), and finally applies Levenshtein edit-distance fuzzy matching for remaining unmatched names. Write the resulting metabolitePathwayTable to CSV, retaining each input name with its matched HMDB ID and assigned pathway(s). Compute the mapping rate as the fraction of inputs assigned a standard HMDB ID; expect roughly 80% for the plasma dataset (182 of 227) and acknowledge ~20% may stay unmapped. If fresher annotations are needed, run lilikoi.update_database() to pull updated HMDB, SMPDB, and MetaboAnalyst data (large, >5GB, >20 minutes).

## Related tools

- **lilikoi** (Provides lilikoi.metab_to_pathway() plus the packaged HMDB/MetaboAnalyst synonym databases that perform exact, synonym, and Levenshtein fuzzy name-to-HMDB-ID matching) — https://github.com/lanagarmire/lilikoi
- **R** (Runtime environment in which the lilikoi package is installed (devtools::install_github) and the mapping CSV is read/written) — https://cran.r-project.org/web/packages/lilikoi/index.html
- **HMDB** (Source of the standard IDs that metabolite names are mapped to; Lilikoi depends on HMDB data shipped with the package or refreshed via lilikoi.update_database())

## Examples

```
metaboliteNames <- colnames(read.csv(system.file('extdata','plasma_breast_cancer.csv',package='lilikoi'),check.names=FALSE,row.names=1))[-1]; metabolitePathwayTable <- lilikoi.metab_to_pathway(metaboliteNames, 'name')
```

## Evaluation signals

- Output CSV exists and is well-formed with one row per input metabolite
- Mapping rate on the plasma dataset is consistent with the paper's reported 182 of 227 metabolites (~80%) mapped to standard HMDB IDs
- Each matched entry carries both the original query name and a standard HMDB ID plus assigned pathway(s)
- Roughly 20% of metabolites legitimately remain unmapped; flag as expected, not as a pipeline error
- Downstream lilikoi.get_pd_scores() accepts the table without name-resolution errors

## Limitations

- About 20% of metabolites still cannot be mapped to standard names in the databases
- Levenshtein edit distance is used as the similarity measure for fuzzy matching and is a possible cause of metabolite name mismatching
- Bundled databases reflect data as of the publication date; updating via lilikoi.update_database() downloads >5GB and can take more than 20 minutes

## Evidence

- [results] Plasma mapping rate: "In the first plasma sample set, 182 of 227 metabolites are mapped to standard HMDB IDs"
- [results] Standardization step: "Lilikoi first transforms the metabolite names to standard IDs"
- [discussion] Levenshtein fuzzy matching: "we used the Levenshtein distance as a measure of the similarity between the user's query metabolites and the metabolites stored in Lilikoi's database"
- [discussion] Unmapped fraction: "a small percentage (20%) of the metabolites still cannot be mapped to the standard names in databases"
- [methods] Database dependencies: "Lilikoi depends on data from HMDB, SMPDB, and MetaboAnalyst."
- [readme] README invocation: "metabolitePathwayTable <- lilikoi.metab_to_pathway(metaboliteNames, "name")"
