---
name: deposited-library-consistency-checking
description: Use when regenerating the CalicoLipids in-silico lipid spectral library
  and you must confirm the locally produced msp output is consistent with the published
  MTBLS3097 deposit before downstream MS/MS matching.
when_to_use_negative:
- When the goal is spectral matching of experimental LC-MS/MS data against an existing
  library (use MAVEN2 search instead of regenerating and validating the library)
- When the input is already a validated msp file from MTBLS3097 that requires no local
  regeneration
- When the lipid class/adduct combination is unsupported (README warns 'Not all adduct
  names are valid for all lipid classes' and unknown classes like ImaginaryClass are
  silently removed)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: CalicoLipidLibrary
  role: Generates the in-silico lipid fragmentation spectra and msp library files
    that are validated against the MTBLS3097 deposit
  repo: https://github.com/calico/CalicoLipidLibrary
- name: MAVEN2
  role: Consumer of the msp libraries; its MS/MS spectral matching and efficient search
    of large-scale fragmentation libraries define the required msp compatibility
  repo: https://github.com/eugenemel/maven
- name: git
  role: Used to retrieve the CalicoLipidLibrary and maven source repositories for
    regeneration
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:caa5548dc9d0a0ff9af6071d7485732ba9b0a5de2b2d9fa4b06a0f45249851a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calico/CalicoLipidLibrary
  tools:
  - CalicoLipidLibrary
  - MAVEN2
  - git
  - make
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# deposited-library-consistency-checking

## Summary

Verifies that locally regenerated in-silico lipid fragmentation libraries (msp files from CalicoLipidLibrary) match the reference library deposited at MetaboLights accession MTBLS3097, ensuring reproducibility of the HCD-mimicking fragmentation model before using the library in MS/MS spectral matching.

## When to use

Use when regenerating the CalicoLipids in-silico lipid spectral library and you must confirm the locally produced msp output is consistent with the published MTBLS3097 deposit before downstream MS/MS matching.

## When NOT to use

- When the goal is spectral matching of experimental LC-MS/MS data against an existing library (use MAVEN2 search instead of regenerating and validating the library)
- When the input is already a validated msp file from MTBLS3097 that requires no local regeneration
- When the lipid class/adduct combination is unsupported (README warns 'Not all adduct names are valid for all lipid classes' and unknown classes like ImaginaryClass are silently removed)

## Inputs

- CalicoLipidLibrary code repository (github.com/calico/CalicoLipidLibrary)
- Reference msp library files from MetaboLights accession MTBLS3097
- Configuration: ionization mode (pos|neg), lipid class subset (-c), output path (-o), output file name (-n)

## Outputs

- Generated msp-format in-silico lipid spectral library files
- Validation report: parse check (NIST msp), class/adduct coverage counts, sampled record diffs vs. MTBLS3097

## How to apply

Clone the calico/CalicoLipidLibrary repository and download the deposited msp files from MetaboLights accession MTBLS3097. Run the generator (generateDB.py) across all supported lipid classes — enumeration covers 61 classes and 15 adduct forms, with positive and negative ionization modes selected via the -m flag — applying the HCD-mimicking model with stepped normalized collision energies of 20, 30, and 40. Serialize spectra to msp format compatible with MAVEN2's MS/MS spectral matching search. For validation, parse the generated files in NIST msp format, confirm entries exist for all 61 lipid classes and 15 adduct forms, and compare a sampled set of records against corresponding msp entries in the MTBLS3097 deposit (output_matches_reference). Because enumerating all classes is time-consuming, partial builds (e.g. -c 'PC,PE,PS') can be validated against the matching subset of the deposit.

## Related tools

- **CalicoLipidLibrary** (Generates the in-silico lipid fragmentation spectra and msp library files that are validated against the MTBLS3097 deposit) — https://github.com/calico/CalicoLipidLibrary
- **MAVEN2** (Consumer of the msp libraries; its MS/MS spectral matching and efficient search of large-scale fragmentation libraries define the required msp compatibility) — https://github.com/eugenemel/maven
- **git** (Used to retrieve the CalicoLipidLibrary and maven source repositories for regeneration)

## Examples

```
python2.7 generateDB.py -m "neg" -c "AcGM2, AcGM3, GcGM2, GcGM3, MIP2C, MIPC, HexCer, LacCer, Sulfatide, SM, Ceramide, Ceramide_P" -n "neg_sphingolipids"
```

## Evaluation signals

- Generated msp files parse successfully in the NIST msp format
- msp entries cover all 61 lipid classes and 15 adduct forms enumerated by the generation rules
- A sampled set of generated spectra records exactly matches the corresponding msp entries in the MTBLS3097 deposit (output_matches_reference check)
- Expected adduct forms present: [M+H]+, [M+Na]+, [M+NH4]+, [M+K]+, [M+2H]2+, [M+Li]+, [M-H]-, [M+Cl]-, [M+37Cl]-, [M+FA-H]-, [M-CH3]-, [M+AcOH-H]-, [M-2H]2-, [M-3H]3-
- CalicoLipidLibrary's own pytest suite passes (pytest tests)

## Limitations

- Full enumeration over all lipid classes is time-consuming; partial libraries via -c are recommended for iteration, but class subsets must be diffed only against the corresponding deposit subset
- Invalid lipid class names (e.g. ImaginaryClass) are silently removed from the -c list rather than raising errors, which can mask coverage gaps
- Only Python 2.7 is supported for the interactive print_spectrum path; pypy is suggested for speed but changes the runtime
- Not all adduct names are valid for all lipid classes, so per-class adduct coverage varies from the full 15-adduct list

## Evidence

- [readme] Scripts to create in silico lipid fragmentation spectra, based on analysis of chemical standards and relevant scientific literature.: [evidence span withheld — anchor status: external_doc]
- [readme] Not all adduct names are valid for all lipid classes.: [evidence span withheld — anchor status: external_doc]
- [abstract] We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries.: "We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries."
- [readme] Generate a subset of the library for a set of classes of interest.: "Enumerating all theoretical spectra over all lipid classes may be time-consuming, so the `-c` option may be used
to generate a subset of the library for a set of classes of interest."
- [readme] If the lipid class or adduct form is not supported, a list of all supported classes and adducts will appear.: [evidence span withheld — anchor status: external_doc]
