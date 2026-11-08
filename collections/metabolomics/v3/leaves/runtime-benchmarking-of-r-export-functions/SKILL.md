---
name: runtime-benchmarking-of-r-export-functions
description: Use when comparing which R metabolomics packages can read or write a
  spectral-library format (NIST msp, MoNA msp, GNPS mgf), or when runtime and attribute
  round-trip behaviour of export functions must be measured.
when_to_use_negative:
- Input is already a feature table or processed metabolomics result rather than a
  raw spectral-library file in msp/mgf dialects
- The comparison targets non-R tooling or packages outside the reviewed metaRbolomics
  set (metaMS, OrgMassSpecR, enviGCMS, TargetSearch, MSnbase/MSnio, compMS2Miner)
- Only one package and one dialect are of interest — a direct round-trip test without
  the comparative matrix is then sufficient
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Programming and statistics environment in which the reader/writer functions
    are exercised and timed
- name: metaMS
  role: R package whose NIST msp reader and export function are benchmarked for capability
    and runtime
- name: TargetSearch
  role: R package tested for NIST msp export support
- name: enviGCMS
  role: R package tested for NIST msp export support
- name: OrgMassSpecR
  role: R package tested for NIST msp export support
- name: MSnbase/MSnio
  role: R package tested for spectral-library dialect reading (parse success/failure
    per dialect)
- name: compMS2Miner
  role: R package tested for spectral-library dialect reading (parse success/failure
    per dialect)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo9100200
    title: The metaRbolomics Toolbox in Bioconductor and beyond
  source_content_hash: sha256:71684b0ec735925e874566aa8e75258809b5fcfcdc4a3c82bb3cce507fc772ba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSnbase/MSnio
  - OrgMassSpecR
  - R
  - TargetSearch
  - compMS2Miner
  - enviGCMS
  - metaMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# runtime-benchmarking-of-r-export-functions

## Summary

Benchmark R package reader/writer functions for NIST msp spectral-library import/export across dialects (basic NIST, canonical NIST, RIKEN PRIME, MoNA msp, GNPS mgf), recording capability matrices and measured export runtimes (e.g. metaMS ~20 min for an 8 MB file) as in the metaRbolomics review.

## When to use

Use when comparing which R metabolomics packages can read or write a spectral-library format (NIST msp, MoNA msp, GNPS mgf), or when runtime and attribute round-trip behaviour of export functions must be measured.

## When NOT to use

- Input is already a feature table or processed metabolomics result rather than a raw spectral-library file in msp/mgf dialects
- The comparison targets non-R tooling or packages outside the reviewed metaRbolomics set (metaMS, OrgMassSpecR, enviGCMS, TargetSearch, MSnbase/MSnio, compMS2Miner)
- Only one package and one dialect are of interest — a direct round-trip test without the comparative matrix is then sufficient

## Inputs

- Supplemental File S1 reference spectral-library snippets (Listing S1 'basic NIST', Listing S2 'canonical NIST', Listing S3 'RIKEN PRIME')
- Public MoNA msp export files
- Public GNPS mgf export files

## Outputs

- CSV capability matrix with one row per R package, dialect and attribute columns, and boolean/enumerated support entries
- Measured export runtimes per package (e.g. metaMS export runtime of ~20 min for an 8 MB file)

## How to apply

Parse the reference spectral-library snippets from Supplemental File S1 (Listing S1 'basic NIST', Listing S2 'canonical NIST', Listing S3 'RIKEN PRIME') with the reader functions of each R package (metaMS, OrgMassSpecR, enviGCMS, TargetSearch, MSnbase/MSnio, compMS2Miner), recording parse success/failure per package per dialect. Repeat the read benchmark on public MoNA msp and GNPS mgf export files to cover the two public exchange dialects. For each package, run its writer functions on the parsed spectra and record which dialect and which attributes (e.g. InChIKey, Collision_energy) survive import/export round-trips. Time the metaMS export step on the reference content (measured ~20 min for an 8 MB file) with system.time, and assemble a package x dialect/attribute capability matrix as a CSV with boolean/enumerated support entries, one row per package.

## Related tools

- **R** (Programming and statistics environment in which the reader/writer functions are exercised and timed)
- **metaMS** (R package whose NIST msp reader and export function are benchmarked for capability and runtime)
- **TargetSearch** (R package tested for NIST msp export support)
- **enviGCMS** (R package tested for NIST msp export support)
- **OrgMassSpecR** (R package tested for NIST msp export support)
- **MSnbase/MSnio** (R package tested for spectral-library dialect reading (parse success/failure per dialect))
- **compMS2Miner** (R package tested for spectral-library dialect reading (parse success/failure per dialect))

## Evaluation signals

- CSV validation: one row per package, dialect and attribute columns present, support entries in valid boolean/enumerated range
- metaMS export runtime reported in the correct order of magnitude (~20 min for an 8 MB file)
- Read benchmark covers all three S1 dialects (basic NIST, canonical NIST, RIKEN PRIME) plus MoNA msp and GNPS mgf
- Round-trip report records survival of attributes such as InChIKey and Collision_energy for each package
- Export-capability findings reproduce the review's claim that metaMS, OrgMassSpecR, enviGCMS, and TargetSearch support NIST msp export

## Limitations

- Export runtime can be prohibitive: the metaMS export step on reference content was measured at ~20 min for an 8 MB file
- Dialect fragmentation: packages differ in which NIST msp variants (basic, canonical, RIKEN PRIME, MoNA, GNPS mgf) they parse, so a package may fail on some dialects while succeeding on others
- Not all attributes (e.g. InChIKey, Collision_energy) survive import/export round-trips in every package

## Evidence

- [abstract] The programming and statistics environment R has emerged as one of the most popular environments to process and analyse Metabolomics datasets.: "The programming and statistics environment R has emerged as one of the most popular environments to process and analyse Metabolomics datasets."
- [abstract] This review provides an extensive overview of existing packages in R for different steps in a typical computational metabolomics workflow: "This review provides an extensive overview of existing packages in R for different steps in a typical computational metabolomics workflow, including data processing, biostatistics, metabolite"
- [abstract] In total, this review summarises more than two hundred metabolomics specific packages primarily available on CRAN, Bioconductor and GitHub.: "In total, this review summarises more than two hundred metabolomics specific packages primarily available on CRAN, Bioconductor and GitHub."
