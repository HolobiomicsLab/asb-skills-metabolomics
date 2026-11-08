---
name: omsl-library-benchmarking
description: 'Decision support for choosing or validating an OMSL (GNPS, MassBank,
  MoNA, MS-Dial-VS17, FragHub-integrated) for MS/MS-based feature annotation: run
  each library in MS-Dial with default parameters on the same aligned feature list
  and compare per-library annotated-feature counts.'
when_to_use_negative:
- The input is already a fully annotated feature table — this skill benchmarks annotation
  of an unannotated aligned feature list, not post-hoc validation of prior annotations.
- You only have a single library and no comparison set — the skill requires running
  the same feature list against multiple OMSLs (or the FragHub-unified consolidation)
  to produce comparative counts.
- You need in-silico or GC-specific library benchmarking on spectra that FragHub has
  flagged as inconsistent (lacking SMILES, InChI, precursor m/z, or adduct information)
  — such spectra are removed during FragHub processing and cannot be annotated.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS-Dial
  role: Feature detection, alignment, MS/MS deconvolution, and the MS/MS-based spectral
    annotation engine against each exported OMSL library (default parameters)
- name: MS-CleanR
  role: Blank-ratio (0.8), incorrect-mass, ghost-peak, RSD (40%), and relative mass
    defect (50–3500 ppm) filtering that yields the benchmark feature list
- name: FragHub
  role: Integrates and harmonizes the OMSLs into a unified .MSP library that is then
    benchmarked against the standalone libraries
  repo: github.com/eMetaboHUB/FragHub
- name: MatchMS
  role: Python package referenced for spectral library processing in the FragHub workflow
- name: MSMetaEnhancer
  role: Python package added to MatchMS satellites for metadata enrichment during
    library integration
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:717401eed8459358f578fa505fdc89f812037fdfcb2e80f8f23c0593622fceb6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/eMetaboHUB/FragHub
  tools:
  - FragHub
  - MS-CleanR
  - MS-Dial
  - MSMetaEnhancer
  - MatchMS
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# omsl-library-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks the annotation performance of open mass spectral libraries (OMSLs) — standalone or FragHub-integrated — by running MS/MS-based spectral annotation of an aligned feature list in MS-Dial and counting annotated features per library, as done for the 435-feature A. thaliana PI-mode benchmark where the integrated FragHub library annotated 102 features (24%) versus 41 for MassBank and 55–81 for GNPS/MoNA/MS-Dial-VS17.

## When to use

Decision support for choosing or validating an OMSL (GNPS, MassBank, MoNA, MS-Dial-VS17, FragHub-integrated) for MS/MS-based feature annotation: run each library in MS-Dial with default parameters on the same aligned feature list and compare per-library annotated-feature counts.

## When NOT to use

- The input is already a fully annotated feature table — this skill benchmarks annotation of an unannotated aligned feature list, not post-hoc validation of prior annotations.
- You only have a single library and no comparison set — the skill requires running the same feature list against multiple OMSLs (or the FragHub-unified consolidation) to produce comparative counts.
- You need in-silico or GC-specific library benchmarking on spectra that FragHub has flagged as inconsistent (lacking SMILES, InChI, precursor m/z, or adduct information) — such spectra are removed during FragHub processing and cannot be annotated.

## Inputs

- Aligned MS-Dial feature list (.msp/.mgf-exported aligned peaks with MS/MS deconvolution) derived from Thermo RAW QC and blank files
- FragHub-processed OMSL spectral libraries in .MSP format (one per library: FragHub-unified, MassBank, GNPS, MoNA, MS-Dial-VS17)
- MS-Dial v5.23 aligned feature/MS/MS table

## Outputs

- Per-library annotated-feature counts (e.g. FragHub = 102/435 = 24%, MassBank = 41, GNPS = 55–81 range)
- Aggregated per-library annotation result table (CSV) comparing the number of annotated features across libraries

## How to apply

First build the feature list: import Thermo RAW QC and blank files (e.g. Nicolle et al., 10.5281/zenodo.8421008) into MS-Dial v5.23, deconvolve MS/MS, align features, then filter with integrated MS-CleanR using a blank ratio of 0.8, removing incorrect masses and ghost peaks, with relative standard deviation of 40% and relative mass defect between 50 and 3500 ppm — yielding e.g. the 435-feature PI-mode list. Next, export each OMSL as .MSP after FragHub processing (FragHub harmonizes .MSP/.MGF/.JSON/.CSV/.XML inputs, deduplicates by SPLASH key, and segregates by ionization mode and chromatography). Then run MS/MS-based spectral annotation of the same aligned feature list in MS-Dial with default parameters against each library in turn, recording per-library annotated-feature counts. Finally, aggregate the per-library counts into a single CSV annotation table and compare against expected values (FragHub = 102 of 435 features, MassBank = 41, GNPS/MoNA/MS-Dial-VS17 in the 55–81 range) to judge which library best supports dereplication.

## Related tools

- **MS-Dial** (Feature detection, alignment, MS/MS deconvolution, and the MS/MS-based spectral annotation engine against each exported OMSL library (default parameters))
- **MS-CleanR** (Blank-ratio (0.8), incorrect-mass, ghost-peak, RSD (40%), and relative mass defect (50–3500 ppm) filtering that yields the benchmark feature list)
- **FragHub** (Integrates and harmonizes the OMSLs into a unified .MSP library that is then benchmarked against the standalone libraries) — github.com/eMetaboHUB/FragHub
- **MatchMS** (Python package referenced for spectral library processing in the FragHub workflow)
- **MSMetaEnhancer** (Python package added to MatchMS satellites for metadata enrichment during library integration)

## Examples

```
msdial -i aligned_peaks.msdial -lib FragHub_integrated.msp -annotate; repeat with MassBank.msp, GNPS.mgf→msp, MoNA.msp, MS-Dial-VS17.msp; then aggregate per-library annotated-feature counts into CSV
```

## Evaluation signals

- Per-library annotated-feature counts must reproduce the paper's reported values: FragHub = 102 annotated features out of 435 (24%), MassBank = 41, and GNPS/MoNA/MS-Dial-VS17 each in the 55–81 range.
- The input feature list must be exactly the MS-CleanR-filtered 435-feature PI-mode set (blank ratio 0.8, RSD 40%, relative mass defect 50–3500 ppm) — any deviation changes the benchmark denominator.
- Each library must be run with identical MS-Dial default annotation parameters on the same aligned feature list; counts should differ only because of library content, not settings.
- The final CSV should contain one row (or column) per library benchmarked (FragHub, MassBank, GNPS, MoNA, MS-Dial-VS17) with integer annotated-feature counts that sum ≤ 435.
- Spectra removed by FragHub as inconsistent (no SMILES/InChI, precursor m/z, or adduct info) should be absent from the integrated library output and therefore cannot inflate its annotated count.

## Limitations

- Benchmark is restricted to the OMSLs processed (GNPS, MoNA, MassBank, MS-Dial-VS17, FragHub); other libraries are not covered.
- MS-Dial default annotation parameters are used, so scores/tolerances are not optimized per library — results may understate achievable annotation for specialized settings.
- The 435-feature list is from a single Thermo RAW A. thaliana PI-mode dataset; portability to other instruments, ion modes, or sample matrices is untested here.
- Redundancy across OMSLs can cause inconsistencies with dot-product and reverse-dot-product scoring, which are highly sensitive to fragment number and intensity.
- OMSL coverage (32,193 unique chemicals after FragHub integration) is low relative to the millions of estimated natural products, so low annotation rates may reflect library coverage rather than method failure.

## Evidence

- [methods] Using FragHub's default filtering criteria, the consolidated integrated library successfully annotated 102 features, corresponding to 24% of the total detected features.: [evidence span withheld — anchor status: not_found]
- [methods] Apply integrated MS-CleanR filtering with a blank ratio of 0.8, removing incorrect masses and ghost peaks, keeping a relative standard deviation of 40% and a relative mass defect between 50 and 3500 ppm.: [evidence span withheld — anchor status: not_found]
