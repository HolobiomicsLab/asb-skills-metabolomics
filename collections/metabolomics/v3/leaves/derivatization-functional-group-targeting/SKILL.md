---
name: derivatization-functional-group-targeting
description: 'Use when annotating MALDI-MSI m/z features acquired with a derivatizing
  matrix (e.g., FMP-10): you must first restrict the candidate metabolite pool to
  molecules containing functional groups specific to that derivatization agent rather
  than using the full database.'
when_to_use_negative:
- MALDI-MSI data acquired with conventional (non-derivatizing) matrices, where the
  common [M+H]+ / [M-H]- adducts apply and a functional-group-reaction filter is unnecessary
- Metabolite identification for LC-MS workflows built on databases like Metlin or
  GNPS, which Met-ID treats as separate from its derivatization-aware HMDB/LIPID MAPS
  SQLite approach
- The analysis goal is MS2 similarity matching against the built-in FMP-10-derivatized
  standards spectra rather than constructing the MS1 candidate pool
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Met-ID
  role: Provides the MetIDServer context manager, SQLite database browsing (list_metabolites,
    list_matrices, list_functional_groups) and the derivatization-matrix functional-group
    filter used to build the reduced target pool
  repo: github.com/pbjarterot/Met-ID
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:d9eef9a1acac829767fcd0a60e1a59b9b2631e2c45f8893a799508fa36e5e771
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Met-ID
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# derivatization-functional-group-targeting

## Summary

Reduces a metabolite reference database (e.g., parsed HMDB in SQLite) to only those molecules bearing functional groups reactive to a specific derivatizing matrix, such as FMP-10 in MALDI-MSI, before mass matching. This shrinks the candidate pool (e.g., 26,202 all-HMDB metabolites to 4,951 endogenous targets) and cuts false positives in Met-ID annotation.

## When to use

Use when annotating MALDI-MSI m/z features acquired with a derivatizing matrix (e.g., FMP-10): you must first restrict the candidate metabolite pool to molecules containing functional groups specific to that derivatization agent rather than using the full database.

## When NOT to use

- MALDI-MSI data acquired with conventional (non-derivatizing) matrices, where the common [M+H]+ / [M-H]- adducts apply and a functional-group-reaction filter is unnecessary
- Metabolite identification for LC-MS workflows built on databases like Metlin or GNPS, which Met-ID treats as separate from its derivatization-aware HMDB/LIPID MAPS SQLite approach
- The analysis goal is MS2 similarity matching against the built-in FMP-10-derivatized standards spectra rather than constructing the MS1 candidate pool

## Inputs

- HMDB metabolite database parsed into an SQLite database readable by Met-ID (optionally LIPID MAPS as well)
- Derivatizing-matrix specification, e.g., the FMP-10 matrix entry with its derivatization-specific functional groups (browsable via list_matrices() and list_functional_groups())
- Biospecimen-subset selector, e.g., typ='All' versus the Endogenous-only subset of HMDB records

## Outputs

- Filtered candidate-pool sizes per database subset (all-HMDB versus endogenous), e.g., written to fmp10_candidate_pools.csv
- A reduced target-molecule subset containing only functional groups specific to the derivatization agent, usable as input to Met-ID's identify() for MS1 mass matching

## How to apply

Parse HMDB (and optionally LIPID MAPS) into the SQLite format Met-ID reads, then start a local Met-ID server with MetIDServer as a context manager and load the parsed database via the client's list_metabolites(). Select the derivatizing matrix (defaults select 'HMDB(All)' and the 'FMP-10' matrix) and apply the derivatization-specific functional-group filter over the full database with typ='All' so Exogenous, Endogenous, and Unspecified HMDB biospecimen records are all included; then re-apply the identical filter restricted to the Endogenous subset to emulate on-tissue endogenous metabolites. The rationale is that a derivatizing matrix like FMP-10 produces ions other than the common [M+H]+/[M-H]-, so only molecules with the reactive functional groups can yield the observed derivatives. Use the filtered pools as targets passed to identify() for MS1 mass matching, and optionally recalibrate with known m/z ratios to boost confidence. Tabulate the filtered pool sizes per subset (e.g., to fmp10_candidate_pools.csv) and check them against expected reference values.

## Related tools

- **Met-ID** (Provides the MetIDServer context manager, SQLite database browsing (list_metabolites, list_matrices, list_functional_groups) and the derivatization-matrix functional-group filter used to build the reduced target pool) — github.com/pbjarterot/Met-ID

## Examples

```
with MetIDServer() as server: server.list_metabolites()
```

## Evaluation signals

- Applying the FMP-10 functional-group filter with typ='All' over the full parsed HMDB must yield 26,202 candidate targets
- Re-applying the same filter restricted to the Endogenous subset must yield 4,951 candidate targets, matching the article's reported value
- The two filtered pool sizes should be tabulated side-by-side per subset (e.g., in fmp10_candidate_pools.csv) for reproducibility
- Cross-check the selected matrix and its reactive functional-group set via list_matrices() and list_functional_groups() before filtering
- Every retained candidate should contain at least one functional group specific to the chosen derivatization agent; molecules lacking such groups must be excluded

## Limitations

- HMDB records classified as Unspecified cannot be assigned to the endogenous subset, so the endogenous-only pool (4,951) excludes them while the full pool (26,202) includes Exogenous, Endogenous, and Unspecified records
- The filter is functional-group-level and generates a potential target subset, not a list of observed derivatives; downstream MS1/MS2 annotation is still required to remove false positives
- The FMP-10 derivatizing matrix was developed in-house and its availability/distribution to other users is not stated; Met-ID is nonetheless extendable to other derivatizing matrices via local functional-group additions
- On MacOS there are currently issues with adding functional groups, and reinstalling Met-ID does not update bundled database files, which must be removed manually (the 'com.farmbio.metid' folder) before reinstall

## Evidence

- [abstract] It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent.: "It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent."
- [methods] The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix.: "The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix."
- [methods] Import ``MetIDServer`` and use it as a context manager.: "Import ``MetIDServer`` and use it as a context manager."
- [readme] Met-ID has a particular focus on derivatizing matrices leading to other ions than the common [M+H]+ in positive mode and [M-H]- in negative mode.: "Met-ID has a particular focus on derivatizing matrices leading to other ions than the common [M+H]+ in positive mode and [M-H]- in negative mode."
