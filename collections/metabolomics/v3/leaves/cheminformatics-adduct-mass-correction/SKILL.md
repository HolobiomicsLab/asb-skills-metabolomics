---
name: cheminformatics-adduct-mass-correction
description: Use when raw MS/MS peak lists contain m/z values of adducted ions (e.g.
  [M+H]+, [M-H]-) that must be converted to monoisotopic neutral Da before fragment
  formula matching or precursor mass comparison.
when_to_use_negative:
- Input peaks are already monoisotopic neutral masses in Da, not m/z of adducted ions
  — a second correction would shift masses.
- The ionization adduct type is unknown or the spectrum contains mixed/unknown adducts,
  since Eq. (1) requires a specific adduct assumption.
- Compounds composed of elements beyond C, H, O, N, S, and P, which FSA has not been
  validated on.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: ProteoWizard
  role: msconvert is used to centroid/deisotope raw MS/MS spectra and remove background
    noise before adduct-corrected m/z conversion
- name: FSA
  role: Provides Eq. (1), the adduct-corrected m/z-to-monoisotopic-mass conversion
    formula applied here
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:799bab6a1e62872aa3261ec7ffa649eb039b949b4df1ab50f8af6b49ace109e0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/FSA
  tools:
  - FSA
  - ProteoWizard
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# cheminformatics-adduct-mass-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts m/z peak values to monoisotopic neutral masses via the adduct-corrected Eq. (1) from the FSA pipeline, a prerequisite before formula matching in LC-MS/MS metabolomics. Uses the FiehnLab ms-adduct-calculator adduct masses and cleans spectra (centroiding/deisotoping) beforehand.

## When to use

Use when raw MS/MS peak lists contain m/z values of adducted ions (e.g. [M+H]+, [M-H]-) that must be converted to monoisotopic neutral Da before fragment formula matching or precursor mass comparison.

## When NOT to use

- Input peaks are already monoisotopic neutral masses in Da, not m/z of adducted ions — a second correction would shift masses.
- The ionization adduct type is unknown or the spectrum contains mixed/unknown adducts, since Eq. (1) requires a specific adduct assumption.
- Compounds composed of elements beyond C, H, O, N, S, and P, which FSA has not been validated on.

## Inputs

- Raw MS/MS spectrum (centroided/deisotoped peak list, e.g. from msconvert in ProteoWizard)
- Precursor m/z and its adduct type
- FiehnLab ms-adduct-calculator adduct masses

## Outputs

- Per-spectrum peak list with monoisotopic fragment masses in Da (preprocessed_peaks.csv)
- Adduct-corrected monoisotopic precursor mass

## How to apply

First centroid and deisotope the raw spectrum to remove background noise, e.g. with msconvert in ProteoWizard. Then apply Eq. (1) from the FSA paper, using adduct masses from the FiehnLab ms-adduct-calculator, to convert each peak's m/z to its monoisotopic Da, correcting for the ionization adduct observed in the precursor. Consistency matters: the precursor mass and every fragment mass must be corrected with the same adduct assumption so downstream filters (dropping fragments heavier than the precursor or within [0, −2] Da of it) operate on comparable neutral masses. The corrected peak list then feeds formula subset analysis for candidate formula ranking. Validation: corrected masses should be at or below the precursor mass after the fragment filter is applied, with an artificial precursor peak present and heights normalized to 0–100.

## Related tools

- **ProteoWizard** (msconvert is used to centroid/deisotope raw MS/MS spectra and remove background noise before adduct-corrected m/z conversion)
- **FSA** (Provides Eq. (1), the adduct-corrected m/z-to-monoisotopic-mass conversion formula applied here)

## Examples

```
msconvert raw.mzML --mzCentroiding --filter "peakPicking" -o centroided.mzML  # then convert peaks via Eq. (1) with ms-adduct-calculator adduct masses
```

## Evaluation signals

- No fragment mass in the corrected peak list exceeds the monoisotopic precursor mass (post-filter invariant).
- The peak list contains the artificial precursor peak added with the height of the base peak.
- All peak heights are normalized to a 0–100 scale with no peak below 0.5.
- Corrected fragment masses fall consistently below the precursor mass, consistent with the [0, −2] Da precursor-adjacent window removal.
- Corrected output peak list is written per spectrum to preprocessed_peaks.csv.

## Limitations

- FSA has so far been validated on datasets containing compounds composed of C, H, O, N, S, and P; applicability to more complex molecules and compounds forming various adduct types is unvalidated.
- Few chemical rules have been implemented to rank possible formulas and filter out unfeasible ones; species/adduct-specific rules are not yet incorporated, so adduct assumptions may leave formula ambiguity unresolved.
- Fragment patterns in real mixtures can differ from those of a single compound, leading to differences between predicted and actual spectra even after correct mass correction.

## Evidence

- [methods] adduct-corrected mass conversion: [evidence span withheld — anchor status: not_found]
- [methods] preprocessing thresholds: [evidence span withheld — anchor status: not_found]
- [intro] centroiding tool: "free software such as msconvert in ProteoWizard (Chambers et al., 2012)"
- [intro] adduct mass source: "A useful calculator for common adducts can be found at https://fiehnlab.ucdavis.edu/staff/kind/metabolomics/ms-adduct-calculator/"
- [discussion] element scope: "FSA has so far been validated on datasets containing compounds composed of C, H, O, N, S, and P."
