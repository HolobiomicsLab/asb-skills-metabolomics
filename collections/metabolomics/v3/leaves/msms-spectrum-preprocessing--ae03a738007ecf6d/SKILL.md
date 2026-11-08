---
name: msms-spectrum-preprocessing
description: Use when raw LC-MS/MS data (or MZmine output) must be converted to .mzXML,
  MS2 centroided, exported as MS2-only .csv plus .mgf spectra, and library-matched
  in GNPS2 FBMN to reproduce cosine/paired-peak annotations.
when_to_use_negative:
- The task is feature filtering or scheduled precursor list (SPL) construction from
  a full-scan .csv — that upstream step is RegFilter's job (npts-based filtering);
  this skill starts from already-acquired MS/MS data.
- Input is raw vendor data not yet converted to .mzXML — MSConvert conversion must
  precede MZmine 2.53 preprocessing in this workflow.
- The goal is purely in silico annotation (e.g., the SIRIUS 5.8.5 layer) rather than
  spectral pairing against a curated in-house/GNPS2 library — no .mgf library match
  is produced in that case.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0593
tools:
- name: MSConvert
  role: Converts raw MS files to .mzXML format before MZmine preprocessing
- name: MZmine 2.53
  role: Preprocesses full scan and MS/MS modes separately (MS-level-2 Centroid detection,
    noise 1E2; 0.001 m/z or 15 ppm tolerance) and exports the MS2-filtered .csv plus
    .mgf via 'Export/Submit to GNPS-FBMN'
- name: GNPS2 FBMN workflow
  role: Imports the .csv and .mgf and performs spectral pairing annotation of common-ion
    nodes against the curated in-house/GNPS2 library, returning cosine scores and
    paired-peak counts
- name: RegFilter
  role: 'Upstream: builds the scheduled precursor list (SPL) from the filtered full-scan
    .csv, defining which ions are targeted for the MS/MS acquisition preprocessed
    here'
  repo: https://github.com/computational-chemical-biology/regression_filter
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:c2626e68ed951a692d592f4fa4ecfa403a9dbd5c0e603e79d0fa7b4554b2599e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/computational-chemical-biology/regression_filter
  repo_url: https://github.com/computational-chemical-biology/regression_filter
  tools:
  - GNPS2
  - GNPS2 FBMN workflow
  - MSConvert
  - MZmine
  - MZmine 2.53
  - RegFilter
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# msms-spectrum-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocesses SPL- and DDA-acquired LC-MS/MS data of actinomycete extracts by converting raw files to .mzXML (MSConvert), centroiding MS2 in MZmine 2.53 (Centroid detector, noise 1E2), exporting MS2-only .csv feature tables and .mgf paired fragmentation spectra, and searching them in the GNPS2 FBMN workflow against curated in-house/GNPS2 libraries for cosine scores and paired-peak counts.

## When to use

Use when raw LC-MS/MS data (or MZmine output) must be converted to .mzXML, MS2 centroided, exported as MS2-only .csv plus .mgf spectra, and library-matched in GNPS2 FBMN to reproduce cosine/paired-peak annotations.

## When NOT to use

- The task is feature filtering or scheduled precursor list (SPL) construction from a full-scan .csv — that upstream step is RegFilter's job (npts-based filtering); this skill starts from already-acquired MS/MS data.
- Input is raw vendor data not yet converted to .mzXML — MSConvert conversion must precede MZmine 2.53 preprocessing in this workflow.
- The goal is purely in silico annotation (e.g., the SIRIUS 5.8.5 layer) rather than spectral pairing against a curated in-house/GNPS2 library — no .mgf library match is produced in that case.

## Inputs

- Raw MS files converted to .mzXML format (via MSConvert) from SPL- and DDA-acquired LC-MS/MS runs of actinomycete organic extracts
- MZmine 2.53 preprocessing output with separately processed full-scan (MS1) and MS/MS (MS2) peak lists
- Feature quantification table (.csv) exported with 'Filter rows: Only with MS2'
- Paired fragmentation spectra list (.mgf)
- Curated in-house/GNPS2 spectral library ([M+H]+ adduct entries, 15 ppm tolerance)

## Outputs

- MS2-filtered feature quantification table (.csv) containing only rows with MS2 spectra
- .mgf file of centroided MS2 paired fragmentation spectra (Centroid detector, noise level 1E2)
- GNPS2 FBMN library-match table with cosine similarity scores and paired-peak counts per strain (reference pairs: 0.9565/25 BRA006, 0.3228/10 BRA010, 0.6048/13 BRA177)

## How to apply

Convert raw MS files to .mzXML with MSConvert, then preprocess full scan and MS/MS modes as separate MZmine 2.53 jobs; for MS2, run mass detection with the Centroid mass detector at noise level 1E2, keeping the pipeline-wide m/z tolerance of 0.001 m/z or 15 ppm. Export results through 'Export/Submit to GNPS-FBMN' with 'Filter rows: Only with MS2', which yields a feature quantification table (.csv, peak area columns) restricted to MS2-bearing features plus a paired fragmentation spectra list (.mgf). Import both files into the GNPS2 Feature Based Molecular Network (FBMN) workflow and perform spectral pairing of the common-ion nodes against the curated in-house/GNPS2 spectral library, considering [M+H]+ adducts at 15 ppm tolerance. For each matched node, extract the library hit, the cosine similarity score, and the number of paired (matched) peaks, then compile the per-strain table for BRA006, BRA010, and BRA177. The rationale is that MS2-only filtering and separate MS1/MS2 preprocessing keep the .mgf spectra clean for library matching; any deviation from the reference pairs (0.9565/25, 0.3228/10, 0.6048/13) requires expert review of the library match.

## Related tools

- **MSConvert** (Converts raw MS files to .mzXML format before MZmine preprocessing)
- **MZmine 2.53** (Preprocesses full scan and MS/MS modes separately (MS-level-2 Centroid detection, noise 1E2; 0.001 m/z or 15 ppm tolerance) and exports the MS2-filtered .csv plus .mgf via 'Export/Submit to GNPS-FBMN')
- **GNPS2 FBMN workflow** (Imports the .csv and .mgf and performs spectral pairing annotation of common-ion nodes against the curated in-house/GNPS2 library, returning cosine scores and paired-peak counts)
- **RegFilter** (Upstream: builds the scheduled precursor list (SPL) from the filtered full-scan .csv, defining which ions are targeted for the MS/MS acquisition preprocessed here) — https://github.com/computational-chemical-biology/regression_filter

## Evaluation signals

- The exported .csv contains only rows with MS2 spectra, mirroring 'Filter rows: Only with MS2' in the Export/Submit to GNPS-FBMN step.
- The .mgf contains one centroided MS2 spectrum per retained feature, generated with MS-level-2 Centroid mass detection at noise level 1E2.
- GNPS2 FBMN library search under [M+H]+, 15 ppm tolerance reproduces the reference cosine/paired-peak pairs: 0.9565/25 (BRA006), 0.3228/10 (BRA010), 0.6048/13 (BRA177).
- All m/z tolerances in the preprocessing chain match the pipeline value of 0.001 m/z or 15 ppm.
- Full-scan and MS/MS preprocessing were executed as separate MZmine jobs, not combined.

## Limitations

- DDA acquisition is stochastic: ions of similar intensity may or may not be fragmented, so DDA-derived MS2 spectra alone miss precursors that the SPL flow captures.
- Low cosine scores (e.g., BRA010 at 0.3228 with 10 paired peaks) indicate weak library matches; any deviation from the reported pairs requires expert review of the library match.
- Match quality depends on a curated spectral library; the workflow is intended for in-house libraries curated according to the user's interests.

## Evidence
