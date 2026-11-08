---
name: lipid-class-adduct-enumeration
description: Use when building a reference in-silico lipid fragmentation library for
  LC-MS/MS spectral matching, or when verifying locally generated msp lipid libraries
  against the deposited MTBLS3097 reference.
when_to_use_negative:
- The goal is matching experimental spectra against already-existing libraries rather
  than generating new reference libraries
- Target lipid classes or adduct forms are outside the 61 supported classes / valid
  adduct list (unsupported entries such as 'ImaginaryClass' are silently removed,
  and invalid fragmentation raises an error)
- Input is experimental LC-MS/MS data needing annotation — that downstream search
  belongs to MAVEN2 spectral matching, not library generation
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: CalicoLipidLibrary
  role: Python package whose generateDB.py script enumerates theoretical lipid fragmentation
    spectra and writes msp library files
  repo: https://github.com/calico/CalicoLipidLibrary
- name: MAVEN2
  role: Consumes the generated msp in-silico lipidomics libraries for MS/MS spectral
    matching and efficient search of large-scale fragmentation libraries
  repo: https://github.com/eugenemel/maven
- name: git
  role: Clone the CalicoLipidLibrary and maven repositories
  repo: https://github.com/eugenemel/maven
- name: make
  role: Build MAVEN2 from source after cloning (qmake -r build.pro; make -j4) if compiling
    the search tool
  repo: https://github.com/eugenemel/maven
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:8c16c1101d8c611d4e0c32c6f1f3c27e1a496f16a591b6d1eefad4a2f19cfc9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/calico/CalicoLipidLibrary
  tools:
  - CalicoLipidLibrary
  - MAVEN2
  - git
  - make
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# lipid-class-adduct-enumeration

## Summary

Enumerate rule-based in-silico lipid MS/MS fragmentation spectra across all supported lipid classes and adduct forms using the CalicoLipidLibrary package, producing msp-format spectral libraries (deposited as MTBLS3097) for MS/MS spectral matching in MAVEN2.

## When to use

Use when building a reference in-silico lipid fragmentation library for LC-MS/MS spectral matching, or when verifying locally generated msp lipid libraries against the deposited MTBLS3097 reference.

## When NOT to use

- The goal is matching experimental spectra against already-existing libraries rather than generating new reference libraries
- Target lipid classes or adduct forms are outside the 61 supported classes / valid adduct list (unsupported entries such as 'ImaginaryClass' are silently removed, and invalid fragmentation raises an error)
- Input is experimental LC-MS/MS data needing annotation — that downstream search belongs to MAVEN2 spectral matching, not library generation

## Inputs

- Lipid class list (comma-delimited) selected from the 61 supported classes (e.g. PC, PE, PS, TG, CL, Ceramide, Sulfatide)
- Ionization mode flag: 'pos' or 'neg'
- Acyl chain definitions as [num_carbons, num_double_bonds, num_oxidations] tuples for single-spectrum queries
- CalicoLipidLibrary repository code (python2.7; pypy optional for speed)

## Outputs

- msp-format in-silico lipid spectral library files (e.g. <date>-Calico-Lipids-<all|<classes>>-<ion>.msp) usable for MS/MS spectral matching
- Printed theoretical MS/MS product-ion spectra for individual lipid class/adduct combinations

## How to apply

Clone calico/CalicoLipidLibrary and run generateDB.py with an ionization-mode flag (-m pos|neg), optionally restricting classes via the comma-delimited -c option (61 classes are available, e.g. PC, PE, TG, CL, HexCer, sulfatides) and setting output name/path with -n and -o. The generator applies a rule-based HCD-mimicking fragmentation model using stepped normalized collision energies of 20, 30, and 40 to compute product-ion spectra for every enumerated lipid-adduct combination (adducts include [M+H]+, [M+Na]+, [M+NH4]+, [M-H]-, [M+FA-H]-, [M-2H]2-, etc., though not every adduct is valid for every class). Output is serialized to msp-format files compatible with MS/MS spectral matching search in MAVEN2. For single-spectrum inspection, use calicolipidlibrary.print_spectrum(class, acyl_chains, adduct) in an interactive python2.7 session; chains are specified as [num_carbons, num_double_bonds, num_oxidations]. Validate by parsing the msp in NIST format, confirming coverage of all intended classes/adducts, and comparing sampled records against the MTBLS3097 deposit.

## Related tools

- **CalicoLipidLibrary** (Python package whose generateDB.py script enumerates theoretical lipid fragmentation spectra and writes msp library files) — https://github.com/calico/CalicoLipidLibrary
- **MAVEN2** (Consumes the generated msp in-silico lipidomics libraries for MS/MS spectral matching and efficient search of large-scale fragmentation libraries) — https://github.com/eugenemel/maven
- **git** (Clone the CalicoLipidLibrary and maven repositories) — https://github.com/eugenemel/maven
- **make** (Build MAVEN2 from source after cloning (qmake -r build.pro; make -j4) if compiling the search tool) — https://github.com/eugenemel/maven

## Examples

```
python2.7 generateDB.py -m "pos" -c "PC,PE,PS" -o /path/to/output -n "glycerophospholipids_pos.msp"
```

## Evaluation signals

- Generated msp files parse cleanly under the NIST msp format
- Library contains entries covering all 61 supported lipid classes and the intended adduct forms (not all adducts are valid for all classes)
- A sampled set of generated records matches corresponding msp entries in the MetaboLights MTBLS3097 deposit (output_matches_reference on representative spectra)
- Single-spectrum spot checks via calicolipidlibrary.print_spectrum() succeed without the 'fragmentation does not make sense' error for valid class/chain/adduct combinations
- pytest tests pass (pip install pytest; pytest tests), optionally accelerated with pypy

## Limitations

- Enumeration over all lipid classes is time-consuming; the -c option should be used to subset classes of interest, and pypy is recommended for significantly faster speed
- Only Python 2 is supported for the interactive print_spectrum workflow (python2.7)
- Not all adduct names are valid for all lipid classes, and unsupported classes in a comma-delimited list are silently removed (e.g. 'ImaginaryClass'), so class lists must be checked
- The p- forms of Alkyl_PC and Alkyl_PE imply an additional double bond, and oxidations supplied to sphingolipids add an extra OH from the hydroxyl head group — easy to mis-specify

## Evidence

- [readme] Scripts to create in silico lipid fragmentation spectra, based on analysis of chemical standards and relevant scientific literature.: [evidence span withheld — anchor status: external_doc]
- [readme] Enumerating all theoretical spectra over all lipid classes may be time-consuming, so the -c option may be used to generate a subset of the library: "Enumerating all theoretical spectra over all lipid classes may be time-consuming, so the `-c` option may be used
to generate a subset of the library for a set of classes of interest."
- [readme] Not all adduct names are valid for all lipid classes.: [evidence span withheld — anchor status: external_doc]
- [abstract] We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries.: "We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries."
- [abstract] we introduce a novel in-silico lipidomics library covering major lipid classes and compare searches using our novel library to searches with existing in-silico lipidomics libraries.: "we introduce a novel in-silico lipidomics library covering major lipid classes and compare searches using our novel library to searches with existing in-silico lipidomics libraries."
- [readme] python2.7 generateDB.py -m "neg" -c "AcGM2, AcGM3, GcGM2, GcGM3, MIP2C, MIPC, HexCer, LacCer, Sulfatide, SM, Ceramide, Ceramide_P" -n "neg_sphingolipids": [evidence span withheld — anchor status: external_doc]
