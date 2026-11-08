---
name: ppm-mass-tolerance-matching
description: Use when annotating deisotoped MSI ions by accurate mass in massPix and
  you need to choose or justify an annotation ppm tolerance, or reproduce the paper's
  5 ppm vs 50 ppm comparison (~200 extra annotations) on lipidomics imzML data.
when_to_use_negative:
- Input is raw spectra not yet preprocessed — ions must first be extracted, binned,
  filtered and deisotoped (mzextractor, peakpicker.bin, filter, deisotope) before
  tolerance-based matching
- You need confident metabolite identification rather than putative annotation — accurate-mass
  matching is only the first step and requires MS/MS confirmation
- Your goal is false-discovery-rate-controlled annotation; the article points to METASPACE
  as the framework for FDR-controlled metabolite annotation in MSI
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: massPix
  role: R package whose makelibrary and annotate functions perform the putative lipid
    annotation by accurate mass matching at a user-set ppm.annotate threshold
  repo: https://github.com/hallz/massPix
- name: R
  role: Runtime environment in which the massPix annotation workflow and the 5 ppm
    vs 50 ppm comparison are executed
  repo: https://github.com/hallz/massPix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-017-1252-5
    title: 'massPix: an R package for annotation and interpretation of mass spectrometry
      imaging data for lipidomics'
  source_content_hash: sha256:d370bfda53c86fdd042b2bb8c42472e28509a321d524ef1e904bf7f0ea6e13ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hallz/massPix
  tools:
  - R
  - massPix
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# ppm-mass-tolerance-matching

## Summary

Quantifies the effect of the accurate-mass matching tolerance (ppm.annotate) on putative lipid annotations in massPix, e.g. comparing 5 ppm vs 50 ppm on the MTBLS487 test_POS.imzML data where the looser tolerance yields roughly 200 additional annotations. The skill grounds annotation in deisotoped ion m/z searched against a generated lipid library of fatty acid, head-group and adduct combinations.

## When to use

Use when annotating deisotoped MSI ions by accurate mass in massPix and you need to choose or justify an annotation ppm tolerance, or reproduce the paper's 5 ppm vs 50 ppm comparison (~200 extra annotations) on lipidomics imzML data.

## When NOT to use

- Input is raw spectra not yet preprocessed — ions must first be extracted, binned, filtered and deisotoped (mzextractor, peakpicker.bin, filter, deisotope) before tolerance-based matching
- You need confident metabolite identification rather than putative annotation — accurate-mass matching is only the first step and requires MS/MS confirmation
- Your goal is false-discovery-rate-controlled annotation; the article points to METASPACE as the framework for FDR-controlled metabolite annotation in MSI

## Inputs

- Deisotoped ion list from massPix preprocessing of test_POS.imzML/test_POS.ibd (MetaboLights MTBLS487)
- Generated lipid mass library (makelibrary output) for the positive ionisation mode
- massPix R package installed from github.com/hallz/massPix with imzMLConverter.jar in place

## Outputs

- Number of putative lipid annotations at ppm.annotate=5
- Number of putative lipid annotations at ppm.annotate=50
- lipid_annotation_ppm_comparison.csv containing tolerance, annotation count and difference

## How to apply

Build a positive-ion-mode lipid library with makelibrary for classes DAG, TAG, PC, PE, PS, LysoPC, CE, SM, Cer and adducts [M+H]+, [M+Na]+, [M+K]+, [M+NH4]+, then annotate the single deisotoped ion list with the annotate function. Preprocess identically before annotation: mzextractor (thres.int=100000, thres.low=200, thres.high=1000, bin.ppm=10), peakpicker.bin, filter (thres.filter=11), and deisotope (ppm=3, no_isotopes=2, prop.1=0.9, prop.2=0.5) so that both tolerance runs see the same ion list. Run annotate once with ppm.annotate=5 and once with ppm.annotate=50, counting putative lipid annotations under each tolerance. The match criterion is adjustable 'according to different MS performance capabilities', so the chosen tolerance should reflect instrument mass accuracy; the README suggests using the same tolerance as the binning width (bin.ppm). Compute the count difference and write a comparison table (tolerance, annotation count, difference) to lipid_annotation_ppm_comparison.csv; validation expects approximately 200 additional annotations at 50 ppm versus 5 ppm, matching the paper's value for this test data.

## Related tools

- **massPix** (R package whose makelibrary and annotate functions perform the putative lipid annotation by accurate mass matching at a user-set ppm.annotate threshold) — https://github.com/hallz/massPix
- **R** (Runtime environment in which the massPix annotation workflow and the 5 ppm vs 50 ppm comparison are executed) — https://github.com/hallz/massPix

## Examples

```
results <- massPix(process=T, ionisation_mode="positive", thres.int=100000, thres.low=200, thres.high=1000, bin.ppm=10, thres.filter=11, ppm.annotate=50, ppm=3, no_isotopes=2, prop.1=0.9, prop.2=0.5)
```

## Evaluation signals

- Both tolerance runs use the identical deisotoped ion list (same mzextractor/peakpicker.bin/filter/deisotope parameters), so annotation counts differ only via ppm.annotate
- The count difference between 50 ppm and 5 ppm is approximately 200 additional annotations, matching the value reported in the paper for the same test data
- lipid_annotation_ppm_comparison.csv exists with the expected columns: tolerance, annotation count, difference
- Retained annotations all satisfy |observed m/z - theoretical m/z| below the chosen ppm threshold, per the README definition of ppm.annotate
- Library composition matches the positive-ion-mode scope (DAG, TAG, PC, PE, PS, LysoPC, CE, SM, Cer with [M+H]+, [M+Na]+, [M+K]+, [M+NH4]+ adducts)

## Limitations

- A database hit based on accurate mass should only be considered the first step in metabolite identification; confirmation by MS/MS is required where appropriate
- The match criteria can be adjusted according to different MS performance capabilities, so the ~200-annotation difference is specific to this instrument/data and tolerance pair, not a universal constant
- The README suggests setting ppm.annotate to the same tolerance as the binning width (bin.ppm); a tolerance far above the instrument's mass accuracy inflates putative annotations
- massPix currently processes one imzML file at a time and supports only imzML-format input

## Evidence

- [methods] Putative lipid annotation by accurate mass is achieved by searching deisotoped ions against a generated library of lipid m/z ratios: "Putative lipid annotation by accurate mass is achieved by searching deisotoped ions against a generated library of lipid m/z ratios"
- [readme] ppm.annotate - Defines ppm threshold for which |observed m/z - theoretical m/z| must be less than for annotation to be retained.: "ppm.annotate - Defines ppm threshold for which |observed m/z - theoretical m/z| must be less than for annotation to be retained."
- [readme] The criteria for a match can be adjusted according to different MS performance capabilities.: "The criteria for a match can be adjusted according to different MS performance capabilities."
- [readme] a database hit based on accurate mass should only be considered the first step in metabolite identification, and confirmation carried out using MS/MS is required: "a database hit based on accurate mass should only be considered the first step in metabolite identification, and confirmation carried out using MS/MS is required"
- [readme] Suggest to use same mass tolerance (ppm.annotate) as bin width (bin.ppm): "Suggest to use same mass tolerance (ppm.annotate) as bin width (bin.ppm)"
