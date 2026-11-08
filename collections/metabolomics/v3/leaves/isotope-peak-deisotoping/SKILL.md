---
name: isotope-peak-deisotoping
description: Use when pre-processing imzML mass spectrometry imaging data and you
  need to remove 13C isotope peaks (M+1, M+2) from a feature table before lipid annotation
  or downstream statistics.
when_to_use_negative:
- Input is already a deisotoped or centroided feature table with isotopes removed
- Data are low-resolution spectra where 3 ppm 13C isotope assignment tolerance cannot
  be reliably applied
- The analysis is being re-run with different PCA/clustering/slicing settings only
  (use process = F, which skips pre-processing including deisotoping)
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: massPix
  role: R package providing the subsetImage, filter and deisotope functions that perform
    isotope-peak deisotoping
  repo: https://github.com/hallz/massPix
- name: imzMLConverter
  role: Java interface used to parse imzML spectra into R as the upstream step before
    deisotoping
  repo: https://github.com/hallz/massPix
- name: R
  role: Language in which massPix and its deisotoping functions are implemented and
    run
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-017-1252-5
    title: 'massPix: an R package for annotation and interpretation of mass spectrometry
      imaging data for lipidomics'
  source_content_hash: sha256:c41f59d2ace0e4d456fc492de583e5c52d7260f459af1e64451dc8485d0898be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hallz/massPix
  tools:
  - R
  - imzMLConverter
  - massPix
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# isotope-peak-deisotoping

## Summary

massPix's deisotoping step identifies molecular ions (M) and removes M+1 and M+2 13C isotope peaks whose intensities fall within calculated proportions of the monoisotope intensity, using average feature intensities computed from a random subset of pixels. It is part of massPix's initial pre-processing of imzML mass spectrometry imaging data prior to lipid annotation.

## When to use

Use when pre-processing imzML mass spectrometry imaging data and you need to remove 13C isotope peaks (M+1, M+2) from a feature table before lipid annotation or downstream statistics.

## When NOT to use

- Input is already a deisotoped or centroided feature table with isotopes removed
- Data are low-resolution spectra where 3 ppm 13C isotope assignment tolerance cannot be reliably applied
- The analysis is being re-run with different PCA/clustering/slicing settings only (use process = F, which skips pre-processing including deisotoping)

## Inputs

- imzML mass spectrometry imaging file pair (.imzML and .ibd), e.g. test_POS from MetaboLights MTBLS487
- Filtered feature table (median m/z per 10 ppm bin, features present above thres.filter = 11 proportion of spectra)
- Average intensities per feature computed from a random 3% subset of pixels

## Outputs

- Deisotoped feature table with m/z values and subset-image average intensities (e.g. deisotoped_features.csv)
- Deisotoped ions carried forward to lipid annotation (makelibrary/annotate) and image construction (constructImage)

## How to apply

After parsing the imzML file into R via the imzMLConverter Java interface and binning ions across spectra (mzextractor with bin.ppm = 10, thres.int = 100000, m/z 200-1000) and filtering features (thres.filter = 11, steps = seq(0,1,0.05)), compute average intensities for all features from a random subset of pixels using subsetImage with percentage.deiso = 3. Then run the deisotope function with a 13C isotope assignment tolerance of ppm = 3, no_isotopes = 2, prop.1 = 0.9, and prop.2 = 0.5: the algorithm identifies the molecular ion (M) and removes isotopes at m/z (M+1) and (M+2) which are within a calculated proportion of the intensity of M. prop.1 is the proportion of monoisotope intensity the 1st isotope intensity must not exceed, and prop.2 the proportion for the 2nd isotope. The rationale is that isotope peaks are artefacts of 13C natural abundance rather than distinct lipids, so removing them prevents redundant single-ion images and false annotation matches.

## Related tools

- **massPix** (R package providing the subsetImage, filter and deisotope functions that perform isotope-peak deisotoping) — https://github.com/hallz/massPix
- **imzMLConverter** (Java interface used to parse imzML spectra into R as the upstream step before deisotoping) — https://github.com/hallz/massPix
- **R** (Language in which massPix and its deisotoping functions are implemented and run)

## Examples

```
results <- massPix(process=T, thres.int=100000, thres.low=200, thres.high=1000, bin.ppm=10, thres.filter=11, percentage.deiso=3, steps=seq(0,1,0.05), ppm=3, no_isotopes=2, prop.1=0.9, prop.2=0.5)
```

## Evaluation signals

- deisotoped_features.csv exists and contains m/z and intensity columns
- All retained feature m/z values fall within the 200-1000 threshold range (thres.low/thres.high)
- No M+1 or M+2 isotope peaks remain within the 3 ppm tolerance and prop.1 = 0.9 / prop.2 = 0.5 proportion limits
- Feature count is reduced relative to the pre-deisotoping filtered feature table
- Re-processing with identical parameters reproduces the same deisotoped feature set

## Limitations

- Average intensities used for deisotoping are computed from only a random subset of pixels (percentage.deiso = 3), not the full image
- Deisotoping considers at most two 13C isotopes (no_isotopes = 1 or 2)
- prop.1/prop.2 thresholds and ppm tolerance must be tuned; incorrect settings may retain isotopes or remove genuine low-intensity co-eluting peaks

## Evidence

- [methods] Deisotoping via subsetImage averaging and proportion rules: "Average intensities for all features from a random subset of pixels are computed and used to perform deisotoping (function: subsetImage)"
- [methods] Deisotoping algorithm definition: "The deisotoping algorithm identifies the molecular ion (M) and removes isotopes at m/z (M+1) and (M+2) which are within a calculated proportion of the intensity of M"
- [readme] Parameter semantics from README: "prop.1 - Proportion of monoisotope intensity the 1st isotope intensity must not exceed to assign as a 13C isotope"
- [readme] Isotope assignment tolerance: "ppm - Defines tolerance (ppm) for the assignment of 13C isotopes."
- [readme] Workflow context: deisotoping precedes annotation: "The overall data processing workflow consists of initial data pre-processing, filtering, image subsetting, deisotoping, annotation, normalisation, scaling, image “slicing” and multivariate statistics."
