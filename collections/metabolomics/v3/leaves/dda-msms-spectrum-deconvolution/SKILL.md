---
name: dda-msms-spectrum-deconvolution
description: Use when raw DDA LC-MS/MS data (e.g. a Metabolomics WorkBench study such
  as ST002044) must be converted into deconvolved, unique MS/MS spectra in NIST .msp
  format for library matching or model training/evaluation.
when_to_use_negative:
- Input is already a deconvolved .msp/.mgf library or a feature table — no peak-picking/deconvolution
  is needed
- Data are DIA/SWATH or otherwise lack precursor-isolated MS2 scans, since IDSL.CSA
  associates fragment spectra with precursor MS1 features
- Goal is MS/MS prediction or fingerprint inference rather than spectrum extraction
  — use IDSL_MINT training/prediction instead
edam_operation: http://edamontology.org/operation_3646
tools:
- name: IDSL.IPA
  role: Chromatographic peak-picking on raw LC-MS/MS data to detect MS1 features and
    their associated MS/MS scans prior to deconvolution
- name: IDSL.CSA
  role: R package that deconvolves MS/MS spectra by associating fragment spectra with
    precursor features and exports unique spectra in NIST .msp format
- name: IDSL_MINT
  role: Downstream consumer of the deconvolved .msp files; requires MSP blocks with
    PrecursorMZ entries and parses SMILES/InChI and comment fields
  repo: https://github.com/idslme/IDSL_MINT
- name: RDKit
  role: Converts InChI and SMILES row entries in MSP blocks into canonical SMILES
    for fingerprint computation
  repo: https://www.rdkit.org/docs/Install.html
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:7f306f541f3febd708033c14f96c7b8d9f877ec8680ce0175423d5afd4d7d25d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL.CSA
  - IDSL.IPA
  - IDSL_MINT
  - RDKit
  edam_operation: http://edamontology.org/operation_3646
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# dda-msms-spectrum-deconvolution

## Summary

Deconvolution of DDA LC-MS/MS data: chromatographic peak-picking with IDSL.IPA followed by MS/MS deconvolution with IDSL.CSA to associate fragment spectra with precursor MS1 features, producing unique, deduplicated spectra in NIST .msp format (separately per ionization mode). This step supplies the .msp inputs for downstream fingerprint-prediction models such as IDSL_MINT.

## When to use

Use when raw DDA LC-MS/MS data (e.g. a Metabolomics WorkBench study such as ST002044) must be converted into deconvolved, unique MS/MS spectra in NIST .msp format for library matching or model training/evaluation.

## When NOT to use

- Input is already a deconvolved .msp/.mgf library or a feature table — no peak-picking/deconvolution is needed
- Data are DIA/SWATH or otherwise lack precursor-isolated MS2 scans, since IDSL.CSA associates fragment spectra with precursor MS1 features
- Goal is MS/MS prediction or fingerprint inference rather than spectrum extraction — use IDSL_MINT training/prediction instead

## Inputs

- Raw DDA LC-MS/MS data files (e.g. study ST002044 from Metabolomics WorkBench)
- Ionization-mode split (ESI+ and ESI−) raw file sets
- Optional parameter YAML files specifying peak-picking and deconvolution settings

## Outputs

- Deduplicated unique MS/MS spectra in NIST .msp format with required PrecursorMZ row entries (e.g. csa_spreadsheet.zip content: 3,386 ESI+ and 1,901 ESI− spectra)
- Per-spectrum metadata blocks (Name, Precursor_type, PrecursorMZ, Ion_mode, Collision_energy, Formula, ExactMass, Num Peaks)
- Library-match annotation results (e.g. spectral similarity matches against NIST 20 and LipidBlast)

## How to apply

Download the raw LC-MS/MS files and run IDSL.IPA chromatographic peak-picking to detect MS1 features and their associated MS/MS scans. Run the IDSL.CSA R package to deconvolve MS/MS spectra — associating fragment spectra with precursor features — separately for ESI+ and ESI− modes. Deduplicate the deconvolved spectra within each ionization mode and export unique MS/MS spectra as NIST .msp blocks; every MSP block must include a `PrecursorMZ:` row entry, and SMILES/InChI rows are later converted to canonical SMILES with RDKit. Validate by counting spectrum blocks per mode: the ST002044 case yielded 3,386 blocks in ESI+ and 1,901 in ESI−, matching the reported values. Keep .msp headers such as Name, Precursor_type, Ion_mode, and Collision_energy intact so downstream tools (e.g. IDSL_MINT) can parse `comment:`/`comments:` fields and SMILES/InChI labels without case sensitivity.

## Related tools

- **IDSL.IPA** (Chromatographic peak-picking on raw LC-MS/MS data to detect MS1 features and their associated MS/MS scans prior to deconvolution)
- **IDSL.CSA** (R package that deconvolves MS/MS spectra by associating fragment spectra with precursor features and exports unique spectra in NIST .msp format)
- **IDSL_MINT** (Downstream consumer of the deconvolved .msp files; requires MSP blocks with PrecursorMZ entries and parses SMILES/InChI and comment fields) — https://github.com/idslme/IDSL_MINT
- **RDKit** (Converts InChI and SMILES row entries in MSP blocks into canonical SMILES for fingerprint computation) — https://www.rdkit.org/docs/Install.html

## Evaluation signals

- Count spectrum blocks in each output .msp: ESI+ file must contain 3,386 spectra and ESI− file 1,901 for study ST002044
- Every MSP block contains a `PrecursorMZ:` row entry (hard requirement for downstream IDSL_MINT parsing)
- InChI/SMILES rows parse cleanly and convert to canonical SMILES via RDKit without errors
- Library-match coverage is sanity-checkable: IDSL.CSA annotated 638/3,386 (ESI+) and 299/1,901 (ESI−) spectra against NIST 20 and LipidBlast, so a comparable dataset should show nonzero matches in both modes
- Unique-spectrum deduplication verified per ionization mode — no duplicate precursor blocks within each output file

## Limitations

- Majority of deconvolved spectra lack library annotations: only 638 of 3,386 ESI+ and 299 of 1,901 ESI− spectra matched NIST 20 and LipidBlast (>80% of spectra typically have no library hit)
- MSP blocks without PrecursorMZ entries are not usable by downstream IDSL_MINT processing
- Deconvolution is mode- and dataset-specific: results depend on separate ESI+/ESI− processing and the DDA acquisition providing precursor-associated MS2 scans

## Evidence

- [results] IDSL.CSA extracted 3,386 and 1,901 unique MS/MS spectra in ESI+ and ESI− modes: "The IDSL.CSA R package [2] was used to extract 3,386 and 1,901 unique MS/MS spectra in ESI+ and ESI− for this study"
- [results] NIST .msp extraction as model evaluation set: "First, we processed the raw data for this study to extract MS/MS spectra in the NIST.msp format that can be used as the model evaluation set."
- [readme] MSP PrecursorMZ requirement: "MSP blocks must include `PrecursorMZ: ` row entries."
- [readme] MSP compatibility and comment parsing: "IDSL_MINT can extract information from `comment: ` and `comments: ` entries in ***.msp*** files which enables this platform to process MoNA, GNPS, and other public library"
