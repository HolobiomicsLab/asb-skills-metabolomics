---
name: deisotoped-ion-library-matching
description: Use when deisotoped m/z features from high-resolution MSI lipidomics
  data (e.g. imzML-parsed ion features) need putative lipid identities via accurate-mass
  matching within ~10 ppm.
when_to_use_negative:
- Input is not high-resolution MSI lipidomics data with deisotoped ion features (e.g.
  already-annotated feature tables or non-lipid metabolite targets)
- Definitive metabolite identification is required — a database hit based on accurate
  mass is only the first step and must be confirmed by MS/MS
- Lipid classes of interest fall outside the searched classes (DAG, TAG, PC, PE, PS,
  LysoPC, CE, SM, Cer in positive mode; PC, PA, PE, PS, PG, PI, FFA in negative mode),
  which is not exhaustive
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3292
tools:
- name: massPix
  role: Provides the makelibrary function that generates the lipid m/z library and
    the annotate function that performs accurate-mass matching of deisotoped ions
    within a ppm threshold
  repo: https://github.com/hallz/massPix
- name: R
  role: Runtime environment in which the massPix package is installed and the makelibrary/annotate
    workflow is executed
  repo: https://github.com/hallz/massPix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-017-1252-5
    title: 'massPix: an R package for annotation and interpretation of mass spectrometry
      imaging data for lipidomics'
  source_content_hash: sha256:c7c4b39d03677b0b412553090223b1c71c78d33d3aafccc2c01962280c5f313a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hallz/massPix
  tools:
  - R
  - massPix
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# deisotoped-ion-library-matching

## Summary

Putative lipid annotation of deisotoped mass spectrometry imaging (MSI) features by accurate-mass matching against a lipid m/z library generated on the fly for all combinations of fatty acids, head-groups and adducts in a given ionisation mode, using massPix's makelibrary and annotate functions in R.

## When to use

Use when deisotoped m/z features from high-resolution MSI lipidomics data (e.g. imzML-parsed ion features) need putative lipid identities via accurate-mass matching within ~10 ppm.

## When NOT to use

- Input is not high-resolution MSI lipidomics data with deisotoped ion features (e.g. already-annotated feature tables or non-lipid metabolite targets)
- Definitive metabolite identification is required — a database hit based on accurate mass is only the first step and must be confirmed by MS/MS
- Lipid classes of interest fall outside the searched classes (DAG, TAG, PC, PE, PS, LysoPC, CE, SM, Cer in positive mode; PC, PA, PE, PS, PG, PI, FFA in negative mode), which is not exhaustive

## Inputs

- Deisotoped ion m/z feature table from massPix pre-processing (mzextractor, peakpicker.bin, subsetImage, deisotope)
- massPix library lookup files from the package 'libraries' folder (lib_class.csv, lib_FA.csv, lib_element.csv)
- Ionisation mode setting ('positive' or 'negative') determining which lipid classes and adducts are searched
- ppm.annotate threshold parameter (suggested 10, matching bin.ppm)

## Outputs

- CSV pairing each retained m/z with its putative lipid annotation (image.norm_short.csv-style output containing lipid annotations and m/z only)
- Full normalised image data matrix with lipid annotations, m/z and intensities (image.norm.csv)

## How to apply

After the massPix pipeline has deisotoped ions (filtered and deisotope removes M+1 and M+2 isotopes within a calculated proportion of the monoisotope), load the library lookup files (lib_class.csv, lib_FA.csv, lib_element.csv) from the package 'libraries' folder and run makelibrary for the chosen ionisation mode ('positive' or 'negative') to compute m/z values for all combinations of common fatty acids, lipid head-groups and anticipated adducts ([M+K]+, [M+H]+, [M+Na]+, [M+NH4]+ in positive mode; [M-H]-, [M+Cl]-, [M+OAc]- in negative mode). Then run annotate with ppm.annotate = 10, matching each deisotoped ion only where |observed m/z - theoretical m/z| is below the ppm threshold; set ppm.annotate to the same value as the bin.ppm bin width (suggested 10 ppm for 60,000 resolving power at m/z 400). Match criteria can be adjusted according to different MS performance capabilities, and library composition (lipid classes, adducts) can be customised via parameters such as sel.class, adducts and fixed_FA. Write the retained m/z-to-annotation pairs to a CSV (image.norm_short.csv-style output containing lipid annotations and m/z only).

## Related tools

- **massPix** (Provides the makelibrary function that generates the lipid m/z library and the annotate function that performs accurate-mass matching of deisotoped ions within a ppm threshold) — https://github.com/hallz/massPix
- **R** (Runtime environment in which the massPix package is installed and the makelibrary/annotate workflow is executed) — https://github.com/hallz/massPix

## Examples

```
library(massPix); results <- massPix(process=T, ionisation_mode="positive", bin.ppm=10, ppm.annotate=10)
```

## Evaluation signals

- Every retained annotation has |observed m/z - theoretical m/z| below the ppm.annotate threshold (suggested 10 ppm, same as the bin.ppm bin width)
- An output CSV exists containing m/z and lipid annotation columns (image.norm_short.csv-style)
- All annotated lipids correspond to combinations of fatty acids, head-groups and adducts available in the generated library for the chosen ionisation mode (e.g. [M+H]+, [M+Na]+, [M+K]+, [M+NH4]+ in positive mode)
- Lipid classes reported match the mode-specific search list (e.g. PC, PA, PE, PS, PG, PI, FFA in negative ion mode)

## Limitations

- A database hit based on accurate mass should only be considered the first step in metabolite identification; confirmation using MS/MS is required where appropriate
- The list of lipid classes searched per ionisation mode is not exhaustive, although it covers the most common classes
- Match criteria (ppm.annotate) must be tuned to instrument MS performance — the suggested 10 ppm assumes mass resolving power of 60,000 at m/z 400; annotation is a lipidomics-specific capability and annotations remain putative

## Evidence

- [methods] Putative lipid annotation by accurate mass is achieved by searching deisotoped ions against a generated library of lipid m/z ratios: "Putative lipid annotation by accurate mass is achieved by searching deisotoped ions against a generated library of lipid m/z ratios"
- [readme] Possible adducts considered are [M+K]+, [M+H]+, [M+Na]+, [M+NH4]+ in positive ion mode and [M–H]-, [M+Cl]-, [M+OAc]- in negative ion mode.: "Possible adducts considered are [M+K]+, [M+H]+, [M+Na]+, [M+NH4]+ in positive ion mode and [M–H]-, [M+Cl]-, [M+OAc]- in negative ion mode."
- [readme] ppm.annotate - Defines ppm threshold for which |observed m/z - theoretical m/z| must be less than for annotation to be retained.: "ppm.annotate - Defines ppm threshold for which |observed m/z - theoretical m/z| must be less than for annotation to be retained."
- [readme] It is important to point out that a database hit based on accurate mass should only be considered the first step in metabolite identification: "It is important to point out that a database hit based on accurate mass should only be considered the first step in metabolite identification, and confirmation carried out using MS/MS is required"
