---
name: msms-fragment-simulation-for-n-me-derivatized-sterols
description: Use when building a spectral library or 4D identification database for
  N-Me derivatized unsaturated sterol lipids, and theoretical fragment ions matching
  N-Me fragmentation patterns are needed for LC-IM-MS/MS annotation.
when_to_use_negative:
- Molecules without C=C double bonds or not carrying the N-Me derivatization, since
  fragmentation rules target double bond positions of N-Me derived unsaturated sterols
- When experimental MS/MS spectra are already available for library building — this
  skill provides in silico predictions, not measured spectra
- Non-sterol lipid classes, which are theoretically covered but were not tested in
  the source work
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: RDkit
  role: Basis of the MS/MS calculation script; built-in functions recognize double
    bond positions and generate fragment ions for N-Me derivatized sterols
- name: QCCAssisted4DSterol
  role: Collection of Python/Jupyter Notebook scripts implementing the MS/MS calculation
    for N-Me derived unsaturated sterol lipids
  repo: github.com/Chen-micslab/QCCAssisted4DSterol
- name: seokhokang/retention_time_gnn
  role: Companion pretrained GNN supplying the retention time dimension that is combined
    with the simulated fragment ions in the 4D database
  repo: https://github.com/seokhokang/retention_time_gnn/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/anie.202507483
    title: Quantum Chemistry Calculation‐Assisted Large‐Scale Collision Cross Section
      Prediction Empowers Derivatization‐Enhanced Multidimensional Metabolomics
  source_content_hash: sha256:082d959bc50e78ca4cfd8b3ab1054b5dff32c6a8387574fade8d5c03fde0d454
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Jupyter Notebook
  - Python
  - QCCAssisted4DSterol
  - RDkit
  - scikit-learn
  - seokhokang/retention_time_gnn
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# MS/MS fragment simulation for N-Me derivatized sterols

## Summary

An RDkit-based in silico MS/MS calculation that recognizes C═C double bond positions and generates fragment ions according to N-Me fragmentation patterns, supplying the fragment-ion dimension of a 4D sterol lipid database (m/z, RT, CCS, fragments; n = 4891).

## When to use

Use when building a spectral library or 4D identification database for N-Me derivatized unsaturated sterol lipids, and theoretical fragment ions matching N-Me fragmentation patterns are needed for LC-IM-MS/MS annotation.

## When NOT to use

- Molecules without C=C double bonds or not carrying the N-Me derivatization, since fragmentation rules target double bond positions of N-Me derived unsaturated sterols
- When experimental MS/MS spectra are already available for library building — this skill provides in silico predictions, not measured spectra
- Non-sterol lipid classes, which are theoretically covered but were not tested in the source work

## Inputs

- Derivatized sterol lipid structures (N-Me tagged, C═C bond-targeted) as RDkit-readable molecule representations
- N-Me fragmentation pattern rules (as encoded in the QCCAssisted4DSterol MS/MS calculation script)

## Outputs

- Per-molecule theoretical fragment ion lists following N-Me fragmentation patterns
- Fragment ion field of the 4D sterol lipid database (sterol_4d_database.csv, combined with m/z, RT, and CCS; n = 4891 entries)

## How to apply

Write the simulation on the basis of RDkit's built-in functions, as implemented in the QCCAssisted4DSterol project. Input each derivatized sterol lipid structure; the script recognizes double bond positions and generates MS/MS based on N-Me fragmentation patterns, so the derivatization state (C═C bond-targeted N-Me tagging) must be encoded in the molecule before simulation. Combine the resulting per-molecule fragment ion lists with independently predicted retention time (pretrained GNN from seokhokang/retention_time_gnn), predicted CCS (LASSO-selected, cross-validated SVR via scikit-learn), and m/z into a single table keyed by sterol lipid identity, written as sterol_4d_database.csv. The theoretical fragments are then matched against experimental LC-IM-MS/MS spectra during 4D sterolomics data processing and identification. Although the script is theoretically applicable to all molecules containing C=C bonds, it has only been tested on sterol lipids.

## Related tools

- **RDkit** (Basis of the MS/MS calculation script; built-in functions recognize double bond positions and generate fragment ions for N-Me derivatized sterols)
- **QCCAssisted4DSterol** (Collection of Python/Jupyter Notebook scripts implementing the MS/MS calculation for N-Me derived unsaturated sterol lipids) — github.com/Chen-micslab/QCCAssisted4DSterol
- **seokhokang/retention_time_gnn** (Companion pretrained GNN supplying the retention time dimension that is combined with the simulated fragment ions in the 4D database) — https://github.com/seokhokang/retention_time_gnn/

## Evaluation signals

- The assembled database contains the reported n = 4891 derivatized sterol lipid entries
- Every entry carries all four 4D fields: retention time, CCS, m/z, and fragment ions
- Generated fragment spectra are consistent with known N-Me fragmentation patterns for the assigned double bond positions
- Fragment ions match experimental LC-IM-MS/MS spectra during downstream sterol identification in ./Search/tissue workflows

## Limitations

- The script is theoretically applicable to all molecules including C=C bond but has only been tested on sterol lipids
- Fragment simulation depends on correct encoding of double bond positions and N-Me derivatization state in the input structure
- No changelog found in the source repository, so version-specific behavior of the script is not documented

## Evidence
