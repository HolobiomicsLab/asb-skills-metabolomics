---
name: monoisotopic-mass-conversion-with-adduct-correction
description: 'Use when raw MS/MS spectra (e.g. centroided with msconvert) must be
  cleaned before formula ranking: m/z peaks need adduct-corrected monoisotopic mass
  conversion and spurious fragment removal prior to feeding FSA.'
when_to_use_negative:
- The spectrum is already a cleaned, formula-ready peak list with normalized heights
  and no precursor-inconsistent peaks — re-running would distort it
- The data are MS1-only profiles with no MS/MS precursor mass available, so precursor-adjacent
  filtering and Eq. (1) conversion cannot be applied
- The goal is compound identification from structural in-silico spectra (e.g. CFM-ID);
  this skill only prepares peaks for formula ranking, not structural elucidation
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ProteoWizard
  role: msconvert is used to centroid and deisotope raw MS/MS spectra, removing background
    noise before mass conversion and filtering
- name: FSA
  role: Downstream formula-ranking tool whose Eq. (1) defines the adduct-corrected
    m/z-to-monoisotopic-mass conversion consumed by this preprocessing skill
  repo: https://github.com/kslynn128171/FSA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:a0aa8bc714fa5e071d68ddd87581397bbe35e3f72b7e8d7bf79fdda0ba761d4b
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# monoisotopic-mass-conversion-with-adduct-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts MS/MS peak m/z values to monoisotopic neutral masses via FSA's adduct-corrected Eq. (1), then filters precursor-inconsistent peaks (heavier than the precursor or within [0, -2] Da), adds an artificial precursor peak, normalizes heights to 100, and drops peaks below 0.5 so spectra are ready for formula matching.

## When to use

Use when raw MS/MS spectra (e.g. centroided with msconvert) must be cleaned before formula ranking: m/z peaks need adduct-corrected monoisotopic mass conversion and spurious fragment removal prior to feeding FSA.

## When NOT to use

- The spectrum is already a cleaned, formula-ready peak list with normalized heights and no precursor-inconsistent peaks — re-running would distort it
- The data are MS1-only profiles with no MS/MS precursor mass available, so precursor-adjacent filtering and Eq. (1) conversion cannot be applied
- The goal is compound identification from structural in-silico spectra (e.g. CFM-ID); this skill only prepares peaks for formula ranking, not structural elucidation

## Inputs

- Raw MS/MS spectrum (centroided/deisotoped peak list) with precursor m/z and adduct annotation
- Adduct definition (e.g. [M+H]+, [M+Na]+) matched to FiehnLab ms-adduct-calculator masses

## Outputs

- preprocessed_peaks.csv — cleaned peak list with adduct-corrected monoisotopic fragment masses, artificial precursor peak, and heights normalized to a 0-100 scale

## How to apply

First centroid/deisotope the raw MS/MS spectrum with ProteoWizard msconvert or an equivalent tool to remove background noise. Apply the paper's Eq. (1) to convert each peak m/z to a monoisotopic Da mass, using adduct masses from the FiehnLab ms-adduct-calculator. Drop fragment peaks heavier than the precursor or lying within [0, -2] Da of the precursor mass, since these cannot be genuine fragments; then append an artificial precursor peak whose height equals the base peak. Normalize all peak heights so the tallest is 100, discard peaks with normalized height below 0.5, and write the cleaned peak list per spectrum to preprocessed_peaks.csv. The rationale is that FSA formula matching assumes fragment masses are neutral monoisotopic masses consistent with the precursor, so uncorrected m/z and noise peaks would corrupt candidate formula generation.

## Related tools

- **ProteoWizard** (msconvert is used to centroid and deisotope raw MS/MS spectra, removing background noise before mass conversion and filtering)
- **FSA** (Downstream formula-ranking tool whose Eq. (1) defines the adduct-corrected m/z-to-monoisotopic-mass conversion consumed by this preprocessing skill) — https://github.com/kslynn128171/FSA

## Examples

```
msconvert raw.mzML --mzcentroid --filter "deisotope" && python fsa_preprocess.py --peaks peaks.tsv --adduct [M+H]+ --precursor-ppm 5 --output preprocessed_peaks.csv
```

## Evaluation signals

- preprocessed_peaks.csv contains an artificial precursor peak with the height of the base peak
- All peak heights are on a 0-100 normalized scale with no peak below 0.5
- No fragment mass in the output exceeds the precursor mass, and no peak lies within [0, -2] Da of the precursor
- Every peak mass equals the m/z converted through Eq. (1) with the correct adduct correction (verifiable against FiehnLab ms-adduct-calculator masses)

## Limitations

- FSA has only been validated on compounds composed of C, H, O, N, S, and P; adduct-corrected conversion for other element compositions or unusual adduct types is untested
- Removing peaks within [0, -2] Da of the precursor may discard genuine fragment peaks close to the precursor mass
- Normalization cutoff of 0.5 (of a 0-100 scale) is very permissive; noisy low-intensity peaks just above 0.5 survive into formula matching
- FSA does not yet utilize structural information, so this preprocessing only supports formula ranking accuracy comparable to prescreening, not state-of-the-art identification

## Evidence

- [methods] The FSA spectral preprocessing pipeline converts m/z values to monoisotopic masses via the adduct-corrected Eq. (1), removes peaks heavier than the precursor or within [0, -2] Da of it: [evidence span withheld — anchor status: not_found]
