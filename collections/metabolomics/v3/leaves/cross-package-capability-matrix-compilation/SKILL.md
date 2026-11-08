---
name: cross-package-capability-matrix-compilation
description: Use when selecting an R package to import/export spectral libraries in
  NIST msp or MoNA/GNPS dialects, and you need a reproducible support matrix rather
  than documentation claims.
when_to_use_negative:
- The input is already a processed feature table or quantification matrix rather than
  a spectral-library exchange file (msp/mgf) — this skill only tests spectral-database
  readers and writers.
- You need a single production-ready conversion rather than a comparative benchmark
  — pick the one package that already passes the dialect check instead of re-running
  the full matrix.
- The target format is neither NIST msp nor MoNA/GNPS mgf (e.g. native instrument
  formats), since the workflow's dialects and attribute round-trip checks do not cover
  it.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: R
  role: Programming environment used to run the package reader/writer benchmarks and
    assemble the capability matrix
- name: metaMS
  role: Package under test; supports export to the NIST msp file format but its export
    step is slow (~20 min for an 8 MB file)
- name: OrgMassSpecR
  role: Reviewed R package supporting export to the NIST msp file format; benchmarked
    for parse and round-trip attribute survival
- name: enviGCMS
  role: Reviewed R package supporting export to the NIST msp file format; benchmarked
    for parse and round-trip attribute survival
- name: TargetSearch
  role: Reviewed R package supporting export to the NIST msp file format; benchmarked
    for parse and round-trip attribute survival
- name: MSnbase/MSnio
  role: Package whose reader functions are tested against the S1 NIST dialect listings
    and public MoNA/GNPS exports
- name: compMS2Miner
  role: Package whose reader functions are tested against the S1 NIST dialect listings
    and public MoNA/GNPS exports
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo9100200
    title: The metaRbolomics Toolbox in Bioconductor and beyond
  source_content_hash: sha256:76de98458421bfc41ee0cdd8c25522a8965ca639efbc0ae38abd6da7e24bcc1e
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# cross-package-capability-matrix-compilation

## Summary

Systematically benchmarks metabolomics R packages (e.g. metaMS, OrgMassSpecR, enviGCMS, TargetSearch, MSnbase/MSnio, compMS2Miner) for their ability to read and write spectral-library exchange dialects such as NIST msp and GNPS mgf, and condenses the results into a package x dialect/attribute capability matrix. It reproduces the review-style comparisons of Table 3 and Table S1 with measurable parse success, attribute round-trip survival, and export runtimes.

## When to use

Use when selecting an R package to import/export spectral libraries in NIST msp or MoNA/GNPS dialects, and you need a reproducible support matrix rather than documentation claims.

## When NOT to use

- The input is already a processed feature table or quantification matrix rather than a spectral-library exchange file (msp/mgf) — this skill only tests spectral-database readers and writers.
- You need a single production-ready conversion rather than a comparative benchmark — pick the one package that already passes the dialect check instead of re-running the full matrix.
- The target format is neither NIST msp nor MoNA/GNPS mgf (e.g. native instrument formats), since the workflow's dialects and attribute round-trip checks do not cover it.

## Inputs

- Reference spectral-library snippets from Supplemental File S1 (basic NIST, canonical NIST, RIKEN PRIME listings)
- Public MoNA msp export file
- Public GNPS mgf export file
- Installed R packages with spectral-library reader/writer functions (metaMS, OrgMassSpecR, enviGCMS, TargetSearch, MSnbase/MSnio, compMS2Miner)

## Outputs

- CSV capability matrix with one row per package and dialect/attribute columns holding boolean or enumerated support entries
- Measured export runtime entries (e.g. ~20 min for the metaMS export of an 8 MB file)

## How to apply

Parse reference spectral-library snippets of Supplemental File S1 (Listing S1 'basic NIST', Listing S2 'canonical NIST', Listing S3 'RIKEN PRIME') with each package's reader functions and record parse success/failure per package per dialect. Repeat the read benchmark on public MoNA msp and GNPS mgf exports to cover the two public exchange dialects, then run each package's writer functions on the parsed spectra and record which dialect and which attributes (e.g. InChIKey, Collision_energy) survive the import/export round-trip. Time known-slow writers — the metaMS export step takes on the order of 20 minutes for an 8 MB reference file — and record this in the matrix. Assemble the results as a CSV capability matrix with one row per package, dialect and attribute columns, and boolean/enumerated support entries, following the review's Table 3/Table S1 layout.

## Related tools

- **R** (Programming environment used to run the package reader/writer benchmarks and assemble the capability matrix)
- **metaMS** (Package under test; supports export to the NIST msp file format but its export step is slow (~20 min for an 8 MB file))
- **OrgMassSpecR** (Reviewed R package supporting export to the NIST msp file format; benchmarked for parse and round-trip attribute survival)
- **enviGCMS** (Reviewed R package supporting export to the NIST msp file format; benchmarked for parse and round-trip attribute survival)
- **TargetSearch** (Reviewed R package supporting export to the NIST msp file format; benchmarked for parse and round-trip attribute survival)
- **MSnbase/MSnio** (Package whose reader functions are tested against the S1 NIST dialect listings and public MoNA/GNPS exports)
- **compMS2Miner** (Package whose reader functions are tested against the S1 NIST dialect listings and public MoNA/GNPS exports)

## Examples

```
matrix <- read.csv("package_capability_matrix.csv"); stopifnot(nrow(matrix) == 6, all(c("dialect","attribute") %in% names(matrix)))
```

## Evaluation signals

- CSV validation: exactly one row per reviewed package, with dialect and attribute columns present.
- All support entries fall within the valid boolean/enumerated range.
- The metaMS export runtime entry is in the reported order of magnitude (~20 min for the 8 MB reference file).
- Round-trip check: recorded survival of attributes such as InChIKey and Collision_energy matches the per-package writer output on the S1 reference listings.
- Read benchmark covers all three S1 dialects (basic NIST, canonical NIST, RIKEN PRIME) plus the MoNA msp and GNPS mgf public dialects for every package.

## Limitations

- The metaMS export step is slow (~20 min for an 8 MB file), so full-matrix generation must budget for long writer runtimes.
- Package support differs per dialect (basic NIST vs canonical NIST vs RIKEN PRIME vs MoNA msp vs GNPS mgf); parse success on one dialect does not imply success on others.
- Only four of the reviewed packages (metaMS, OrgMassSpecR, enviGCMS, TargetSearch) support NIST msp export; other packages are readers only.

## Evidence

- [abstract] This review provides an extensive overview of existing packages in R for different steps in a typical computational metabolomics workflow: "This review provides an extensive overview of existing packages in R for different steps in a typical computational metabolomics workflow"
- [abstract] In total, this review summarises more than two hundred metabolomics specific packages primarily available on CRAN, Bioconductor and GitHub.: "In total, this review summarises more than two hundred metabolomics specific packages primarily available on CRAN, Bioconductor and GitHub."
- [abstract] The programming and statistics environment R has emerged as one of the most popular environments to process and analyse Metabolomics datasets.: "The programming and statistics environment R has emerged as one of the most popular environments to process and analyse Metabolomics datasets."
