---
name: metabolite-database-filtering
description: Use when preparing MALDI-MSI metabolite identification for a derivatizing
  matrix (e.g. FMP-10) and you need to restrict the target database to molecules bearing
  the functional groups the matrix reacts with, or to the endogenous subset of HMDB.
when_to_use_negative:
- Data were acquired with a conventional (non-derivatizing) matrix where standard
  [M+H]+ / [M-H]- adduct annotation suffices
- The target database is not HMDB or LIPID MAPS, since Met-ID's filtering operates
  on databases parsed into its SQLite format
- The goal is MS2 spectral comparison against standards rather than reducing the MS1
  mass-matching candidate pool
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3361
tools:
- name: Met-ID
  role: Provides the MetIDServer client, the parsed HMDB/LIPID MAPS SQLite databases,
    and the derivatization-matrix functional-group filter used to restrict candidate
    pools
  repo: https://github.com/pbjarterot/Met-ID
- name: HMDB
  role: Source metabolite database whose full (26,202) and endogenous (4,951) subsets
    are filtered by FMP-10 functional groups
- name: LIPID MAPS
  role: Secondary metabolite database that Met-ID can parse into SQLite and filter
    analogously
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:3cd3e3ad96e3acd7e0d43ff5085bd4090afed2610be4c73a6f3495c7a7554b0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pbjarterot/Met-ID
  tools:
  - HMDB
  - LIPID MAPS
  - Met-ID
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# metabolite-database-filtering

## Summary

Narrow the metabolite identification candidate space by applying a derivatization-specific functional-group filter to a parsed metabolite database (HMDB/LIPID MAPS in Met-ID's SQLite format), e.g. reducing the full HMDB pool of 26,202 metabolites to 4,951 endogenous candidates for the FMP-10 reactive matrix.

## When to use

Use when preparing MALDI-MSI metabolite identification for a derivatizing matrix (e.g. FMP-10) and you need to restrict the target database to molecules bearing the functional groups the matrix reacts with, or to the endogenous subset of HMDB.

## When NOT to use

- Data were acquired with a conventional (non-derivatizing) matrix where standard [M+H]+ / [M-H]- adduct annotation suffices
- The target database is not HMDB or LIPID MAPS, since Met-ID's filtering operates on databases parsed into its SQLite format
- The goal is MS2 spectral comparison against standards rather than reducing the MS1 mass-matching candidate pool

## Inputs

- Parsed HMDB SQLite database (as read by Met-ID, via list_metabolites())
- Derivatization matrix definition (e.g. FMP-10) with its specific functional groups
- Optional: parsed LIPID MAPS SQLite database

## Outputs

- Filtered candidate-pool size for the full database (all typ: Exogenous, Endogenous, Unspecified) — expected 26,202 for FMP-10 against full HMDB
- Filtered candidate-pool size for the endogenous-only subset — expected 4,951 for FMP-10
- Tabulated pool sizes (e.g. fmp10_candidate_pools.csv)

## How to apply

Parse HMDB (and optionally LIPID MAPS) into the SQLite database format Met-ID can read — the repository contains files for this parsing step. Start the local Met-ID server with MetIDServer as a context manager and enumerate records via the client's list_metabolites() method. Apply the derivatization-matrix functional-group filter (e.g. the FMP-10 matrix, whose defaults select 'HMDB(All)') with typ='All' so Exogenous, Endogenous, and Unspecified records are all included; this yields the full candidate pool of 26,202 metabolites. Re-apply the same FMP-10 functional-group filter restricted to the Endogenous subset to obtain the tissue-relevant pool of 4,951 candidates, and tabulate both pool sizes (e.g. to fmp10_candidate_pools.csv). The rationale, per the article, is that Met-ID 'utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent', which removes molecules that cannot form the matrix-derived ion species.

## Related tools

- **Met-ID** (Provides the MetIDServer client, the parsed HMDB/LIPID MAPS SQLite databases, and the derivatization-matrix functional-group filter used to restrict candidate pools) — https://github.com/pbjarterot/Met-ID
- **HMDB** (Source metabolite database whose full (26,202) and endogenous (4,951) subsets are filtered by FMP-10 functional groups)
- **LIPID MAPS** (Secondary metabolite database that Met-ID can parse into SQLite and filter analogously)

## Examples

```
from metid import MetIDServer; with MetIDServer() as server: mets = server.client.list_metabolites(); print(len(mets))
```

## Evaluation signals

- Full-HMDB pool filtered with the FMP-10 functional-group matrix (typ='All') must contain exactly 26,202 candidate targets
- Endogenous-only subset under the same FMP-10 filter must contain exactly 4,951 candidate targets, matching the article's reported values
- The endogenous pool size (4,951) must be strictly smaller than the all-typ pool size (26,202), confirming the subset restriction was applied
- Output CSV contains one row per queried subset (All vs Endogenous) with non-zero, integer pool counts
- list_metabolites() returns records before filtering, confirming the SQLite database files loaded are the intended (current) bundled versions

## Limitations

- Restricting to the endogenous subset excludes Exogenous and Unspecified HMDB records, which may drop relevant candidates for non-tissue or xenobiotic-focused experiments
- Reinstalling newer Met-ID versions does not update bundled database files; stale 'com.farmbio.metid' app-data folders must be removed manually so the filter runs against the intended database version
- The functional-group filter reflects only the derivatization chemistry encoded in Met-ID; custom matrices require local modifications (and adding functional groups is currently problematic on MacOS)

## Evidence

- [abstract] structural subsetting for derivatizing matrices: "It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent."
- [discussion] database basis: "Met-ID is based on the openly available HMDB and LIPID MAPS"
- [discussion] SQLite parsing: [evidence span withheld — anchor status: not_found]
- [methods] default filter settings: "The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix."
- [readme] derivatizing-matrix focus: "Met-ID has a particular focus on derivatizing matrices leading to other ions than the common [M+H]+ in positive mode and [M-H]- in negative mode."
