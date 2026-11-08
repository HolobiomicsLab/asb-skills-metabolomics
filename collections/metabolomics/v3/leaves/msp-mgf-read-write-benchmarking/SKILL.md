---
name: msp-mgf-read-write-benchmarking
description: Use when selecting an R package to read or export spectral libraries
  in NIST msp, MoNA msp, or GNPS mgf format, or when verifying that attributes like
  InChIKey and Collision_energy survive a read/write round-trip.
when_to_use_negative:
- The input is already a parsed in-memory spectral library rather than an msp/mgf
  exchange file
- The goal is raw MS data processing (e.g. peak picking or alignment) rather than
  spectral-library format interoperability
- Only a single format and single package are involved, so a comparative capability
  matrix adds no value
tools:
- name: R
  role: Scripting environment in which the reader/writer benchmark functions of all
    reviewed packages are executed
- name: metaMS
  role: Reviewed package whose NIST msp reader and writer are benchmarked and whose
    export step is timed (~20 min for 8 MB)
- name: OrgMassSpecR
  role: Reviewed package reported to support export to the NIST msp format; its reader/writer
    are exercised in the benchmark
- name: enviGCMS
  role: Reviewed package reported to support export to the NIST msp format; its reader/writer
    are exercised in the benchmark
- name: TargetSearch
  role: Reviewed package reported to support export to the NIST msp format; its reader/writer
    are exercised in the benchmark
- name: MSnbase/MSnio
  role: Reviewed package whose reader functions are tested against the msp dialects
    in the benchmark
- name: compMS2Miner
  role: Reviewed package whose reader functions are tested against the msp dialects
    in the benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo9100200
    title: The metaRbolomics Toolbox in Bioconductor and beyond
  source_content_hash: sha256:d5cbaf5561531a83c4f8c5994b4760104efd70d473750d4f336b1d7dfb14c24f
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# msp-mgf-read-write-benchmarking

## Summary

A systematic benchmark of R package support for reading and writing spectral-library exchange dialects (NIST msp variants, MoNA msp, GNPS mgf), producing a package x dialect/attribute capability matrix with measured export runtimes.

## When to use

Use when selecting an R package to read or export spectral libraries in NIST msp, MoNA msp, or GNPS mgf format, or when verifying that attributes like InChIKey and Collision_energy survive a read/write round-trip.

## When NOT to use

- The input is already a parsed in-memory spectral library rather than an msp/mgf exchange file
- The goal is raw MS data processing (e.g. peak picking or alignment) rather than spectral-library format interoperability
- Only a single format and single package are involved, so a comparative capability matrix adds no value

## Inputs

- Supplemental File S1 reference spectral-library snippets (Listing S1 'basic NIST', Listing S2 'canonical NIST', Listing S3 'RIKEN PRIME')
- Public MoNA msp export files
- Public GNPS mgf export files

## Outputs

- Package x dialect/attribute capability matrix as CSV with boolean/enumerated support entries, one row per package
- Per-package parse success/failure records per dialect
- Measured export runtimes (e.g. metaMS ~20 min for an 8 MB file)

## How to apply

Parse the reference spectral-library snippets (Supplemental File S1: Listing S1 'basic NIST', Listing S2 'canonical NIST', Listing S3 'RIKEN PRIME') with the reader functions of each candidate package (metaMS, OrgMassSpecR, enviGCMS, TargetSearch, MSnbase/MSnio, compMS2Miner) and record parse success/failure per dialect. Repeat the read benchmark on public MoNA msp and GNPS mgf export files to cover the two public exchange dialects. For each package, run the writer functions on the parsed spectra and record which dialect and which attributes (e.g. InChIKey, Collision_energy) survive import/export round-trips. Time the metaMS export step on the reference content — the article reports roughly 20 minutes for an 8 MB file — and record it in the matrix. Assemble the results as a CSV with one row per package and dialect/attribute columns holding boolean/enumerated support entries.

## Related tools

- **R** (Scripting environment in which the reader/writer benchmark functions of all reviewed packages are executed)
- **metaMS** (Reviewed package whose NIST msp reader and writer are benchmarked and whose export step is timed (~20 min for 8 MB))
- **OrgMassSpecR** (Reviewed package reported to support export to the NIST msp format; its reader/writer are exercised in the benchmark)
- **enviGCMS** (Reviewed package reported to support export to the NIST msp format; its reader/writer are exercised in the benchmark)
- **TargetSearch** (Reviewed package reported to support export to the NIST msp format; its reader/writer are exercised in the benchmark)
- **MSnbase/MSnio** (Reviewed package whose reader functions are tested against the msp dialects in the benchmark)
- **compMS2Miner** (Reviewed package whose reader functions are tested against the msp dialects in the benchmark)

## Evaluation signals

- CSV contains exactly one row per reviewed package (metaMS, OrgMassSpecR, enviGCMS, TargetSearch, MSnbase/MSnio, compMS2Miner)
- CSV has explicit dialect and attribute columns; all support entries fall in a valid boolean/enumerated range
- metaMS export runtime on the 8 MB reference content is on the order of ~20 minutes
- Round-trip check records whether attributes such as InChIKey and Collision_energy survive import/export for each package
- Expected finding: metaMS, OrgMassSpecR, enviGCMS, and TargetSearch export to NIST msp, matching the review's Table 3 / Table S1 comparison

## Limitations

- Dialect coverage is limited to the three reference snippets in Supplemental File S1 plus MoNA msp and GNPS mgf exports; other vendor dialects are not covered
- Attribute round-trip survival is package- and dialect-dependent (e.g. InChIKey, Collision_energy may be dropped), so matrix entries are not transferable between dialects
- Export performance can be a practical bottleneck: metaMS export took roughly 20 minutes for an 8 MB file

## Evidence

- [abstract] R has emerged as one of the most popular environments to process and analyse Metabolomics datasets: "The programming and statistics environment R has emerged as one of the most popular environments to process and analyse Metabolomics datasets."
- [abstract] This review provides an extensive overview of existing packages in R for different steps in a typical computational metabolomics workflow: "This review provides an extensive overview of existing packages in R for different steps in a typical computational metabolomics workflow, including data processing, biostatistics, metabolite"
- [abstract] In total, this review summarises more than two hundred metabolomics specific packages primarily available on CRAN, Bioconductor and GitHub: "In total, this review summarises more than two hundred metabolomics specific packages primarily available on CRAN, Bioconductor and GitHub."
