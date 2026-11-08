---
name: nist-msp-spectral-export
description: Use when raw LC-MS/MS data (e.g., from an untargeted metabolomics study
  like ST002044 on Metabolomics WorkBench) must be converted into unique, deconvoluted
  MS/MS spectra in .msp format, split by ESI+ and ESI− modes.
when_to_use_negative:
- The MS/MS spectra already exist in valid .msp format with PrecursorMZ entries —
  consume them directly instead of re-running peak-picking and deconvolution
- The analysis only needs MS1 feature detection without MS/MS fragmentation spectra
  (IDSL.IPA peak-picking alone suffices)
- The input is DIA-like data without a reliable precursor-to-fragment association,
  since IDSL.CSA deconvolution depends on linking fragment spectra to precursor features
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3292
tools:
- name: IDSL.IPA
  role: Chromatographic peak-picking on raw LC-MS/MS data to detect MS1 features and
    their associated MS/MS scans before deconvolution
- name: IDSL.CSA
  role: R package that deconvolutes MS/MS spectra by associating fragment spectra
    with precursor features and exports unique spectra in NIST .msp format
- name: IDSL_MINT
  role: Downstream consumer of the exported .msp files; requires PrecursorMZ row entries
    and parses comment/comments and SMILES/InChI labels
  repo: https://github.com/idslme/IDSL_MINT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:a0b0a764b02775e017ce395921231c71fe94e17512e6e4e364548f2924378af1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL.CSA
  - IDSL.IPA
  - IDSL_MINT
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# nist-msp-spectral-export

## Summary

Extracts and deduplicates deconvoluted MS/MS spectra from LC-MS/MS raw data and exports them as unique spectrum blocks in NIST .msp format, per ionization mode, so they can serve as evaluation sets or inputs for downstream tools such as IDSL_MINT, NIST MS Search, or spectral library searches.

## When to use

Use when raw LC-MS/MS data (e.g., from an untargeted metabolomics study like ST002044 on Metabolomics WorkBench) must be converted into unique, deconvoluted MS/MS spectra in .msp format, split by ESI+ and ESI− modes.

## When NOT to use

- The MS/MS spectra already exist in valid .msp format with PrecursorMZ entries — consume them directly instead of re-running peak-picking and deconvolution
- The analysis only needs MS1 feature detection without MS/MS fragmentation spectra (IDSL.IPA peak-picking alone suffices)
- The input is DIA-like data without a reliable precursor-to-fragment association, since IDSL.CSA deconvolution depends on linking fragment spectra to precursor features

## Inputs

- Raw LC-MS/MS data files from an untargeted metabolomics study (e.g., Metabolomics WorkBench study ST002044)
- MS1 feature tables / associated MS/MS scan assignments from IDSL.IPA peak-picking
- Ionization mode designation (ESI+ and ESI− processed separately)

## Outputs

- Deduplicated MS/MS spectra exported in NIST .msp format, one file per ionization mode (for ST002044: 3,386 ESI+ and 1,901 ESI− spectra)
- MSP blocks containing required `PrecursorMZ: ` entries and optionally Name, Synon, InChI/SMILES, Precursor_type, Spectrum_type, Instrument_type, Ion_mode, Collision_energy, Formula, ExactMass, Num Peaks, and peak m/z–intensity pairs

## How to apply

Run IDSL.IPA chromatographic peak-picking on the raw LC-MS/MS files to detect MS1 features and their associated MS/MS scans, then run the IDSL.CSA R package to deconvolute the MS/MS spectra by associating fragment spectra with precursor features, processing ESI+ and ESI− separately. Deduplicate the deconvoluted spectra within each ionization mode and export the unique spectra as NIST .msp blocks; each block must include a `PrecursorMZ: ` row entry, and IDSL_MINT downstream can additionally parse `comment: `/`comments: ` entries and `SMILES: `/`InChI: ` labels (case-insensitive). When duplicate header labels appear in an MSP block, the parser selects the one with the longest content, so keep headers unambiguous when authoring. For the reference study ST002044 this pipeline yielded 3,386 unique spectra in ESI+ and 1,901 in ESI−, deposited as .msp files (e.g., Zenodo entry [27], file#1 csa_spreadsheet.zip).

## Related tools

- **IDSL.IPA** (Chromatographic peak-picking on raw LC-MS/MS data to detect MS1 features and their associated MS/MS scans before deconvolution)
- **IDSL.CSA** (R package that deconvolutes MS/MS spectra by associating fragment spectra with precursor features and exports unique spectra in NIST .msp format)
- **IDSL_MINT** (Downstream consumer of the exported .msp files; requires PrecursorMZ row entries and parses comment/comments and SMILES/InChI labels) — https://github.com/idslme/IDSL_MINT

## Evaluation signals

- Count the unique MS/MS spectrum blocks in each output .msp file and verify counts match reported values (ESI+ = 3,386; ESI− = 1,901 for ST002044)
- Every MSP block contains a `PrecursorMZ: ` row entry (mandatory per IDSL_MINT parsing requirements)
- ESI+ and ESI− spectra were exported to separate .msp files with no cross-mode mixing
- Within each ionization mode, exported spectra are unique (de-duplicated), matching the 'unique MS/MS spectra' definition
- Peak lists inside each block conform to m/z–intensity pairs with a Num Peaks header consistent with the number of listed peaks

## Limitations

- MSP blocks lacking `PrecursorMZ: ` entries cannot be parsed by IDSL_MINT, so exported files missing this field fail downstream
- When multiple similar headers occur in one MSP block, IDSL_MINT selects the one with the longest content, which can cause mis-parsing if headers are inconsistently authored
- Coverage depends on the spectral library used afterwards; in the ST002044 case study, IDSL.CSA library matching against NIST 20 and LipidBlast annotated only 638/3,386 (ESI+) and 299/1,901 (ESI−) spectra, so most exported spectra remain unannotated

## Evidence

- [results] IDSL.CSA extracted 3,386 and 1,901 unique MS/MS spectra in ESI+ and ESI− for study ST002044: "The IDSL.CSA R package [2] was used to extract 3,386 and 1,901 unique MS/MS spectra in ESI+ and ESI− for this study"
- [readme] MSP blocks require PrecursorMZ entries: "MSP blocks must include `PrecursorMZ: ` row entries."
- [results] Raw data processed to NIST .msp format as evaluation set: "First, we processed the raw data for this study to extract MS/MS spectra in the NIST.msp format that can be used as the model evaluation set."
- [readme] IDSL_MINT parses comment entries and SMILES/InChI labels from MSP files: "**IDSL_MINT** can extract information from `comment: ` and `comments: ` entries in ***.msp*** files"
- [readme] Duplicate header handling rule in MSP parsing: "In case multiple similar headers are present in a MSP block, the one with the longest content is selected for parsing."
