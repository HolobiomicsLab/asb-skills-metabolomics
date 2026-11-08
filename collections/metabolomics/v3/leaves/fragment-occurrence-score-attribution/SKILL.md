---
name: fragment-occurrence-score-attribution
description: Use when you need to generate a new metabolite fragment-library entry
  (.csv) from an experimental or MassBank .msp MS/MS spectrum, or when extending/customizing
  the default MetaboAnnotatoR libraries with scored marker peaks.
when_to_use_negative:
- Input is already a MetaboAnnotatoR-formatted .csv library entry or a complete processed
  feature table — the skill builds library entries, it does not annotate features.
- Acquisition is a data-independent scheme other than AIF (e.g., SWATH), which MetaboAnnotatoR
  does not currently support.
- You need annotation on instruments the authors did not test (e.g., Orbitraps), where
  library m/z accuracy assumptions may not hold as validated.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnnotatoR
  role: Provides genFragEntry and mspToLib, the functions that attribute occurrence
    scores to marker peaks and write the .csv library entry
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: R
  role: Runtime environment in which the MetaboAnnotatoR package functions are invoked
- name: MassBank
  role: Source of public-domain .msp MS/MS records (e.g., MSBNK-RIKEN-PR100295) that
    can be converted with mspToLib
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:b116e1b765be26c0e15582ae269df01a1e171be25e03b4818ff8b8e9ef128235
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - MassBank
  - MetaboAnnotatoR
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# fragment-occurrence-score-attribution

## Summary

Builds MetaboAnnotatoR fragment-library entries by assigning occurrence scores to fragment ('marker') peaks of an experimental MS/MS spectrum using genFragEntry, so that pseudo-MS/MS features can later be matched by scored fragments. Entries are written as .csv library rows and .msp records can be converted to the same format with mspToLib.

## When to use

Use when you need to generate a new metabolite fragment-library entry (.csv) from an experimental or MassBank .msp MS/MS spectrum, or when extending/customizing the default MetaboAnnotatoR libraries with scored marker peaks.

## When NOT to use

- Input is already a MetaboAnnotatoR-formatted .csv library entry or a complete processed feature table — the skill builds library entries, it does not annotate features.
- Acquisition is a data-independent scheme other than AIF (e.g., SWATH), which MetaboAnnotatoR does not currently support.
- You need annotation on instruments the authors did not test (e.g., Orbitraps), where library m/z accuracy assumptions may not hold as validated.

## Inputs

- Experimental MS/MS spectrum of a metabolite standard (or a public-database .msp record, e.g., MassBank MSBNK-RIKEN-PR100295)
- Metabolite name, adduct label, and accurate adduct m/z
- mpeaksThres threshold value and noise-level cutoff

## Outputs

- MetaboAnnotatoR .csv fragment-library entry (metabolite name, adduct, adduct m/z, fragment m/z values, occurrence scores)
- Same-format library entry produced via mspToLib from .msp input

## How to apply

Run MetaboAnnotatoR's genFragEntry on the experimental MS/MS spectrum (e.g., the Pantothenic acid [M+H]+ MassBank record MSBNK-RIKEN-PR100295), supplying the metabolite name, adduct, and accurate adduct m/z. The function attributes occurrence scores to the peaks that fall above the mpeaksThres threshold and above the noise level — these are the 'marker' peaks — and writes the scored peak list as a MetaboAnnotatoR .csv library entry under the given output filename. For spectra obtained from public databases, use mspToLib to parse .msp records and produce the same library-entry format. Validate the generated entry against a known example (e.g., the Pantothenic acid [M+H]+ entry in the package vignette) by comparing fragment m/z values and occurrence scores; discrepancies indicate incorrect occurrence attribution or thresholding.

## Related tools

- **MetaboAnnotatoR** (Provides genFragEntry and mspToLib, the functions that attribute occurrence scores to marker peaks and write the .csv library entry) — https://github.com/gggraca/MetaboAnnotatoR
- **R** (Runtime environment in which the MetaboAnnotatoR package functions are invoked)
- **MassBank** (Source of public-domain .msp MS/MS records (e.g., MSBNK-RIKEN-PR100295) that can be converted with mspToLib)

## Examples

```
library(MetaboAnnotatoR); genFragEntry(spec, name="Pantothenic acid", adduct="[M+H]+", adductmz=220.1089, mpeaksThres=mpeaksThres, outfile="pantothenate_lib.csv")
```

## Evaluation signals

- Generated .csv entry matches the vignette's example Pantothenic acid [M+H]+ entry (MSBNK-RIKEN-PR100295) in both fragment m/z values and occurrence scores
- Every retained fragment peak is above the mpeaksThres threshold and above the noise level
- Entry contains the metabolite name, adduct label, and accurate adduct m/z as explicitly supplied
- mspToLib output for the same record is consistent with the genFragEntry-built entry format
- Fragment m/z values fall within the mass accuracy range of the instrument used (low-res AIF tested; Orbitrap accuracy untested by authors)

## Limitations

- Only AIF data-independent acquisition is supported; other DIA schemes such as SWATH are not.
- Occurrence-scored libraries inherit the small size of the nonlipid fragment library, which drove low recall (10–88%) on non-serum matrices in the paper.
- Performance depends on the chosen mpeaksThres and noise-level settings; poor choices change which peaks are scored.
- Isotopologues and in-source fragments are handled downstream by the annotation workflow, not during entry generation.

## Evidence

- [other] Library entries are generated from experimental MS/MS spectra using genFragEntry, attributing occurrence scores to peaks above a mpeaksThres threshold ('marker peaks') and above the noise level.: [evidence span withheld — anchor status: not_found]
- [readme] Vignettes cover generation of metabolite fragment database entries from experimental and .msp spectra.: "- Generation of Metabolite fragment database entry from MS/MS experimental spectra.
- Generation of Metabolite fragment database entry from MS/MS spectra from public databases (in .msp format)."
- [discussion] External libraries can be imported as .txt and .msp files.: "these can be customized by the user, for instance, by importing external libraries as .txt and .msp files."
