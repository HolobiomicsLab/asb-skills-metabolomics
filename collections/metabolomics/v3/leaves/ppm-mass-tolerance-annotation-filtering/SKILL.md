---
name: ppm-mass-tolerance-annotation-filtering
description: Use when processing .imzML mass spectrometry imaging data in massPix
  and you need putative lipid annotations for deisotoped ion features based on accurate
  mass within a defined ppm tolerance.
when_to_use_negative:
- The input is already a feature table with confirmed lipid annotations requiring
  no mass matching
- Only a database hit is insufficient for the analysis goal — accurate-mass matches
  must be confirmed by MS/MS, so this skill alone cannot provide definitive metabolite
  identification
- The analytes of interest are lipid classes outside those searched by massPix (e.g.
  not among DAG, TAG, PC, PE, PS, LysoPC, CE, SM, Cer, PA, PG, PI, FFA), since the
  generated library is restricted to common lipid classes
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: massPix
  role: R package providing the makelibrary function that computes lipid m/z values
    for all fatty-acid/head-group/adduct combinations and the annotate function that
    matches deisotoped ions against the generated library within the ppm threshold
  repo: https://github.com/hallz/massPix
- name: R
  role: Runtime environment in which the massPix library generation and ppm-thresholded
    annotation functions are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-017-1252-5
    title: 'massPix: an R package for annotation and interpretation of mass spectrometry
      imaging data for lipidomics'
  source_content_hash: sha256:bf44c00ba46ab85f832c4bfd08b904e0a44c9ed642cf4e41730f57bd65093adc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hallz/massPix
  tools:
  - R
  - massPix
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# ppm-mass-tolerance-annotation-filtering

## Summary

Annotates deisotoped mass spectrometry imaging ion features with putative lipid identities by accurate-mass matching against a generated lipid m/z library, retaining only matches whose |observed m/z - theoretical m/z| falls below a ppm threshold (e.g. 10 ppm).

## When to use

Use when processing .imzML mass spectrometry imaging data in massPix and you need putative lipid annotations for deisotoped ion features based on accurate mass within a defined ppm tolerance.

## When NOT to use

- The input is already a feature table with confirmed lipid annotations requiring no mass matching
- Only a database hit is insufficient for the analysis goal — accurate-mass matches must be confirmed by MS/MS, so this skill alone cannot provide definitive metabolite identification
- The analytes of interest are lipid classes outside those searched by massPix (e.g. not among DAG, TAG, PC, PE, PS, LysoPC, CE, SM, Cer, PA, PG, PI, FFA), since the generated library is restricted to common lipid classes

## Inputs

- Deisotoped ion feature list (m/z values) from massPix preprocessing (after mzextractor, peakpicker.bin, subsetImage and deisotope functions)
- massPix library lookup files from the 'libraries' folder (lib_class.csv, lib_FA.csv, lib_element.csv)
- Ionisation mode ('positive' or 'negative') determining which lipid classes and adducts are searched

## Outputs

- CSV file pairing each retained m/z with its putative lipid annotation (image.norm_short.csv-style output containing lipid annotations and m/z only)
- Full normalised image data matrix with lipid annotations, m/z and intensities (image.norm.csv)

## How to apply

First generate the lipid m/z library with the makelibrary function from the lookup files in the package's 'libraries' folder (lib_class.csv, lib_FA.csv, lib_element.csv), computing all combinations of common fatty acids, lipid head-groups and anticipated adducts for the chosen ionisation mode ([M+K]+, [M+H]+, [M+Na]+, [M+NH4]+ in positive mode; [M-H]-, [M+Cl]-, [M+OAc]- in negative mode). Then run the annotate function, which searches each deisotoped ion m/z against the generated library and retains only matches where |observed m/z - theoretical m/z| is below the ppm.annotate threshold; suggest using the same mass tolerance as the bin width (bin.ppm = 10 ppm for data acquired with mass resolving power 60,000 at m/z 400). Write retained m/z/annotation pairs to a CSV output (image.norm_short.csv-style, containing lipid annotations and m/z only). Positive-mode lipid classes searched include DAG, TAG, PC, PE, PS, LysoPC, CE, SM and Cer; negative-mode classes include PC, PA, PE, PS, PG, PI and FFA.

## Related tools

- **massPix** (R package providing the makelibrary function that computes lipid m/z values for all fatty-acid/head-group/adduct combinations and the annotate function that matches deisotoped ions against the generated library within the ppm threshold) — https://github.com/hallz/massPix
- **R** (Runtime environment in which the massPix library generation and ppm-thresholded annotation functions are executed)

## Examples

```
results <- massPix(process=T, ionisation_mode="positive", bin.ppm=10, ppm.annotate=10, thres.int=100000, thres.low=200, thres.high=1000)
```

## Evaluation signals

- Every retained annotation satisfies |observed m/z - theoretical m/z| < ppm.annotate (suggested 10 ppm, matching the bin.ppm bin width)
- The output CSV exists and contains lipid annotation and m/z columns (image.norm_short.csv-style output)
- The annotation m/z values fall within the preprocessed m/z range bounded by thres.low (200) and thres.high (1000) used when parsing the imzML data
- Only lipid classes valid for the chosen ionisation mode appear in the annotations (positive mode: DAG, TAG, PC, PE, PS, LysoPC, CE, SM, Cer; negative mode: PC, PA, PE, PS, PG, PI, FFA)
- Match criteria can be cross-checked by varying ppm.annotate with different MS performance capabilities and observing expected changes in the number of retained annotations

## Limitations

- A database hit based on accurate mass should only be considered the first step in metabolite identification; confirmation using MS/MS is required where appropriate
- The lipid class list searched is not exhaustive, though it covers the most common lipid classes
- The criteria for a match must be adjusted according to different MS performance capabilities (bin.ppm suggested starting at 10 ppm for mass resolving power 60,000 at m/z 400, to be increased/decreased with lower/higher resolving power)
- Annotations are putative only and based on accurate mass, not structural confirmation

## Evidence

- [methods] Putative lipid annotation by accurate mass is achieved by searching deisotoped ions against a generated library of lipid m/z ratios: "Putative lipid annotation by accurate mass is achieved by searching deisotoped ions against a generated library of lipid m/z ratios"
- [readme] ppm.annotate threshold definition: "Defines ppm threshold for which |observed m/z - theoretical m/z| must be less than for annotation to be retained. Suggest to use same mass tolerance (ppm.annotate) as bin width (bin.ppm)"
- [readme] library generation description: "computed for all combinations of common fatty acids, lipid head-groups and anticipated adducts in each ionisation mode (functions: makelibrary and annotate)"
- [readme] MS/MS confirmation caveat: "a database hit based on accurate mass should only be considered the first step in metabolite identification, and confirmation carried out using MS/MS is required"
