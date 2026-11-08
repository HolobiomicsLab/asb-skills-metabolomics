---
name: lc-msms-spectral-library-matching
description: Use when you have an aligned LC-MS/MS feature table (e.g. from MS-Dial
  after MS-CleanR filtering) and need to annotate detected features by matching their
  MS/MS spectra against a spectral library.
when_to_use_negative:
- Input is raw unprocessed mass spectra without an aligned feature list or deconvoluted
  MS/MS — feature detection and MS/MS deconvolution must precede library matching
- Libraries have not been harmonized or deduplicated (e.g. raw concatenated OMSLs
  with inconsistent metadata) — FragHub preprocessing or equivalent normalization
  is needed first
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MS-Dial
  role: Runs MS/MS-based spectral annotation of the aligned feature list against the
    .MSP library with default parameters
- name: FragHub
  role: Integrates and harmonizes OMSLs into a unified, filtered .MSP library ready
    for MS-Dial annotation
  repo: https://github.com/eMetaboHUB/FragHub
- name: MS-CleanR
  role: Filters detected features (blank ratio 0.8, relative SD 40%, relative mass
    defect 50–3500 ppm) before annotation
- name: spectra-hash (SPLASH)
  role: Generates SPLASH keys used by FragHub to deduplicate spectra and maintain
    a repository of treated spectra
  repo: https://github.com/berlinguyinca/spectra-hash
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:b4a7390e0739801dbe366901c834602f7c406c9b77dae5e31c6d835712a32d5f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/eMetaboHUB/FragHub
  repo_url: https://github.com/eMetaboHUB/FragHub
  tools:
  - FragHub
  - MS-CleanR
  - MS-Dial
  - spectra-hash (SPLASH)
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# LC-MS/MS spectral library matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Performs MS/MS-based spectral annotation of an aligned LC-MS feature list against consolidated open mass spectral libraries (e.g. FragHub-processed OMSLs exported as .MSP) in MS-Dial, turning detected features into putative compound annotations.

## When to use

Use when you have an aligned LC-MS/MS feature table (e.g. from MS-Dial after MS-CleanR filtering) and need to annotate detected features by matching their MS/MS spectra against a spectral library.

## When NOT to use

- Input is raw unprocessed mass spectra without an aligned feature list or deconvoluted MS/MS — feature detection and MS/MS deconvolution must precede library matching
- Libraries have not been harmonized or deduplicated (e.g. raw concatenated OMSLs with inconsistent metadata) — FragHub preprocessing or equivalent normalization is needed first

## Inputs

- Aligned LC-MS/MS feature list from MS-Dial (after MS-CleanR filtering, e.g. 435 positive-ion-mode features from A. thaliana)
- FragHub-processed open mass spectral library exported in .MSP format (or individual libraries: GNPS .MGF, MoNA .MSP/.SDF/.JSON, MassBank records)

## Outputs

- Per-library annotated-feature counts (e.g. FragHub = 102, MassBank = 41, GNPS/MoNA/MS-Dial-VS17 in the 55–81 range)
- Aggregated annotation result table (CSV) comparing number of annotated features across libraries

## How to apply

Export each FragHub-processed OMSL in .MSP format and import it into MS-Dial (v5.23 in the benchmark) alongside the aligned feature list obtained from thermo RAW QC and blank files. Run MS/MS-based spectral annotation of the aligned feature list using default MS-Dial parameters; repeat the run per library (FragHub-unified, MassBank, GNPS, MoNA, MS-Dial-VS17) to obtain per-library annotation counts. Aggregate the per-library counts into a single CSV annotation table to compare library performance. The FragHub-unified library is preferred as it deduplicates spectra via SPLASH keys and removes inconsistent spectra lacking SMILES, InChI, precursor m/z, or adduct information, yielding 602,744 spectra for 32,193 unique chemicals from the four integrated OMSLs.

## Related tools

- **MS-Dial** (Runs MS/MS-based spectral annotation of the aligned feature list against the .MSP library with default parameters)
- **FragHub** (Integrates and harmonizes OMSLs into a unified, filtered .MSP library ready for MS-Dial annotation) — https://github.com/eMetaboHUB/FragHub
- **MS-CleanR** (Filters detected features (blank ratio 0.8, relative SD 40%, relative mass defect 50–3500 ppm) before annotation)
- **spectra-hash (SPLASH)** (Generates SPLASH keys used by FragHub to deduplicate spectra and maintain a repository of treated spectra) — https://github.com/berlinguyinca/spectra-hash

## Examples

```
Run MS-Dial v5.23 with default parameters: import thermo RAW QC/blank files, align features, MS-CleanR filter (blank ratio 0.8, RSD 40%, RMD 50–3500 ppm), then MS/MS annotation against each FragHub-unified .MSP library.
```

## Evaluation signals

- Per-library annotated-feature counts should fall in expected ranges: FragHub ≈ 102 (24% of 435 features), MassBank ≈ 41, GNPS/MoNA/MS-Dial-VS17 between 55 and 81
- FragHub-unified library should contain 602,744 spectra for 32,193 unique chemical identifiers after integration of the four OMSLs (794,985 spectra → 602,744)
- The aggregated CSV must contain one row per library with the number of annotated features for the same 435-feature list
- Spectra removed by FragHub should be those lacking SMILES, InChI, precursor m/z, or adduct information

## Limitations

- Collision energy harmonization was not addressed due to their varied and nonstandardized measures (around 70 different formats), which can affect library matching
- OMSL compound coverage (32,193) is low relative to the estimated several million natural products, limiting annotation breadth
- Redundancy across OMSLs may lead to inconsistencies with dot product and reverse dot product scoring, which are highly sensitive to fragment number and intensity
- License changed from MIT to CC-BY-NC 4.0, imposing non-commercial restrictions on the FragHub library

## Evidence

- [methods] Run MS/MS-based spectral annotation of the aligned feature list using each FragHub-processed OMSL exported in .MSP format with default MS-Dial parameters: [evidence span withheld — anchor status: not_found]
