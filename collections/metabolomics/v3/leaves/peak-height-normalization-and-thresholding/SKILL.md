---
name: peak-height-normalization-and-thresholding
description: Use when raw MS/MS peak lists must be cleaned and scaled before formula
  ranking — converting m/z to monoisotopic Da via adduct correction, normalizing all
  peak heights to 100, and discarding peaks whose normalized height is below 0.5.
when_to_use_negative:
- The goal is library or in-silico spectrum matching (e.g. CFM-ID, SIRIUS) rather
  than FSA formula-subset ranking, which expects this specific normalization scheme
- Input is already a normalized feature table or peak list with heights on a 0–100
  scale and low-intensity peaks removed
- The spectrum contains elements beyond C, H, O, N, S, P — FSA has not been validated
  for such compounds or for diverse adduct types
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: ProteoWizard
  role: msconvert is used to remove background noise by centroiding and deisotoping
    of raw MS/MS spectra before peak-height normalization
- name: FSA
  role: Pipeline whose Eq. (1) adduct-corrected mass conversion and 0–100 intensity
    normalization with a 0.5 cutoff this skill implements; the cleaned peak list feeds
    FSA formula ranking
  repo: kslynn128171/FSA
- name: ms-adduct-calculator
  role: Provides common adduct masses needed to convert peak m/z to monoisotopic mass
    via the paper's Eq. (1) prior to normalization
  repo: https://fiehnlab.ucdavis.edu/staff/kind/metabolomics/ms-adduct-calculator/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:148f01133e4a3bd5d1254aaf4187cbaabddd7f8b9c52de07d896fb77558b3de6
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
  - ms-adduct-calculator
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# peak-height-normalization-and-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A spectral preprocessing routine from the FSA metabolite-identification pipeline: after adduct-corrected conversion of m/z to monoisotopic mass, peak heights are normalized to a 0–100 scale (base peak = 100) and low-intensity peaks are discarded below a 0.5 threshold before formula matching.

## When to use

Use when raw MS/MS peak lists must be cleaned and scaled before formula ranking — converting m/z to monoisotopic Da via adduct correction, normalizing all peak heights to 100, and discarding peaks whose normalized height is below 0.5.

## When NOT to use

- The goal is library or in-silico spectrum matching (e.g. CFM-ID, SIRIUS) rather than FSA formula-subset ranking, which expects this specific normalization scheme
- Input is already a normalized feature table or peak list with heights on a 0–100 scale and low-intensity peaks removed
- The spectrum contains elements beyond C, H, O, N, S, P — FSA has not been validated for such compounds or for diverse adduct types

## Inputs

- Raw MS/MS spectrum (centroided/deisotoped peak list with precursor m/z and adduct type)
- Adduct correction specification (e.g. FiehnLab ms-adduct-calculator adduct masses) for Eq. (1) conversion

## Outputs

- preprocessed_peaks.csv — cleaned, normalized peak list per spectrum (fragment masses in monoisotopic Da, heights 0–100, artificial precursor peak present)
- Validation-ready peak table where no fragment mass exceeds the precursor mass

## How to apply

Load the raw MS/MS spectrum and remove background noise by centroiding/deisotoping, e.g. with msconvert in ProteoWizard. Convert each peak's m/z to a monoisotopic Da value using the FSA paper's Eq. (1) with adduct correction (FiehnLab ms-adduct-calculator masses), then drop fragment peaks heavier than the precursor or within [0, −2] Da of it. Add an artificial precursor peak to the peak list with the height of the base peak, then normalize all peak heights so the tallest peak is 100. Finally, discard peaks whose normalized height is below 0.5 and write the cleaned peak list per spectrum to preprocessed_peaks.csv. The rationale is that FSA matches fragments against formula candidates by mass and relative intensity, so peaks must share a common intensity scale and spurious precursor-region peaks must not contaminate matching.

## Related tools

- **ProteoWizard** (msconvert is used to remove background noise by centroiding and deisotoping of raw MS/MS spectra before peak-height normalization)
- **FSA** (Pipeline whose Eq. (1) adduct-corrected mass conversion and 0–100 intensity normalization with a 0.5 cutoff this skill implements; the cleaned peak list feeds FSA formula ranking) — kslynn128171/FSA
- **ms-adduct-calculator** (Provides common adduct masses needed to convert peak m/z to monoisotopic mass via the paper's Eq. (1) prior to normalization) — https://fiehnlab.ucdavis.edu/staff/kind/metabolomics/ms-adduct-calculator/

## Evaluation signals

- The output peak list contains the artificial precursor peak, added with the height of the base peak
- All peak heights are normalized to a 0–100 scale and no peak in the output has a normalized height below 0.5
- No fragment mass in the cleaned peak list exceeds the precursor mass, and no peak lies within [0, −2] Da of the precursor
- Fragment masses are monoisotopic values produced by the adduct-corrected Eq. (1) conversion, not raw m/z

## Limitations

- Few chemical rules have been implemented in FSA to rank possible formulas and filter out unfeasible ones, and species/adduct-specific rules are not incorporated, which affects downstream ranking after this preprocessing
- FSA has so far been validated only on compounds composed of C, H, O, N, S, and P, so normalized spectra of more complex molecules are out of scope
- Normalization discards everything below 0.5 on the 0–100 scale, so very low-intensity but chemically informative fragments are irreversibly removed before matching

## Evidence

- [methods] Load the raw MS/MS spectrum and remove background noise by centroiding/deisotoping, e.g. with msconvert in ProteoWizard or an equivalent tool.: [evidence span withheld — anchor status: not_found]
- [methods] Normalize all peak heights so the tallest peak is 100. Discard peaks whose normalized height is below 0.5.: [evidence span withheld — anchor status: not_found]
- [results] The FSA spectral preprocessing pipeline converts m/z values to monoisotopic masses via the adduct-corrected Eq. (1), removes peaks heavier than the precursor or within [0, −2] Da of it: [evidence span withheld — anchor status: not_found]
