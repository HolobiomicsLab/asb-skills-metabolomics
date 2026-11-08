---
name: mgf-spectrum-field-parsing
description: Use when input is an MGF tandem mass spectrometry file whose spectra
  must be parsed into FIDDLE's required 'precursor_mz' and 'precursor_type' fields
  before candidate formula generation.
when_to_use_negative:
- Input spectra are not in MGF format (e.g., raw vendor files or mzML without conversion)
  — convert to MGF first
- Spectra have fewer than five peaks or m/z above 1500, which FIDDLE excludes as unreliable
  or out of scope
- The task is de novo formula annotation from a single m/z value without MS/MS spectra,
  where msbuddy alone suffices
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: FIDDLE
  role: Consumes the parsed 'precursor_mz' and 'precursor_type' fields from the MGF
    file to run deep-learning formula prediction and breadth-first candidate refinement
  repo: https://github.com/JosieHong/FIDDLE
- name: msfiddle
  role: PyPI/CLI package for FIDDLE that reads spectra from MGF files for prediction
  repo: https://github.com/josiehong/msfiddle
- name: msbuddy
  role: Optional BUDDY-based tool whose candidate outputs can be incorporated alongside
    FIDDLE in file-based workflows
  repo: https://github.com/Philipbear/msbuddy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:7115a3c2bbe000b9da8de63d722224bb12299666edc853d2da949ec798f2d382
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/JosieHong/FIDDLE
  repo_url: https://github.com/JosieHong/FIDDLE
  tools:
  - FIDDLE
  - msbuddy
  - msfiddle
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# mgf-spectrum-field-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parse MGF spectral files to extract the 'precursor_mz' and 'precursor_type' fields (plus peak lists) required by FIDDLE's input format, supplying the precursor masses and adduct annotations that drive candidate formula refinement.

## When to use

Use when input is an MGF tandem mass spectrometry file whose spectra must be parsed into FIDDLE's required 'precursor_mz' and 'precursor_type' fields before candidate formula generation.

## When NOT to use

- Input spectra are not in MGF format (e.g., raw vendor files or mzML without conversion) — convert to MGF first
- Spectra have fewer than five peaks or m/z above 1500, which FIDDLE excludes as unreliable or out of scope
- The task is de novo formula annotation from a single m/z value without MS/MS spectra, where msbuddy alone suffices

## Inputs

- MGF file with per-spectrum fields 'precursor_mz', 'precursor_type', and MS/MS peak lists
- Instrument type annotation (Q-TOF or Orbitrap) determining the ppm tolerance downstream

## Outputs

- Per-spectrum records of precursor m/z, precursor/adduct type, and filtered peak lists ready for FIDDLE prediction
- Candidate formulas written to candidate_formulas.csv (downstream of parsing)

## How to apply

Read each spectrum entry from the input MGF file and extract the 'precursor_mz' and 'precursor_type' fields, which FIDDLE's input format requires, together with the MS/MS peak list. Retain only spectra with common precursor types (e.g., [M + H]+, [M + Na]+, [2M + H]+), m/z in (0, 1500], and at least five peaks, excluding peaks below 50 m/z. The parsed precursor m/z defines the target mass for the breadth-first refinement (Algorithm 1), which uses a 10 ppm tolerance for Q-TOF and 5 ppm for Orbitrap instruments. The parsed fields feed candidate formula generation and SENIOR-rule filtering confined to atoms C, H, O, N, F, S, Cl, P, B, I, Br, Na, K with a total atom count capped at 300. Correct parsing is essential: a misread precursor type (e.g., uncharged neutral molecules such as H2O or NH3 adducts, which FIDDLE simplifies by absorbing into the formula) yields candidate formulas inconsistent with the true neutral mass.

## Related tools

- **FIDDLE** (Consumes the parsed 'precursor_mz' and 'precursor_type' fields from the MGF file to run deep-learning formula prediction and breadth-first candidate refinement) — https://github.com/JosieHong/FIDDLE
- **msfiddle** (PyPI/CLI package for FIDDLE that reads spectra from MGF files for prediction) — https://github.com/josiehong/msfiddle
- **msbuddy** (Optional BUDDY-based tool whose candidate outputs can be incorporated alongside FIDDLE in file-based workflows) — https://github.com/Philipbear/msbuddy

## Examples

```
msfiddle --input spectra.mgf --output candidate_formulas.csv
```

## Evaluation signals

- Every parsed spectrum carries non-empty 'precursor_mz' and 'precursor_type' values matching FIDDLE's required input fields
- Retained precursor types are common adducts (e.g., [M + H]+, [M + Na]+, [2M + H]+); uncharged neutral molecules (H2O, NH3, CO2, CH2O2, CH4O2) are handled by FIDDLE's adduct-simplification scheme
- Parsed precursor m/z values fall within (0, 1500] and spectra have at least five peaks after excluding peaks below 50 m/z
- Downstream candidate formulas satisfy SENIOR rules and fall within 10 ppm (Q-TOF) or 5 ppm (Orbitrap) of the parsed precursor mass
- Parsed spectra produce top-5 candidate lists consistent with FIDDLE's 93.6% top-5 accuracy benchmark

## Limitations

- Mass spectra with fewer than five peaks are excluded due to potential unreliability
- Peaks below 50 m/z are excluded from scans, so low-mass fragment information is not parsed
- m/z values above 1500 are out of scope due to their rarity
- Only common precursor types are supported; FIDDLE does not predict adduct types, so mis-specified 'precursor_type' fields cannot be corrected by the pipeline
- FIDDLE's performance degrades on chimeric (co-eluting) spectra as spectral mixing increases

## Evidence

- [methods] MGF input format requires 'precursor_mz' and 'precursor_type' fields per spectrum: [evidence span withheld — anchor status: not_found]
- [readme] msfiddle supports MGF file workflows: [evidence span withheld — anchor status: external_doc]
