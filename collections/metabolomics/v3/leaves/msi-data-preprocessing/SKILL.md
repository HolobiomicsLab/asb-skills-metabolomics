---
name: msi-data-preprocessing
description: Use when raw .imzML/.ibd MSI pixel data (e.g. test_POS from MetaboLights
  MTBLS487) must be converted into a filtered, deisotoped ion feature table before
  lipid annotation at a chosen ppm.annotate tolerance.
when_to_use_negative:
- Input is already a deisotoped, filtered ion feature table — rerun only the annotate
  step (process=F) instead of the full preprocessing pipeline.
- Data are not in imzML format and cannot be converted, or the target analytes fall
  outside the lipid classes massPix searches (e.g. non-lipid metabolites).
- MS/MS-level lipid confirmation is required — accurate-mass matching in massPix is
  explicitly only the first step of metabolite identification.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: massPix
  role: R package whose functions mzextractor, peakpicker.bin, subsetImage, filter,
    deisotope and constructImage perform the ion extraction, binning, missing-value
    filtering, deisotoping and image reconstruction steps
  repo: https://github.com/hallz/massPix
- name: R
  role: runtime environment in which the massPix package and its preprocessing functions
    are executed
  repo: https://github.com/hallz/massPix
- name: imzMLConverter
  role: parses imzML pixel spectra into R so mzextractor can operate on them; must
    sit in the imzMLConverter/ subfolder of the massPix master folder
  repo: http://www.cs.bham.ac.uk/~ibs/imzMLConverter/#
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-017-1252-5
    title: 'massPix: an R package for annotation and interpretation of mass spectrometry
      imaging data for lipidomics'
  source_content_hash: sha256:dfaa972ef808f6f67eaff44ecff307f0c95952219f1a95b333b2e471d873d511
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hallz/massPix
  tools:
  - R
  - imzMLConverter
  - massPix
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# msi-data-preprocessing

## Summary

Preprocess mass spectrometry imaging (MSI) lipidomics data in imzML format with the massPix R package — ion extraction, peak binning, missing-value filtering, deisotoping and image construction — to produce a single deisotoped ion list suitable for accurate-mass lipid annotation.

## When to use

Use when raw .imzML/.ibd MSI pixel data (e.g. test_POS from MetaboLights MTBLS487) must be converted into a filtered, deisotoped ion feature table before lipid annotation at a chosen ppm.annotate tolerance.

## When NOT to use

- Input is already a deisotoped, filtered ion feature table — rerun only the annotate step (process=F) instead of the full preprocessing pipeline.
- Data are not in imzML format and cannot be converted, or the target analytes fall outside the lipid classes massPix searches (e.g. non-lipid metabolites).
- MS/MS-level lipid confirmation is required — accurate-mass matching in massPix is explicitly only the first step of metabolite identification.

## Inputs

- .imzML file (e.g. test_POS.imzML from MetaboLights MTBLS487)
- companion .ibd binary file kept together in the data/ folder
- imzMLConverter jar placed in the massPix master folder
- lipid library files in the libraries/ folder for makelibrary

## Outputs

- single deisotoped ion feature list (m/z values) after filtering and deisotoping
- image.norm.csv (full normalised image data matrix: lipid annotations, m/z and intensities)
- image.norm_short.csv (lipid annotations and m/z only)
- comparison table lipid_annotation_ppm_comparison.csv (tolerance, annotation count, difference)

## How to apply

Parse the .imzML file into R via the imzMLConverter jar, then run massPix preprocessing with ion extraction at thres.int=100000, thres.low=200, thres.high=1000 and bin.ppm=10 (bin width should match mass resolving power, e.g. 60,000 at m/z 400). Retain spectral features defined by the median m/z per bin with peakpicker.bin, and filter missing values with thres.filter=11 (steps run 1–21, where 11 means the ion must be present in at least half the pixels). Perform deisotoping on a random pixel subset (subsetImage) with ppm=3, no_isotopes=2, prop.1=0.9, prop.2=0.5 so that M+1/M+2 isotopes within those intensity proportions of the monoisotopic ion are removed, then rebuild the image across all pixels with constructImage. Because the resulting deisotoped ion list is the annotation input, keep it identical when re-annotating at different ppm.annotate values (e.g. 5 vs 50 ppm) to isolate the effect of mass tolerance. Annotate with makelibrary/annotate against lipid classes (DAG, TAG, PC, PE, PS, LysoPC, CE, SM, Cer) and adducts ([M+H]+, [M+Na]+, [M+K]+, [M+NH4]+) in positive mode, and record annotation counts to quantify tolerance sensitivity (~200 additional annotations at 50 ppm vs 5 ppm on the MTBLS487 test data).

## Related tools

- **massPix** (R package whose functions mzextractor, peakpicker.bin, subsetImage, filter, deisotope and constructImage perform the ion extraction, binning, missing-value filtering, deisotoping and image reconstruction steps) — https://github.com/hallz/massPix
- **R** (runtime environment in which the massPix package and its preprocessing functions are executed) — https://github.com/hallz/massPix
- **imzMLConverter** (parses imzML pixel spectra into R so mzextractor can operate on them; must sit in the imzMLConverter/ subfolder of the massPix master folder) — http://www.cs.bham.ac.uk/~ibs/imzMLConverter/#

## Examples

```
results <- massPix(process=T, ionisation_mode="positive", thres.int=100000, thres.low=200, thres.high=1000, bin.ppm=10, thres.filter=11, ppm.annotate=10, ppm=3, no_isotopes=2, prop.1=0.9, prop.2=0.5)
```

## Evaluation signals

- bin.ppm (10 ppm) matches the instrument resolving power guideline (60,000 at m/z 400) and equals the suggested ppm.annotate setting per the README
- thres.filter=11 falls within the recommended 11–15 range of the 1–21 missing-value steps (ion present in at least ~half of pixels)
- Deisotoped list contains no M+1/M+2 isotopes exceeding prop.1=0.9 / prop.2=0.5 of monoisotope intensity within ppm=3 tolerance
- Re-annotating the identical deisotoped ion list at ppm.annotate=50 vs 5 yields ~200 additional annotations, matching the paper's reported value
- Output files image.norm.csv and image.norm_short.csv are produced with columns for lipid annotations, m/z and intensities

## Limitations

- massPix processes one file at a time
- A database hit based on accurate mass is only the first step in metabolite identification; MS/MS confirmation is required
- The bin width (bin.ppm) must be adjusted to the instrument's mass resolving power; 10 ppm is only a starting suggestion
- The lipid class list (DAG, TAG, PC, PE, PS, LysoPC, CE, SM, Cer in positive mode) is not exhaustive, so annotations are putative and class-limited

## Evidence

- [methods] Ions with intensities greater than a threshold, from each spectra, are extracted and grouped to user-adjustable mass bins (function: mzextractor): "Ions with intensities greater than a threshold, from each spectra, are extracted and grouped to user-adjustable mass bins (function: mzextractor)"
- [methods] Deisotoping removes isotopes: "The deisotoping algorithm identifies the molecular ion (M) and removes isotopes at m/z (M+1) and (M+2) which are within a calculated proportion of the intensity of M"
- [readme] peakpicker.bin defines features: "Spectral features are defined by the median m/z value in each bin, and only features detected above a threshold proportion of spectra are retained (function: peakpicker.bin)"
- [readme] annotate ppm threshold definition: "ppm.annotate - Defines ppm threshold for which |observed m/z - theoretical m/z| must be less than for annotation to be retained."
- [readme] MS/MS confirmation caveat: "a database hit based on accurate mass should only be considered the first step in metabolite identification, and confirmation carried out using MS/MS is required"
