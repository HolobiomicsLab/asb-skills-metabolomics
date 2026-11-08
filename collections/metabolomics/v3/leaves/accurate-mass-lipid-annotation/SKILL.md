---
name: accurate-mass-lipid-annotation
description: Use when deisotoped ion m/z values from imzML-based MSI lipidomics data
  need putative lipid annotations (e.g. DAG, TAG, PC, PE, PS, LysoPC, CE, SM, Cer
  with [M+H]+, [M+Na]+, [M+K]+, [M+NH4]+ adducts).
when_to_use_negative:
- Input is already a feature table with lipid annotations, or the goal is confident
  identification — an accurate-mass database hit is only the first step and requires
  MS/MS confirmation
- Data are not lipids or the lipid classes of interest fall outside the searched classes/adducts
  (e.g. classes not in the DAG/TAG/PC/PE/PS/LysoPC/CE/SM/Cer positive-mode or PC/PA/PE/PS/PG/PI/FFA
  negative-mode lists)
- The m/z list has not been deisotoped and filtered (peakpicker.bin, deisotope), since
  annotate expects deisotoped ions
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: massPix
  role: Provides makelibrary (computes lipid m/z for all fatty acid/head-group/adduct
    combinations) and annotate (accurate-mass matching of deisotoped ions with ppm.annotate
    threshold)
  repo: https://github.com/hallz/massPix
- name: R
  role: Runtime environment in which the massPix package functions (makelibrary, annotate)
    are called; results are written to CSV via read.csv/write.csv workflows
  repo: https://github.com/hallz/massPix
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_006
  source_papers:
  - doi: 10.1007/s11306-017-1252-5
    title: 'massPix: an R package for annotation and interpretation of mass spectrometry
      imaging data for lipidomics'
  source_content_hash: sha256:ecab9f5d0422f82e143be7a296187e2231d76a133661be5edfba08c81fab7c73
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hallz/massPix
  tools:
  - R
  - massPix
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# accurate-mass-lipid-annotation

## Summary

Assigns putative lipid identities to deisotoped ions from mass spectrometry imaging (MSI) data by accurate-mass matching against a lipid m/z library generated on the fly by massPix (makelibrary) from fatty acid, head-group and adduct combinations.

## When to use

Use when deisotoped ion m/z values from imzML-based MSI lipidomics data need putative lipid annotations (e.g. DAG, TAG, PC, PE, PS, LysoPC, CE, SM, Cer with [M+H]+, [M+Na]+, [M+K]+, [M+NH4]+ adducts).

## When NOT to use

- Input is already a feature table with lipid annotations, or the goal is confident identification — an accurate-mass database hit is only the first step and requires MS/MS confirmation
- Data are not lipids or the lipid classes of interest fall outside the searched classes/adducts (e.g. classes not in the DAG/TAG/PC/PE/PS/LysoPC/CE/SM/Cer positive-mode or PC/PA/PE/PS/PG/PI/FFA negative-mode lists)
- The m/z list has not been deisotoped and filtered (peakpicker.bin, deisotope), since annotate expects deisotoped ions

## Inputs

- Deisotoped ion list (m/z values) from massPix preprocessing (mzextractor with thres.int=100000, thres.low=200, thres.high=1000, bin.ppm=10; deisotope ppm=3, no_isotopes=2)
- .imzML/.ibd MSI data files (e.g. test_POS.imzML/test_POS.ibd from MetaboLights MTBLS487)
- massPix library files (lib_class.csv, lib_FA.csv, lib_element.csv) in the 'libraries' folder
- Ionisation mode string ('positive' or 'negative') and ppm.annotate tolerance parameter

## Outputs

- CSV of putative lipid annotations paired with m/z values (image.norm_short.csv containing lipid annotations and m/z only)
- Full normalised annotated image matrix image.norm.csv (lipid annotations, m/z and intensities)

## How to apply

Load the massPix library lookup files (lib_class.csv, lib_FA.csv, lib_element.csv) from the package's 'libraries' folder and run makelibrary for the chosen ionisation mode to compute theoretical m/z for all combinations of common fatty acids, lipid head-groups and anticipated adducts ([M+K]+, [M+H]+, [M+Na]+, [M+NH4]+ in positive mode; [M–H]-, [M+Cl]-, [M+OAc]- in negative mode). Then run annotate on the deisotoped ion list (produced via mzextractor, peakpicker.bin, subsetImage, filter and deisotope) with ppm.annotate set to a threshold matching the bin width (suggest 10 ppm, matching bin.ppm=10 for data with mass resolving power 60,000 at m/z 400). A match is retained only when |observed m/z − theoretical m/z| is below ppm.annotate; the criteria can be adjusted according to MS performance. Write results to a CSV pairing each retained m/z with its putative lipid annotation (image.norm_short.csv-style output). Treat each database hit as only the first step in metabolite identification, requiring MS/MS confirmation where appropriate.

## Related tools

- **massPix** (Provides makelibrary (computes lipid m/z for all fatty acid/head-group/adduct combinations) and annotate (accurate-mass matching of deisotoped ions with ppm.annotate threshold)) — https://github.com/hallz/massPix
- **R** (Runtime environment in which the massPix package functions (makelibrary, annotate) are called; results are written to CSV via read.csv/write.csv workflows) — https://github.com/hallz/massPix

## Examples

```
results <- massPix(process=T, ionisation_mode="positive", bin.ppm=10, thres.filter=11, ppm.annotate=10)
```

## Evaluation signals

- Every retained annotation satisfies |observed m/z − theoretical m/z| < ppm.annotate (suggested 10 ppm, matching bin.ppm bin width)
- Output CSV exists and contains m/z and lipid annotation columns (image.norm_short.csv-style)
- On the MTBLS487 test_POS data, widening tolerance from ppm.annotate=5 to 50 yields approximately 200 additional annotations, matching the paper's reported value
- Known cerebellum ions such as [PC(36:1)+K]+, [PC(38:6)+K]+ and [PC(40:6)+K]+ appear among the annotations in positive mode
- The number of library entries reflects all combinations of the fatty acids, head-groups and adducts for the selected ionisation_mode

## Limitations

- An accurate-mass database hit is only the first step in metabolite identification; confirmation by MS/MS is required where appropriate
- Annotation count is highly sensitive to the ppm.annotate threshold (e.g. ~200 additional annotations between 5 and 50 ppm on the test data), so tolerance must match MS performance
- Lipid class coverage is not exhaustive — limited to the predefined classes and adducts per ionisation mode

## Evidence

- [readme] Annotation achieved by accurate-mass library matching: "Putative lipid annotation by accurate mass is achieved by searching deisotoped ions against a generated library of lipid m/z ratios"
- [readme] ppm.annotate threshold definition: "ppm.annotate - Defines ppm threshold for which |observed m/z - theoretical m/z| must be less than for annotation to be retained."
- [readme] MS/MS confirmation caveat: "a database hit based on accurate mass should only be considered the first step in metabolite identification, and confirmation carried out using MS/MS is required"
- [intro] Lipidomics-specific annotation rationale: "it has been designed specifically for lipidomics applications, enabling putative lipid annotations based on accurate mass"
- [readme] Library composition per ionisation mode: "Lipid classes searched in positive ion mode are diacylglycerides (DAG), triacylglycerides (TAG), phosphatidylcholines (PC)"
