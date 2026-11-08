---
name: cross-library-coverage-comparison
description: Use when you have MS/MS search results from the same features against
  multiple in-silico lipidomics libraries and need to quantify which lipid classes
  or adducts each library covers, or which classes are unique to a single library.
when_to_use_negative:
- Input is a single-library search result — there is no second library to cross-compare
  coverage against
- Goal is scoring individual feature identifications rather than characterizing whole-library
  class coverage (use RBP/hypergeometric score thresholding workflows instead)
- Data are metabolomics (non-lipid) spectral matches; the class/adduct enumeration
  schema described here is lipid-class-specific
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: MAVEN2
  role: Runs the MS/MS spectral matching searches of lipidomics features against each
    in-silico library whose coverage is being compared
  repo: https://github.com/eugenemel/maven
- name: Maven GUI
  role: Interface for loading mzML samples and reference libraries and exporting the
    per-library search outputs used in the comparison
  repo: https://github.com/eugenemel/maven
- name: CalicoLipidLibrary
  role: Generates the in-silico CalicoLipids MSP library (per class and ionization
    mode) whose class coverage is enumerated against MS-DIAL
  repo: https://github.com/calico/CalicoLipidLibrary
provenance:
  source_task_ids:
  - task_005
  - task_002
  - task_004
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:ac2eb97d41626c89d8ee7cbe889ca23561cef1968ba5b9e4cf024fc7714abc66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - CalicoLipidLibrary
  - MAVEN2
  - Maven GUI
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cross-library-coverage-comparison

## Summary

Systematically compares the per-lipid-class and per-adduct coverage of two or more in-silico MS/MS spectral libraries (e.g., CalicoLipids vs MS-DIAL) as searched in MAVEN2, building a class-by-library count matrix of library entries and annotated features and flagging classes represented in only one library.

## When to use

Use when you have MS/MS search results from the same features against multiple in-silico lipidomics libraries and need to quantify which lipid classes or adducts each library covers, or which classes are unique to a single library.

## When NOT to use

- Input is a single-library search result — there is no second library to cross-compare coverage against
- Goal is scoring individual feature identifications rather than characterizing whole-library class coverage (use RBP/hypergeometric score thresholding workflows instead)
- Data are metabolomics (non-lipid) spectral matches; the class/adduct enumeration schema described here is lipid-class-specific

## Inputs

- Thresholded per-feature annotation table from MAVEN2 searches of Thermo lipidomics mzML files against CalicoLipids and MS-DIAL libraries
- Library composition statistics (per-class, per-adduct library entry counts, e.g. MSP spectra per class)
- RBP (Retention Balance Point) score-thresholded dual-library annotation results for the 12,054 detected features

## Outputs

- Per-class, per-library, per-adduct count matrix of library entries and annotated features (CSV, e.g. dataset_s5_lipid_class_counts.csv)
- List of lipid classes represented in only one library (e.g. OxPC only in MS-DIAL, CE only in CalicoLipids)

## How to apply

Load the thresholded annotation table and library composition statistics exported from the MAVEN2 dual-library searches (CalicoLipids and MS-DIAL). Group records by lipid class and adduct form, then count library entries (e.g., SM spectra 14,076 vs 117 across libraries) and annotated features (e.g., Ceramide 36,288 vs 39,882) per class/adduct/library to build a count matrix. Cross-compare columns of the matrix and flag classes present in only one library — in this dataset, OxPCs appear only in MS-DIAL while CEs appear only in CalicoLipids. Optionally enrich the comparison with the RBP-score-thresholded annotation agreement statistics (2,612/12,054 features annotated; 1,142 shared; ~94% class-level and ~87% full-structure agreement). Write the count matrix and the single-library class list to a structured CSV such as dataset_s5_lipid_class_counts.csv.

## Related tools

- **MAVEN2** (Runs the MS/MS spectral matching searches of lipidomics features against each in-silico library whose coverage is being compared) — https://github.com/eugenemel/maven
- **Maven GUI** (Interface for loading mzML samples and reference libraries and exporting the per-library search outputs used in the comparison) — https://github.com/eugenemel/maven
- **CalicoLipidLibrary** (Generates the in-silico CalicoLipids MSP library (per class and ionization mode) whose class coverage is enumerated against MS-DIAL) — https://github.com/calico/CalicoLipidLibrary

## Examples

```
python2.7 generateDB.py -m "pos" -c "PC,PE,PS"  # regenerate CalicoLipids class subset MSP, then enumerate per-class entries vs MS-DIAL before searching in MAVEN2
```

## Evaluation signals

- Per-class counts in the output matrix match the reported values (e.g., SM library entries 14,076 vs 117; Ceramide annotated features 36,288 vs 39,882)
- Single-library class list reproduces the known result: OxPC present only in MS-DIAL, CE present only in CalicoLipids
- Output CSV parses, and every lipid class/adduct/library combination has an explicit (possibly zero) count
- Downstream agreement statistics are consistent with the RBP-thresholded partition (1,009 CalicoLipids-only + 461 MS-DIAL-only + 1,142 shared = 2,612 of 12,054 features; 94% class-level, 87% full-structure agreement among shared)
- Each library entry count is non-negative and does not exceed the total spectra in the corresponding generated MSP file

## Limitations

- Coverage counts depend on the lipid classes enumerated when generating the library MSP; subsets selected via the -c option (e.g., only PC, PE, PS) will under-represent classes intentionally excluded
- Class- or adduct-specific gaps (e.g., no OxPC in CalicoLipids, no CE in MS-DIAL) mean one library cannot be used to validate or cross-check annotations from the other for those classes
- Fragmentation enumeration for large classes can be time-consuming, so partial libraries may not reflect full theoretical coverage

## Evidence

- [abstract] Novel in-silico lipidomics library compared to existing libraries: "we introduce a novel in-silico lipidomics library covering major lipid classes and compare searches using our novel library to searches with existing in-silico lipidomics libraries."
- [abstract] Algorithms for MS/MS spectral matching and library search: "We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries."
- [readme] CalicoLipidLibrary purpose and class enumeration: [evidence span withheld — anchor status: external_doc]
- [readme] Subset library generation by lipid class: [evidence span withheld — anchor status: external_doc]
