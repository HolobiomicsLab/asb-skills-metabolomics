---
name: metabolomics-study-data-acquisition
description: Use when a public metabolomics study's raw LC-MS/MS files must be fetched
  (e.g. Metabolomics WorkBench ST002044) and converted into deduplicated MS/MS .msp
  spectra per ESI+ / ESI− mode for downstream modeling or library matching.
when_to_use_negative:
- Input is already a processed feature table or an existing .msp/.mgf spectral library
  — no raw-data peak-picking or deconvolution is needed.
- The target study has no publicly deposited raw LC-MS/MS files (e.g. only summary
  annotations are available) — IDSL.IPA/IDSL.CSA cannot be run without raw scans.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: IDSL.IPA
  role: Chromatographic peak-picking on raw LC-MS/MS data to detect MS1 features and
    associated MS/MS scans
- name: IDSL.CSA
  role: R package used to deconvolute MS/MS spectra and extract unique spectra per
    ionization mode into .msp files (3,386 ESI+ and 1,901 ESI− for ST002044)
- name: IDSL_MINT
  role: Downstream consumer of the generated .msp files; defines MSP formatting requirements
    (PrecursorMZ rows, SMILES/InChI parsing) the exported spectra must satisfy
  repo: https://github.com/idslme/IDSL_MINT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:e0d83cf40f640472afd4ed613cb47024855bbbfe79be559518306ae70b452d6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL.CSA
  - IDSL.IPA
  - IDSL_MINT
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# metabolomics-study-data-acquisition

## Summary

Acquire raw LC-MS/MS data for a publicly available untargeted metabolomics study (e.g. Metabolomics WorkBench study ST002044) and convert it into unique deconvolved MS/MS spectra in NIST .msp format, separately per ionization mode, using IDSL.IPA peak-picking and IDSL.CSA spectral deconvolution.

## When to use

Use when a public metabolomics study's raw LC-MS/MS files must be fetched (e.g. Metabolomics WorkBench ST002044) and converted into deduplicated MS/MS .msp spectra per ESI+ / ESI− mode for downstream modeling or library matching.

## When NOT to use

- Input is already a processed feature table or an existing .msp/.mgf spectral library — no raw-data peak-picking or deconvolution is needed.
- The target study has no publicly deposited raw LC-MS/MS files (e.g. only summary annotations are available) — IDSL.IPA/IDSL.CSA cannot be run without raw scans.

## Inputs

- Raw LC-MS/MS data files of Metabolomics WorkBench study ST002044
- Ionization-mode separation of the raw files (ESI+ and ESI− runs)

## Outputs

- Unique deconvolved MS/MS spectra in NIST .msp format for ESI+ (3,386 spectra; Zenodo entry [27] file#1 csa_spreadsheet.zip)
- Unique deconvolved MS/MS spectra in NIST .msp format for ESI− (1,901 spectra)
- MSP blocks with required PrecursorMZ entries (plus Name, InChI/SMILES, Formula, Num Peaks fields when available)

## How to apply

Download the raw LC-MS/MS data files for the target study (ST002044 from the Metabolomics WorkBench). Run IDSL.IPA chromatographic peak-picking on the raw data to detect MS1 features and their associated MS/MS scans. Then run the IDSL.CSA R package to deconvolute MS/MS spectra, associating fragment spectra with precursor features, separately for ESI+ and ESI− modes. Deduplicate the deconvolved spectra within each ionization mode and export the unique MS/MS spectra in NIST .msp format, ensuring each MSP block contains a required PrecursorMZ row entry (IDSL_MINT's parser also reads comment/comments, SMILES and InChI labels case-insensitively). Validate by counting the spectrum blocks: ST002044 yields 3,386 unique ESI+ spectra and 1,901 unique ESI− spectra.

## Related tools

- **IDSL.IPA** (Chromatographic peak-picking on raw LC-MS/MS data to detect MS1 features and associated MS/MS scans)
- **IDSL.CSA** (R package used to deconvolute MS/MS spectra and extract unique spectra per ionization mode into .msp files (3,386 ESI+ and 1,901 ESI− for ST002044))
- **IDSL_MINT** (Downstream consumer of the generated .msp files; defines MSP formatting requirements (PrecursorMZ rows, SMILES/InChI parsing) the exported spectra must satisfy) — https://github.com/idslme/IDSL_MINT

## Evaluation signals

- Count MS/MS spectrum blocks in each output .msp file: the ESI+ file must contain 3,386 spectra and the ESI− file 1,901, matching the reported values for ST002044.
- Every MSP block contains a PrecursorMZ row entry (mandatory for downstream IDSL_MINT parsing).
- Files are split per ionization mode (ESI+ and ESI− processed and exported separately).
- Spectra are deduplicated within each ionization mode — no repeated identical spectrum blocks in a single .msp output.

## Limitations

- Spectral counts are study-specific: 3,386/1,901 applies to ST002044 only; other studies will yield different numbers of unique spectra.
- Library-match coverage is limited even after acquisition: matching against NIST 20 and LipidBlast annotated only 638/3,386 ESI+ and 299/1,901 ESI− spectra.
- MSP parsing is format-sensitive: with multiple similar headers in a block, IDSL_MINT selects the one with the longest content, so malformed or ambiguous headers can change what is parsed.

## Evidence

- [results] IDSL.CSA extracted 3,386 and 1,901 unique MS/MS spectra in ESI+ and ESI− for study ST002044: "The IDSL.CSA R package [2] was used to extract 3,386 and 1,901 unique MS/MS spectra in ESI+ and ESI− for this study"
- [results] Raw data processed to .msp evaluation set: "First, we processed the raw data for this study to extract MS/MS spectra in the NIST.msp format that can be used as the model evaluation set."
- [results] Study sourced from Metabolomics WorkBench: "We showcase the application of IDSL_MINT for a publicly available untargeted metabolomics study (ST002044) from the Metabolomics WorkBench database."
- [readme] MSP format requirement for downstream use: "MSP blocks must include `PrecursorMZ: ` row entries."
- [readme] MSP header parsing behavior: "In case multiple similar headers are present in a MSP block, the one with the longest content is selected for parsing."
