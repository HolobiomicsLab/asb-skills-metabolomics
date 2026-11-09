---
name: feature-based-molecular-annotation
description: Use when you have an aligned MS-Dial feature list with MS/MS deconvolved
  spectra (e.g. 435 MS-CleanR-filtered PI-mode features) and need to annotate them
  against open mass spectral libraries.
when_to_use_negative:
- You only have raw spectra and no aligned, MS-CleanR-filtered feature list yet —
  feature detection and deconvolution must precede annotation.
- You want to integrate or standardize OMSL libraries rather than use them to annotate
  features — that is the FragHub integration workflow, not this annotation step.
- Your input spectra are GC-EI libraries — this benchmark used LC ionization-mode-split
  libraries and the counting semantics may not transfer.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: MS-Dial
  role: Feature detection, MS/MS deconvolution, alignment, and MS/MS-based spectral
    annotation of the aligned feature list against each library
- name: FragHub
  role: Produces the consolidated, harmonized OMSL exports (.MSP) used as the annotation
    reference libraries
  repo: github.com/eMetaboHUB/FragHub
- name: MS-CleanR
  role: Integrated blank/mass-defect filtering of MS-Dial features before annotation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:8568e00ca684f1b191f8b81f22cb322128962f23b7d57e429505940f141cc63a
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# feature-based-molecular-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates MS-Dial-detected LC–MS features against consolidated open mass spectral libraries prepared by FragHub, scoring MS/MS spectral matches per feature and reporting the fraction of the feature list (e.g. 102 of 435 A. thaliana PI-mode features) that receives a hit.

## When to use

Use when you have an aligned MS-Dial feature list with MS/MS deconvolved spectra (e.g. 435 MS-CleanR-filtered PI-mode features) and need to annotate them against open mass spectral libraries.

## When NOT to use

- You only have raw spectra and no aligned, MS-CleanR-filtered feature list yet — feature detection and deconvolution must precede annotation.
- You want to integrate or standardize OMSL libraries rather than use them to annotate features — that is the FragHub integration workflow, not this annotation step.
- Your input spectra are GC-EI libraries — this benchmark used LC ionization-mode-split libraries and the counting semantics may not transfer.

## Inputs

- Aligned MS-Dial feature list with MS/MS spectra from thermo RAW quality-control and blank files (Nicolle et al. Zenodo 10.5281/zenodo.8421008)
- Filtered feature list produced by integrated MS-CleanR with blank ratio 0.8, RSD 40%, relative mass defect 50–3500 ppm
- FragHub-processed open mass spectral library exports in .MSP format (e.g. FragHub-unified, MassBank, GNPS, MoNA, MS-Dial-VS17)

## Outputs

- Per-library annotated-feature counts for the filtered feature list
- Aggregated annotation result table (CSV) comparing the number of annotated features across libraries

## How to apply

First generate an aligned feature list with MS/MS deconvolution in MS-Dial (v5.23), importing the raw QC and blank files, then apply integrated MS-CleanR filtering (blank ratio 0.8, relative standard deviation 40%, relative mass defect 50–3500 ppm, removing incorrect masses and ghost peaks) to obtain the filtered feature list. Export each FragHub-processed open mass spectral library (.MSP) from the integrated set built from GNPS (.MGF), MoNA (.MSP/.SDF/.JSON) and MassBank, which yields 602,744 spectra for 32,193 unique chemicals. Run MS/MS-based spectral annotation of the aligned feature list with MS-Dial default parameters, once per library export (FragHub-unified, MassBank, GNPS, MoNA, MS-Dial-VS17), then aggregate per-library annotated-feature counts into a comparison table (CSV). Correct application is judged by reproducing the paper's per-library counts (FragHub = 102, MassBank = 41, GNPS/MoNA/MS-Dial-VS17 in the 55–81 range) on the 435-feature A. thaliana PI-mode benchmark.

## Related tools

- **MS-Dial** (Feature detection, MS/MS deconvolution, alignment, and MS/MS-based spectral annotation of the aligned feature list against each library)
- **FragHub** (Produces the consolidated, harmonized OMSL exports (.MSP) used as the annotation reference libraries) — github.com/eMetaboHUB/FragHub
- **MS-CleanR** (Integrated blank/mass-defect filtering of MS-Dial features before annotation)

## Examples

```
MS-Dial v5.23: open aligned MS-CleanR-filtered feature list; annotation tab: run MS/MS-based annotation against FragHub-unified_1.4.1.MSP with default MS-Dial parameters; repeat for MassBank, GNPS, MoNA, MS-Dial-VS17 exports and compare counts.
```

## Evaluation signals

- Per-library annotated-feature counts match the paper's values: FragHub = 102, MassBank = 41, GNPS/MoNA/MS-Dial-VS17 within the 55–81 range
- Annotated features equal 102 of the 435 MS-CleanR-filtered features (24%) under FragHub default filtering criteria
- Libraries were exported in .MSP format and annotation used MS-Dial default parameters consistently across all libraries
- The 435-feature input list results from blank ratio 0.8, RSD 40%, and relative mass defect 50–3500 ppm filters

## Limitations

- Collision energy harmonization was not addressed due to their varied and nonstandardized measures (~70 different formats), so scoring across libraries may be inconsistent
- Redundancy across OMSLs can lead to inconsistencies with dot-product and reverse dot-product scoring, which are sensitive to fragment number and intensity
- OMSL compound coverage (~32,193 unique identifiers) is low relative to the estimated millions of natural products
- MacOS and Linux support was removed in the 20_12_2024 FragHub release and the license changed from MIT to CC-BY-NC 4.0

## Evidence

- [methods] Run MS/MS-based spectral annotation of the aligned feature list using each FragHub-processed OMSL exported in .MSP format with default MS-Dial parameters.: [evidence span withheld — anchor status: not_found]
- [results] Fragment count result: [evidence span withheld — anchor status: not_found]
