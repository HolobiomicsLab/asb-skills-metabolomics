---
name: adduct-and-lipid-class-search-space-definition
description: Use when building a theoretical lipid m/z library for accurate-mass annotation
  of deisotoped MSI features and you must choose which lipid classes and adducts to
  enumerate for the given ionisation mode.
when_to_use_negative:
- When the goal is unambiguous metabolite identification — a database hit based on
  accurate mass is only the first step and requires MS/MS confirmation
- When annotating non-lipid metabolites — the default library covers only the most
  common lipid classes and is not exhaustive
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: massPix
  role: R package providing makelibrary/annotate functions and the library lookup
    files used to build and search the lipid m/z library
  repo: https://github.com/hallz/massPix
- name: R
  role: Runtime environment in which the massPix library generation and annotation
    functions are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-017-1252-5
    title: 'massPix: an R package for annotation and interpretation of mass spectrometry
      imaging data for lipidomics'
  source_content_hash: sha256:0438bded7604562bc73d73def0071c9d91b762ff450770aee1a5002f36d4b783
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hallz/massPix
  tools:
  - R
  - massPix
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Adduct and lipid class search space definition

## Summary

Defines the lipid class, fatty-acid and adduct combinations that form the theoretical m/z library used for accurate-mass annotation of MSI features in massPix. Constraining the search space to ionisation-mode-appropriate classes and adducts controls false annotations in lipidomics imaging data.

## When to use

Use when building a theoretical lipid m/z library for accurate-mass annotation of deisotoped MSI features and you must choose which lipid classes and adducts to enumerate for the given ionisation mode.

## When NOT to use

- When the goal is unambiguous metabolite identification — a database hit based on accurate mass is only the first step and requires MS/MS confirmation
- When annotating non-lipid metabolites — the default library covers only the most common lipid classes and is not exhaustive

## Inputs

- lib_class.csv, lib_FA.csv and lib_element.csv lookup files from the massPix 'libraries' folder
- ionisation mode designation ('positive' or 'negative') from processing_script.R
- Optional: custom sel.class, fixed, fixed_FA and lookup tables passed to the massPix() wrapper

## Outputs

- Generated library of theoretical lipid m/z values for all fatty-acid/head-group/adduct combinations (via makelibrary)
- Lipid annotations paired with matched m/z values in the image.norm_short.csv-style output

## How to apply

Load the library lookup files (lib_class.csv, lib_FA.csv, lib_element.csv) from the massPix 'libraries' folder via read.csv and run makelibrary for the chosen ionisation_mode ('positive' or 'negative'). Restrict lipid classes per mode: DAG, TAG, PC, PE, PS, LysoPC, CE, SM and Cer in positive mode; PC, PA, PE, PS, PG, PI and FFA in negative mode. Enumerate anticipated adducts: [M+K]+, [M+H]+, [M+Na]+, [M+NH4]+ in positive mode and [M–H]-, [M+Cl]-, [M+OAc]- in negative mode. The function computes m/z values for all combinations of common fatty acids, head-groups and adducts; downstream annotate() then matches deisotoped ions against this library using ppm.annotate = 10 (suggested to equal bin.ppm). The search space can be customised via sel.class, fixed, fixed_FA, and the lookup_lipid_class/lookup_FA/lookup_element tables.

## Related tools

- **massPix** (R package providing makelibrary/annotate functions and the library lookup files used to build and search the lipid m/z library) — https://github.com/hallz/massPix
- **R** (Runtime environment in which the massPix library generation and annotation functions are executed)

## Examples

```
lib_class <- read.csv('libraries/lib_class.csv'); lib_FA <- read.csv('libraries/lib_FA.csv'); lib_element <- read.csv('libraries/lib_element.csv'); makelibrary(ionisation_mode='positive', lookup_lipid_class=lib_class, lookup_FA=lib_FA, lookup_element=lib_element)
```

## Evaluation signals

- Every annotation in the output CSV satisfies |observed m/z − theoretical m/z| below the ppm.annotate threshold (suggested 10 ppm, matching bin.ppm)
- Set of searchable lipid classes matches the declared ionisation_mode (e.g. no CE or SM entries when ionisation_mode = 'negative')
- Only the anticipated adducts for the mode appear among matched species ([M+K]+, [M+H]+, [M+Na]+, [M+NH4]+ in positive; [M–H]-, [M+Cl]-, [M+OAc]- in negative)
- image.norm_short.csv exists and contains lipid annotation and m/z columns for retained features

## Limitations

- The default lipid class list is not exhaustive, covering only the most common lipid classes per ionisation mode
- Accurate-mass database hits are only putative — confirmation by MS/MS is required where appropriate
- Match criteria must be adjusted to the instrument's mass accuracy (ppm.annotate should match bin.ppm, e.g. 10 ppm for 60,000 resolving power at m/z 400)

## Evidence

- [readme] Lipid classes searched in positive ion mode are diacylglycerides (DAG), triacylglycerides (TAG), phosphatidylcholines (PC), phosphatidylethanolamines (PE), phosphatidylserines (PS), LysoPC, cholesteryl esters (CE), sphingomyelins (SM) and ceramides (Cer).: "Lipid classes searched in positive ion mode are diacylglycerides (DAG), triacylglycerides (TAG), phosphatidylcholines (PC), phosphatidylethanolamines (PE), phosphatidylserines (PS), LysoPC,"
- [readme] Possible adducts considered are [M+K]+, [M+H]+, [M+Na]+, [M+NH4]+ in positive ion mode and [M–H]-, [M+Cl]-, [M+OAc]- in negative ion mode.: "Possible adducts considered are [M+K]+, [M+H]+, [M+Na]+, [M+NH4]+ in positive ion mode and [M–H]-, [M+Cl]-, [M+OAc]- in negative ion mode."
- [methods] Putative lipid annotation by accurate mass is achieved by searching deisotoped ions against a generated library of lipid m/z ratios.: "Putative lipid annotation by accurate mass is achieved by searching deisotoped ions against a generated library of lipid m/z ratios"
- [readme] a database hit based on accurate mass should only be considered the first step in metabolite identification, and confirmation carried out using MS/MS is required: "a database hit based on accurate mass should only be considered the first step in metabolite identification, and confirmation carried out using MS/MS is required, where this appropriate."
- [readme] ppm.annotate - Defines ppm threshold for which |observed m/z - theoretical m/z| must be less than for annotation to be retained.: "ppm.annotate - Defines ppm threshold for which |observed m/z - theoretical m/z| must be less than for annotation to be retained."
