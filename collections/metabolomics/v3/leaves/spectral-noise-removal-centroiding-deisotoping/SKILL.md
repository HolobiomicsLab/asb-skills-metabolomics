---
name: spectral-noise-removal-centroiding-deisotoping
description: Use when raw MS/MS peak lists contain noise, isotopic, or precursor-adjacent
  peaks and must be cleaned into a normalized monoisotopic peak list before formula
  ranking or spectrum matching.
when_to_use_negative:
- The input is already a cleaned, normalized peak list satisfying FSA's constraints
  (artificial precursor peak present, heights in 0-100, no fragment above precursor
  mass) — re-running would add a duplicate precursor peak.
- The goal is in silico spectrum prediction or database MS/MS matching (e.g. CFM-ID
  4.0, SIRIUS), which consumes spectra differently rather than requiring this formula-matching
  preprocessing.
tools:
- name: ProteoWizard
  role: msconvert is used to centroid and deisotope raw MS/MS spectra (remove background
    noise) before adduct-corrected mass conversion and intensity filtering
- name: FiehnLab ms-adduct-calculator
  role: supplies adduct masses used in the Eq. (1) conversion of peak m/z to monoisotopic
    molecular weight
  repo: https://fiehnlab.ucdavis.edu/staff/kind/metabolomics/ms-adduct-calculator/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:56b4386a01bb508189e2f5b4658e1a0fb4935e181023c3b897d281d00366236b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/FSA
  tools:
  - FiehnLab ms-adduct-calculator
  - ProteoWizard
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# spectral-noise-removal-centroiding-deisotoping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cleans raw LC-MS/MS spectra before FSA formula matching by centroiding/deisotoping (e.g. with msconvert in ProteoWizard), converting m/z to monoisotopic masses via adduct-corrected Eq. (1), removing precursor-adjacent and high-mass artifact peaks, and intensity-normalizing the peak list.

## When to use

Use when raw MS/MS peak lists contain noise, isotopic, or precursor-adjacent peaks and must be cleaned into a normalized monoisotopic peak list before formula ranking or spectrum matching.

## When NOT to use

- The input is already a cleaned, normalized peak list satisfying FSA's constraints (artificial precursor peak present, heights in 0-100, no fragment above precursor mass) — re-running would add a duplicate precursor peak.
- The goal is in silico spectrum prediction or database MS/MS matching (e.g. CFM-ID 4.0, SIRIUS), which consumes spectra differently rather than requiring this formula-matching preprocessing.

## Inputs

- raw MS/MS spectrum (centroided/deisotoped peak list, e.g. from msconvert)
- precursor m/z and adduct type for Eq. (1) monoisotopic mass conversion

## Outputs

- preprocessed_peaks.csv — cleaned peak list with artificial precursor peak, 0-100 normalized heights, no fragment mass exceeding the precursor

## How to apply

Load the raw MS/MS spectrum and remove background noise by centroiding/deisotoping with msconvert in ProteoWizard or an equivalent tool. Convert each peak's m/z to monoisotopic Da using the paper's Eq. (1) with adduct correction, drawing adduct masses from the FiehnLab ms-adduct-calculator. Drop fragment peaks heavier than the precursor or within [0, -2] Da of the precursor mass, then add back an artificial precursor peak whose height equals the base peak. Normalize all peak heights so the tallest peak is 100, and discard peaks whose normalized height falls below 0.5. Write the cleaned peaks per spectrum to preprocessed_peaks.csv; this list is the input format FSA expects for formula matching.

## Related tools

- **ProteoWizard** (msconvert is used to centroid and deisotope raw MS/MS spectra (remove background noise) before adduct-corrected mass conversion and intensity filtering)
- **FiehnLab ms-adduct-calculator** (supplies adduct masses used in the Eq. (1) conversion of peak m/z to monoisotopic molecular weight) — https://fiehnlab.ucdavis.edu/staff/kind/metabolomics/ms-adduct-calculator/

## Examples

```
msconvert raw.mzML --filter "peakPicking true" --filter "deisotope" -o centroided.mzML
```

## Evaluation signals

- Output peak list contains the artificial precursor peak, whose height equals the base peak before normalization.
- All peak heights are normalized to a 0-100 scale with no peak below 0.5.
- No fragment mass in the output exceeds the precursor mass, and no fragment lies within [0, -2] Da below the precursor (except the re-added precursor peak).
- Peak masses are monoisotopic Da values produced by the Eq. (1) adduct correction, not raw m/z values.
- One cleaned peak list per input spectrum is written to preprocessed_peaks.csv.

## Limitations

- The [0, -2] Da precursor-adjacent exclusion window and the 0.5 height cutoff were tuned for the FSA pipeline; other formula-ranking tools may require different normalization schemes.
- Cleaning spectra does not compensate for genuine differences in fragment patterns caused by instrument variation, environmental conditions, sample preparation, or metabolite interactions in real mixtures.
- FSA (and thus its downstream formula matching on these cleaned spectra) has been validated mainly on compounds composed of C, H, O, N, S, and P, and with a slightly outdated PubChem snapshot (109,383,767 compounds).

## Evidence

- [other] converts m/z values to monoisotopic masses via the adduct-corrected Eq. (1), removes peaks heavier than the precursor or within [0, −2] Da of it: [evidence span withheld — anchor status: not_found]
