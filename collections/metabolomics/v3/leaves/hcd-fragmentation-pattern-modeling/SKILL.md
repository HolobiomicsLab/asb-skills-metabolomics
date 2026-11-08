---
name: hcd-fragmentation-pattern-modeling
description: Use when building or verifying an in-silico lipid MS/MS reference library,
  or when reproducing the rule-based fragmentation spectra behind the CalicoLipids
  library deposited under MetaboLights MTBLS3097.
when_to_use_negative:
- Input is raw LC-MS/MS data needing peak detection or alignment — use MAVEN2's LC-MS/MS
  analysis workflow instead of library generation
- 'The lipid class/adduct combination is unsupported: print_spectrum() will error
  or the class is silently removed (e.g. ImaginaryClass), and not all adducts are
  valid for all classes'
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: CalicoLipidLibrary
  role: Generates the in-silico lipid fragmentation spectra and serializes them to
    msp-format library files
  repo: https://github.com/calico/CalicoLipidLibrary
- name: MAVEN2
  role: Consumes the msp libraries for MS/MS spectral matching and efficient search
    of large-scale fragmentation libraries
  repo: https://github.com/eugenemel/maven
- name: MAVEN
  role: Predecessor platform; MAVEN2 is the major update supporting LC-MS/MS analysis
    of metabolomics and lipidomics samples
  repo: https://github.com/eugenemel/maven
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:e6267559aa7a627e473b747b490b573ea5c667467347443825658ad90f304536
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calico/CalicoLipidLibrary
  tools:
  - CalicoLipidLibrary
  - MAVEN
  - MAVEN2
  - git
  - make
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7368421052631579
n_claims: 19
---

# HCD fragmentation pattern modeling

## Summary

Rule-based in-silico modeling of HCD-mimicking lipid fragmentation to enumerate product-ion spectra across lipid classes and adduct forms, producing msp-format spectral libraries (deposited as MTBLS3097) used for MS/MS spectral matching in MAVEN2.

## When to use

Use when building or verifying an in-silico lipid MS/MS reference library, or when reproducing the rule-based fragmentation spectra behind the CalicoLipids library deposited under MetaboLights MTBLS3097.

## When NOT to use

- Input is raw LC-MS/MS data needing peak detection or alignment — use MAVEN2's LC-MS/MS analysis workflow instead of library generation
- The lipid class/adduct combination is unsupported: print_spectrum() will error or the class is silently removed (e.g. ImaginaryClass), and not all adducts are valid for all classes

## Inputs

- Lipid class list (e.g. 'PC,PE,PS' or all 61 supported classes)
- Ionization mode (pos or neg)
- Adduct forms supported per lipid class (e.g. "[M+H]+", "[M-H]-", "[M+NH4]+", "[M+2H]2+", "[M-2H]2-")
- Acyl/alkyl chain compositions as [num_carbons, num_double_bonds, num_oxidations] tuples

## Outputs

- msp-format in-silico lipid spectral library file (e.g. <date>-Calico-Lipids-<all|<classes>>-<ion>.msp)
- Reference library deposited at MetaboLights accession MTBLS3097

## How to apply

Clone calico/CalicoLipidLibrary and build the enumeration generator (generateDB.py), which enumerates theoretical spectra across the supported lipid classes (e.g. PC, PE, PS, Ceramide, SM, CL, HexCer) and adduct forms ([M+H]+, [M+NH4]+, [M-H]-, [M+FA-H]-, [M-2H]2-, etc.). Apply the HCD-mimicking fragmentation model — calibrated by analysis of chemical standards and relevant literature — using stepped normalized collision energies of 20, 30, and 40 to compute product ions for each lipid-adduct-chain combination, using the -m flag to select positive or negative ionization mode and -c to restrict to a subset of classes. Serialize the enumerated spectra to msp-format library files compatible with MS/MS spectral matching search in MAVEN2, which implements efficient search of large-scale fragmentation libraries. Validation: confirm the msp output parses in NIST msp format, covers all lipid classes and adduct forms requested, and that sampled records match the reference msp entries in the MTBLS3097 deposit.

## Related tools

- **CalicoLipidLibrary** (Generates the in-silico lipid fragmentation spectra and serializes them to msp-format library files) — https://github.com/calico/CalicoLipidLibrary
- **MAVEN2** (Consumes the msp libraries for MS/MS spectral matching and efficient search of large-scale fragmentation libraries) — https://github.com/eugenemel/maven
- **MAVEN** (Predecessor platform; MAVEN2 is the major update supporting LC-MS/MS analysis of metabolomics and lipidomics samples) — https://github.com/eugenemel/maven

## Examples

```
python2.7 generateDB.py -m "pos" -c "PC,PE,PS" -n "glycophospholipids_pos.msp"
```

## Evaluation signals

- Generated msp files parse in the NIST msp format without errors
- Library contains entries for all requested lipid classes (61 supported) and 15 adduct forms per the generation rules
- Sampled msp records match the corresponding entries in the MTBLS3097 deposit (output_matches_reference on representative spectra)
- pytest tests in the CalicoLipidLibrary repository pass (pytest tests)
- Invalid class/adduct combinations return the documented error rather than a spectrum (fragmentation must make chemical sense for the chain composition)

## Limitations

- Enumerating all theoretical spectra over all lipid classes may be time-consuming; use the -c option to subset or pypy for faster execution
- Only Python 2 (python2.7) is supported for the interactive single-spectrum API
- Not all adduct names are valid for all lipid classes; unsupported adducts or chemically nonsensical fragmentations raise errors
- Changelog for some MAVEN releases is incomplete, so version-specific library-generation changes may be undocumented

## Evidence

- [readme] Scripts to create in silico lipid fragmentation spectra, based on analysis of chemical standards and relevant scientific literature.: [evidence span withheld — anchor status: external_doc]
- [readme] Enumerating all theoretical spectra over all lipid classes may be time-consuming, so the `-c` option may be used: [evidence span withheld — anchor status: external_doc]
- [abstract] We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries.: "We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries."
- [abstract] we introduce a novel in-silico lipidomics library covering major lipid classes and compare searches using our novel library to searches with existing in-silico lipidomics libraries.: "we introduce a novel in-silico lipidomics library covering major lipid classes and compare searches using our novel library to searches with existing in-silico lipidomics libraries."
- [readme] If the fragmentation does not make sense for the given chain combination and structural composition of the lipid, an error will display.: [evidence span withheld — anchor status: external_doc]
