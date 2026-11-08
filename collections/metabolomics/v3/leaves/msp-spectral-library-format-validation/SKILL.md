---
name: msp-spectral-library-format-validation
description: Use when you have generated in-silico lipid fragmentation spectra in
  msp format via CalicoLipidLibrary and must verify parseability, coverage of all
  supported lipid classes/adducts, and agreement with the MTBLS3097 reference deposit.
when_to_use_negative:
- Input is already an experimental MS/MS peak list or a curated empirical library
  rather than in-silico generated lipid spectra
- Target search engine does not consume NIST msp-format libraries
- Lipid classes or adduct forms are outside the set supported by CalicoLipidLibrary
  (unsupported classes like ImaginaryClass are silently removed)
edam_operation: http://edamontology.org/operation_0321
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: CalicoLipidLibrary
  role: Generates the in-silico lipid fragmentation msp libraries whose format and
    coverage are validated
  repo: https://github.com/calico/CalicoLipidLibrary
- name: MAVEN2
  role: Downstream consumer of the msp libraries; provides algorithms for MS/MS spectral
    matching and efficient search of large-scale fragmentation libraries
  repo: https://github.com/eugenemel/maven
- name: git
  role: Used to clone the repository recursively to retrieve generator scripts
  repo: https://github.com/eugenemel/maven
- name: make
  role: Build step used to compile MAVEN/MAVEN2 from the cloned repository
  repo: https://github.com/eugenemel/maven
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:237aefb4632399f37d9f580b02922c8e883a392d03218e484d4bb17b1054bd2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calico/CalicoLipidLibrary
  tools:
  - CalicoLipidLibrary
  - MAVEN2
  - git
  - make
  edam_operation: http://edamontology.org/operation_0321
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# msp-spectral-library-format-validation

## Summary

Validates locally generated in-silico lipid msp-format spectral library files (from CalicoLipidLibrary) against the reference deposit at MetaboLights MTBLS3097, confirming format compatibility with MS/MS spectral matching search and completeness of lipid-class and adduct coverage.

## When to use

Use when you have generated in-silico lipid fragmentation spectra in msp format via CalicoLipidLibrary and must verify parseability, coverage of all supported lipid classes/adducts, and agreement with the MTBLS3097 reference deposit.

## When NOT to use

- Input is already an experimental MS/MS peak list or a curated empirical library rather than in-silico generated lipid spectra
- Target search engine does not consume NIST msp-format libraries
- Lipid classes or adduct forms are outside the set supported by CalicoLipidLibrary (unsupported classes like ImaginaryClass are silently removed)

## Inputs

- CalicoLipidLibrary code repository (calico/CalicoLipidLibrary)
- Generated msp-format lipid library files (pos or neg ionization mode)
- Reference msp library files from MetaboLights accession MTBLS3097

## Outputs

- Validated NIST-parseable msp spectral library files covering all 61 lipid classes and 15 adduct forms
- Match/mismatch report comparing sampled generated spectra to MTBLS3097 reference entries

## How to apply

Retrieve the CalicoLipidLibrary code repository (calico/CalicoLipidLibrary) and the deposited msp library files from MetaboLights accession MTBLS3097. Generate the library with generateDB.py, choosing the ionization mode with -m (pos|neg), optionally restricting lipid classes with the comma-delimited -c option, and setting the output path and file name with -o and -n. Enumerate spectra across the supported lipid classes and the supported adduct forms (e.g. [M+H]+, [M+Na]+, [M+NH4]+, [M+K]+, [M+2H]2+, [M+Li]+, [M-H]-, [M+Cl]-, [M+FA-H]-, [M-CH3]-, [M+AcOH-H]-, [M-2H]2-, [M-3H]3-), applying the HCD-mimicking fragmentation model with stepped normalized collision energies of 20, 30, and 40. Validate that the resulting msp files parse in NIST msp format, contain entries covering all expected classes and adducts, and that a sampled set of records output_matches_reference against the corresponding MTBLS3097 entries. A single class/adduct spectrum can be spot-checked interactively via calicolipidlibrary.print_spectrum(<class>, <chains>, <adduct>) in Python 2.

## Related tools

- **CalicoLipidLibrary** (Generates the in-silico lipid fragmentation msp libraries whose format and coverage are validated) — https://github.com/calico/CalicoLipidLibrary
- **MAVEN2** (Downstream consumer of the msp libraries; provides algorithms for MS/MS spectral matching and efficient search of large-scale fragmentation libraries) — https://github.com/eugenemel/maven
- **git** (Used to clone the repository recursively to retrieve generator scripts) — https://github.com/eugenemel/maven
- **make** (Build step used to compile MAVEN/MAVEN2 from the cloned repository) — https://github.com/eugenemel/maven

## Examples

```
python2.7 generateDB.py -m "pos" -c "PC,PE,PS" -n "glycerophospholipids_pos.msp"
```

## Evaluation signals

- Generated msp files parse without error under NIST msp format
- Library entries cover all 61 available lipid classes and 15 adduct forms defined by the generation rules
- A sampled set of generated records matches the corresponding MTBLS3097 msp entries (output_matches_reference check on representative spectra)
- pytest tests in the CalicoLipidLibrary repository pass (pytest tests, optionally under pypy)
- Spot-check a single class/adduct spectrum with calicolipidlibrary.print_spectrum against the expected theoretical product ions

## Limitations

- Currently only Python 2 is supported for the interactive single-spectrum printing workflow
- Enumerating all theoretical spectra over all lipid classes may be time-consuming; pypy is recommended for significantly faster speed, and the -c option can subset by class
- Not all adduct names are valid for all lipid classes; invalid class/adduct combinations raise an error or are silently removed
- Changelog-type gaps aside, MAKEN/CalicoLipidLibrary-generated libraries must be re-verified against MTBLS3097 after any regeneration, since reproducibility of the deposit is the validation anchor

## Evidence

- [readme] Scripts to create in silico lipid fragmentation spectra, based on analysis of chemical standards and relevant scientific literature.: [evidence span withheld — anchor status: external_doc]
- [readme] python2.7 generateDB.py -m "neg": "# all negative mode lipids
python2.7 generateDB.py -m "neg""
- [readme] Enumerating all theoretical spectra over all lipid classes may be time-consuming, so the `-c` option may be used to generate a subset of the library: [evidence span withheld — anchor status: external_doc]
- [abstract] We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries.: "We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries."
- [other] The package's scripts produce msp-format spectral libraries, and the generated library files were deposited to MetaboLights under accession number MTBLS3097: [evidence span withheld — anchor status: not_found]
- [other] the generated library files were deposited to MetaboLights, providing the reference against which locally generated msp output can be verified: [evidence span withheld — anchor status: not_found]
