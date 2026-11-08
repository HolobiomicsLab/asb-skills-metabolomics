---
name: isotopic-trace-peak-shape-filtering
description: Use when filtering raw FeatureFinderMetabo output from centroided ddMS2
  mzML data to remove spurious or poorly shaped features before blank correction and
  PFAS prioritization.
when_to_use_negative:
- Input is already a curated/custom feature list (external_feature_list.xlsx) — PFΔScreen's
  FeatureFinding and this shape filter are bypassed in that path
- Data are profile-mode or non-centroided spectra; the README specifies centroided
  ddMS2 mzML input
- Analytes lack the isotopic structure required for trace modeling (e.g., very low
  intensity peaks yielding fewer than two isotopic traces)
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pyOpenMS
  role: Runs FeatureFinderMetabo feature detection, which applies the metabolites
    (5% RMS) isotope model and yields FWHM and isotopic trace annotations used by
    this filter
  repo: https://github.com/JonZwe/PFAScreen
- name: OpenMS
  role: Supplies the metabolites (5% RMS) isotope model used to exclude features with
    unusual peak shapes of isotopic traces
- name: PFΔScreen
  role: Implements the FeatureFinding tab where the FWHM (1 s–1 min) and ≥2 isotopic
    trace constraints are applied
  repo: https://github.com/JonZwe/PFAScreen
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:3d80d3ddfe9412ffbbe1157b6b8110c9c02b9621003fb3945377d2e31bc8aaea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JonZwe/PFAScreen
  tools:
  - OpenMS
  - PFΔScreen
  - Python
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Isotopic trace peak shape filtering

## Summary

Post-feature-detection quality filter in PFΔScreen's FeatureFinding tab that retains only features with a full width at half maximum (FWHM) above 1 s and below 1 min and at least two isotopic traces, excluding features with unusual isotopic peak shapes using the OpenMS metabolites (5% RMS) isotope model.

## When to use

Use when filtering raw FeatureFinderMetabo output from centroided ddMS2 mzML data to remove spurious or poorly shaped features before blank correction and PFAS prioritization.

## When NOT to use

- Input is already a curated/custom feature list (external_feature_list.xlsx) — PFΔScreen's FeatureFinding and this shape filter are bypassed in that path
- Data are profile-mode or non-centroided spectra; the README specifies centroided ddMS2 mzML input
- Analytes lack the isotopic structure required for trace modeling (e.g., very low intensity peaks yielding fewer than two isotopic traces)

## Inputs

- Centroided ddMS2 mzML sample file (and optional extraction blank mzML) with MS1 and data-dependent MS2 spectra
- Raw FeatureFinderMetabo feature list with per-feature m/z, RT, intensity, FWHM, and isotopic trace annotations

## Outputs

- Filtered feature list retaining only features with FWHM > 1 s and < 1 min and ≥ 2 isotopic traces
- Cleaned feature table (m/z, RT, intensity) ready for MS2 assignment and blank correction

## How to apply

After running pyOpenMS FeatureFinderMetabo on the centroided ddMS2 mzML sample file (mass error 10 ppm, MS1 intensity threshold 2000 counts), apply the isotope/peak-shape constraints: keep only features with FWHM greater than 1 s and less than 1 min and at least two isotopic traces. Feature detection itself uses the metabolites (5% RMS) isotope model from OpenMS to exclude features with unusual peak shapes of isotopic traces. This filter reduces the raw feature list prior to MS2-to-precursor assignment and blank correction (which later removes sample/blank features within 2 mDa and RT difference < 0.1 min unless ≥5-fold more abundant in the sample). Correctness can be judged by checking that retained features carry isotopic trace annotations and FWHM values in the accepted window, and that final feature counts are on the reported scale (e.g., 4209 features in soil M1 ESI− before blank correction, 3750 after).

## Related tools

- **pyOpenMS** (Runs FeatureFinderMetabo feature detection, which applies the metabolites (5% RMS) isotope model and yields FWHM and isotopic trace annotations used by this filter) — https://github.com/JonZwe/PFAScreen
- **OpenMS** (Supplies the metabolites (5% RMS) isotope model used to exclude features with unusual peak shapes of isotopic traces)
- **PFΔScreen** (Implements the FeatureFinding tab where the FWHM (1 s–1 min) and ≥2 isotopic trace constraints are applied) — https://github.com/JonZwe/PFAScreen

## Examples

```
from pyOpenMS import FeatureFinderMetabo; ff = FeatureFinderMetabo(); p = ff.getDefaults(); p.setValue('metabolites_isotope_model', 'metabolites (5% RMS)'); ff.setParameters(p); ff.run('sample.mzML')
```

## Evaluation signals

- Every retained feature has FWHM strictly > 1 s and < 1 min (60 s)
- Every retained feature has at least two annotated isotopic traces
- Feature counts after filtering are on the reported scale (e.g., 4209 features detected in soil M1 ESI−, reduced to 3750 after subsequent blank correction)
- Output feature_list.csv contains the required m/z, RT, intensity, and MS2 columns

## Limitations

- Thresholds (FWHM 1 s–1 min, ≥2 isotopic traces) are fixed in the article's workflow; PFΔScreen offers no default/recommended parameter values across all contexts, so users must adjust parameters per sample and instrument
- Features with only one isotopic trace (e.g., low-abundance or fluorine-dominated signals with limited isotope patterns) may be incorrectly removed

## Evidence

- [results] Peaks must have FWHM above 1 s and below 1 min, and at least two isotopic traces: "Peaks reported after feature detection have to have a full width at half maximum (FWHM) above 1 s and below 1 min, and at least two isotopic traces."
- [results] Metabolites isotope model from OpenMS: "the metabolites (5% RMS) isotope model from OpenMS was used to exclude features with unusual peak shapes of isotopic traces"
- [results] Feature detection parameters (10 ppm, 2000 counts): "The mass error for feature detection was set to 10 ppm, the MS1 intensity threshold was set to 2000 counts and the metabolites (5% RMS) isotope model from OpenMS was used"
- [results] Feature count scale check after blank correction: "Exemplified on soil M1, 4209 features were detected, which were reduced to 3750 features after blank correction in the ESI− mode."
- [readme] README: centroided ddMS2 mzML requirement: "Sample and blank for raw data input in PFΔScreen should have been measured under data-dependent acquisition (ddMS2) with centroided spectra"
