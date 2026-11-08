---
name: lipidomics-msms-spectral-matching
description: Use when in-house Thermo lipidomics mzML files need per-feature MS/MS-based
  lipid annotations via library spectral matching, or when comparing alternative in-silico
  lipid libraries.
when_to_use_negative:
- Input is already a feature-level annotation table (no raw MS/MS spectra to search)
- The task is general metabolomics ID where no lipid-focused in-silico fragmentation
  library is appropriate
- Only a single lipid class is of interest and library enumeration cost outweighs
  benefit (use CalicoLipidLibrary's -c subset option instead of a full-library search)
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: MAVEN2
  role: Performs the MS/MS spectral matching search of lipidomics features against
    in-silico fragmentation libraries and assigns hypergeometric match scores
  repo: https://github.com/eugenemel/maven
- name: Maven GUI
  role: Graphical interface used to load mzML samples and reference libraries and
    run the searches
  repo: https://github.com/eugenemel/maven
- name: CalicoLipidLibrary
  role: Generates the CalicoLipids in-silico lipid fragmentation spectra (.msp) used
    as a reference library
  repo: https://github.com/calico/CalicoLipidLibrary
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:4bab27a136fdfab99d9774b25ecb4cbb73d29edc3f850da14ebb3ebecb1ba0ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - CalicoLipidLibrary
  - MAVEN2
  - Maven GUI
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# lipidomics-msms-spectral-matching

## Summary

Search LC-MS/MS lipidomics features in MAVEN2 against in-silico spectral libraries (CalicoLipids and MS-DIAL) and score spectral matches with the hypergeometric score, yielding per-feature lipid annotations.

## When to use

Use when in-house Thermo lipidomics mzML files need per-feature MS/MS-based lipid annotations via library spectral matching, or when comparing alternative in-silico lipid libraries.

## When NOT to use

- Input is already a feature-level annotation table (no raw MS/MS spectra to search)
- The task is general metabolomics ID where no lipid-focused in-silico fragmentation library is appropriate
- Only a single lipid class is of interest and library enumeration cost outweighs benefit (use CalicoLipidLibrary's -c subset option instead of a full-library search)

## Inputs

- Thermo lipidomics mzML files (LC-MS/MS samples)
- CalicoLipids in-silico lipidomics library (.msp, generated via CalicoLipidLibrary)
- MS-DIAL in-silico lipid library (alternate reference library)

## Outputs

- CSV table of per-feature hypergeometric scores and lipid annotations from the CalicoLipids search
- CSV table of per-feature hypergeometric scores and lipid annotations from the MS-DIAL search

## How to apply

Load the lipidomics mzML files into the MAVEN2 GUI as search samples, then load the CalicoLipids in-silico library (generated from calico/CalicoLipidLibrary, e.g. generateDB.py producing an .msp file in the desired ionization mode) as the reference fragmentation library. Run the MAVEN2 MS/MS spectral matching search, which uses algorithms developed to support spectral matching and efficient search of large-scale fragmentation libraries, and record the per-feature hypergeometric score and best lipid annotation for each feature. Repeat the identical search against the MS-DIAL in-silico lipid library as the alternate reference, then export each search as a structured CSV table retaining feature identifiers, hypergeometric scores, and proposed annotations for downstream thresholding. Validate that each CSV parses and contains a score and annotation entry for every searched feature from both library searches.

## Related tools

- **MAVEN2** (Performs the MS/MS spectral matching search of lipidomics features against in-silico fragmentation libraries and assigns hypergeometric match scores) — https://github.com/eugenemel/maven
- **Maven GUI** (Graphical interface used to load mzML samples and reference libraries and run the searches) — https://github.com/eugenemel/maven
- **CalicoLipidLibrary** (Generates the CalicoLipids in-silico lipid fragmentation spectra (.msp) used as a reference library) — https://github.com/calico/CalicoLipidLibrary

## Examples

```
python2.7 generateDB.py -m "neg" -c "PC,PE,PS" -n "neg_lipids.msp"
```

## Evaluation signals

- Each exported table exists and parses as valid CSV
- Every searched feature has both a hypergeometric score and a lipid annotation entry in each library's table
- Two tables are produced, one per reference library (CalicoLipids and MS-DIAL)
- Hypergeometric scores fall in the valid score range and proposed annotations correspond to lipid classes present in the loaded libraries

## Limitations

- Spectral matching cannot fully separate authentic from spurious identifications; score thresholding is required and should be validated (e.g. with standards spiked into water and yeast backgrounds)
- Enumerating all theoretical spectra over all lipid classes in CalicoLipidLibrary can be time-consuming; use the -c option to subset classes
- CalicoLipidLibrary generation is Python 2 based (python2.7 generateDB.py); pypy recommended for speed
- As of 2024-11-05 Linux builds of Maven have been retired; only macOS and Windows executables are produced by Appveyor

## Evidence

- [abstract] MAVEN2 supports LC-MS/MS spectral matching against in-silico lipidomics libraries: "This manuscript describes a major update to the program, MAVEN2, which supports LC-MS/MS analysis of metabolomics and lipidomics samples."
- [full_text] Hypergeometric score used for spectral match quality: [evidence span withheld — anchor status: not_found]
- [abstract] Novel in-silico lipidomics library introduced and compared: "we introduce a novel in-silico lipidomics library covering major lipid classes and compare searches using our novel library to searches with existing in-silico lipidomics libraries."
- [readme] CalicoLipidLibrary generates in-silico lipid fragmentation spectra as .msp files: [evidence span withheld — anchor status: external_doc]
- [readme] Library generation command for MSP output: [evidence span withheld — anchor status: external_doc]
