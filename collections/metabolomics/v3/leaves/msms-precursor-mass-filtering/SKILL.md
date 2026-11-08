---
name: msms-precursor-mass-filtering
description: Use when cleaning raw MS/MS spectra prior to formula identification —
  e.g., converting m/z to monoisotopic Da via adduct correction and removing spurious
  precursor-adjacent and contamination peaks.
when_to_use_negative:
- The spectrum is already a cleaned, formula-ready peak list (no precursor-adjacent
  peaks, already normalized) — re-filtering would distort relative intensities
- The goal is peak picking on profile-mode (non-centroided) data alone; centroiding/deisotoping
  is a prerequisite handled by tools like msconvert in ProteoWizard
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ProteoWizard
  role: msconvert is used to centroid and deisotope the raw MS/MS spectrum before
    mass filtering
- name: FSA
  role: Filtering is executed inside the FSA preprocessing pipeline before formula
    matching
  repo: https://github.com/kslynn128171/FSA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:c554799de326250bea988553148807c9ceaf6ce834e1f2ce45c9f45c3a541434
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/FSA
  repo_url: https://github.com/kslynn128171/FSA
  tools:
  - FSA
  - ProteoWizard
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# msms-precursor-mass-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocessing step of the FSA LC-MS/MS pipeline: after centroiding/deisotoping, fragment peaks heavier than the precursor or lying within [0, −2] Da of it are removed, adduct-corrected monoisotopic masses are computed, and peaks are normalized to a 0–100 scale before formula matching.

## When to use

Use when cleaning raw MS/MS spectra prior to formula identification — e.g., converting m/z to monoisotopic Da via adduct correction and removing spurious precursor-adjacent and contamination peaks.

## When NOT to use

- The spectrum is already a cleaned, formula-ready peak list (no precursor-adjacent peaks, already normalized) — re-filtering would distort relative intensities
- The goal is peak picking on profile-mode (non-centroided) data alone; centroiding/deisotoping is a prerequisite handled by tools like msconvert in ProteoWizard

## Inputs

- Raw MS/MS spectrum (centroided/deisotoped peak list)
- Precursor m/z and adduct assignment for the spectrum

## Outputs

- Cleaned peak list per spectrum (e.g., preprocessed_peaks.csv) with monoisotopic fragment masses, artificial precursor peak, and normalized 0–100 heights
- Spectra filtered below 0.5 normalized peak height

## How to apply

Load the raw MS/MS spectrum and remove background noise by centroiding/deisotoping with msconvert in ProteoWizard or an equivalent tool. Convert each peak's m/z to its monoisotopic Da using the paper's Eq. (1) with adduct correction (masses obtainable from the FiehnLab ms-adduct-calculator). Drop fragment peaks heavier than the precursor or within [0, −2] Da of the precursor mass, then add back an artificial precursor peak whose height equals the base peak. Normalize all peak heights so the tallest peak is 100 and discard peaks below a normalized height of 0.5. Write the cleaned peak list (e.g., preprocessed_peaks.csv) for downstream FSA formula matching.

## Related tools

- **ProteoWizard** (msconvert is used to centroid and deisotope the raw MS/MS spectrum before mass filtering)
- **FSA** (Filtering is executed inside the FSA preprocessing pipeline before formula matching) — https://github.com/kslynn128171/FSA

## Examples

```
python scripts/preprocess_fsa.py --input raw.mzML --adduct [M+H]+ --min-height 0.5 --output preprocessed_peaks.csv
```

## Evaluation signals

- The output peak list contains the artificial precursor peak
- All peak heights are normalized to the 0–100 scale with no peak below 0.5
- No fragment mass exceeds the precursor mass
- No retained fragment lies within [0, −2] Da of the precursor mass

## Limitations

- Filtering relies on a correct adduct assignment for the Eq. (1) mass conversion; wrong adducts shift monoisotopic masses
- FSA has been validated only on compounds composed of C, H, O, N, S, and P; applicability to other elements/adduct types is unverified
- Spectrum variation from instruments, environment, and sample preparation can alter fragment patterns even after filtering

## Evidence

- [methods] Drop fragment peaks that are heavier than the precursor or lie within [0, -2] Da of the precursor mass.: [evidence span withheld — anchor status: not_found]
- [results] Normalize all peak heights so the tallest peak is 100 and discard peaks below a normalized height of 0.5.: [evidence span withheld — anchor status: not_found]
- [readme] # FSA
Metabolite identification using fragment subset analysis: "Metabolite identification using fragment subset analysis"
