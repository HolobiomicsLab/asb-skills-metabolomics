---
name: lipid-mass-library-construction
description: Use when deisotoped MSI ion features must be assigned putative lipid
  identities via accurate-mass matching, requiring a generated in silico lipid m/z
  library for the chosen positive or negative ionisation mode.
when_to_use_negative:
- Input is already a curated, externally supplied lipid m/z database rather than a
  need to generate one from fatty-acid/head-group/adduct combinations
- Definitive lipid identification is required — accurate-mass library matching is
  only the first step and must be confirmed by MS/MS
- The analytes of interest fall outside the searched lipid classes (e.g. non-lipid
  metabolites), since the library covers only common lipid classes
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: massPix
  role: R package providing the makelibrary and annotate functions, library lookup
    files and lipid annotation workflow
  repo: https://github.com/hallz/massPix
- name: R
  role: Runtime environment in which the massPix package functions are executed
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1007/s11306-017-1252-5
    title: 'massPix: an R package for annotation and interpretation of mass spectrometry
      imaging data for lipidomics'
  source_content_hash: sha256:486a908d86f0207ecbac6d3da473ccf7397479fa4ebd6204794e0c28aba342fd
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# lipid-mass-library-construction

## Summary

Builds a theoretical lipid m/z reference library for mass spectrometry imaging annotation by enumerating all combinations of common fatty acids, lipid head-groups and anticipated adducts per ionisation mode, using massPix's library files and makelibrary function in R.

## When to use

Use when deisotoped MSI ion features must be assigned putative lipid identities via accurate-mass matching, requiring a generated in silico lipid m/z library for the chosen positive or negative ionisation mode.

## When NOT to use

- Input is already a curated, externally supplied lipid m/z database rather than a need to generate one from fatty-acid/head-group/adduct combinations
- Definitive lipid identification is required — accurate-mass library matching is only the first step and must be confirmed by MS/MS
- The analytes of interest fall outside the searched lipid classes (e.g. non-lipid metabolites), since the library covers only common lipid classes

## Inputs

- massPix library lookup CSV files: lib_class.csv, lib_FA.csv, lib_element.csv from the package's 'libraries' folder
- ionisation mode string ('positive' or 'negative') determining lipid classes and adducts
- optional customisation: library composition, lipid class selection (sel.class), adduct list, fatty-acid lookup (lookup_FA), modified-species search settings (search.mod, lookup_mod)

## Outputs

- Generated lipid m/z library computed for all combinations of fatty acids, head-groups and adducts in each ionisation mode
- Putative lipid annotations paired with m/z in image.norm_short.csv-style output (lipid annotations and m/z only), plus full image.norm.csv with intensities

## How to apply

Load the massPix library lookup files (lib_class.csv, lib_FA.csv, lib_element.csv) from the package's 'libraries' folder with read.csv in R. Run the makelibrary function for the chosen ionisation_mode ('positive' or 'negative'), which computes lipid m/z values for all combinations of common fatty acids, lipid head-groups and anticipated adducts: [M+K]+, [M+H]+, [M+Na]+, [M+NH4]+ in positive mode and [M–H]-, [M+Cl]-, [M+OAc]- in negative mode. In positive mode the classes covered are DAG, TAG, PC, PE, PS, LysoPC, CE, SM and Cer; in negative mode PC, PA, PE, PS, PG, PI and FFA. The generated library is then used by the annotate function, where a match is retained only when |observed m/z - theoretical m/z| is below the ppm.annotate threshold (suggested 10 ppm, matching the bin.ppm bin width). Because a database hit based on accurate mass is only the first step in metabolite identification, follow-up confirmation by MS/MS is required where appropriate.

## Related tools

- **massPix** (R package providing the makelibrary and annotate functions, library lookup files and lipid annotation workflow) — https://github.com/hallz/massPix
- **R** (Runtime environment in which the massPix package functions are executed)

## Examples

```
lib_class <- read.csv("libraries/lib_class.csv"); lib_FA <- read.csv("libraries/lib_FA.csv"); lib_element <- read.csv("libraries/lib_element.csv"); makelibrary(ionisation_mode="positive")
```

## Evaluation signals

- The makelibrary run completes for the requested ionisation_mode without error and includes the expected classes (e.g. DAG, TAG, PC, PE, PS, LysoPC, CE, SM, Cer in positive mode)
- Every retained annotation satisfies |observed m/z - theoretical m/z| below the ppm.annotate threshold (suggested 10 ppm, matching bin.ppm)
- Output CSV (image.norm_short.csv-style) exists and contains paired m/z and lipid annotation columns
- Annotation counts behave sensibly with tolerance: 50 ppm yields more annotations than 5 ppm (~200 additional annotations on the MTBLS487 test data), and ppm.annotate should match the bin.ppm bin width
- Adduct coverage matches the ionisation mode ([M+H]+, [M+Na]+, [M+K]+, [M+NH4]+ for positive; [M–H]-, [M+Cl]-, [M+OAc]- for negative)

## Limitations

- Accurate-mass database hits are only the first step in metabolite identification; MS/MS confirmation is required where appropriate
- The lipid class list is not exhaustive, though it covers the most common lipid classes; coverage differs between ionisation modes
- Match criteria (ppm.annotate) must be adjusted according to the instrument's MS performance/mass accuracy

## Evidence

- [methods] Putative lipid annotation by accurate mass is achieved by searching deisotoped ions against a generated library of lipid m/z ratios: "Putative lipid annotation by accurate mass is achieved by searching deisotoped ions against a generated library of lipid m/z ratios"
- [readme] Lipid classes searched in positive ion mode: "Lipid classes searched in positive ion mode are diacylglycerides (DAG), triacylglycerides (TAG), phosphatidylcholines (PC), phosphatidylethanolamines (PE), phosphatidylserines (PS), LysoPC"
- [readme] Possible adducts considered: "Possible adducts considered are [M+K]+, [M+H]+, [M+Na]+, [M+NH4]+ in positive ion mode and [M–H]-, [M+Cl]-, [M+OAc]- in negative ion mode."
- [readme] ppm.annotate threshold guidance: "ppm.annotate - Defines ppm threshold for which |observed m/z - theoretical m/z| must be less than for annotation to be retained."
- [readme] Accurate-mass hits need MS/MS confirmation: "a database hit based on accurate mass should only be considered the first step in metabolite identification, and confirmation carried out using MS/MS is required"
