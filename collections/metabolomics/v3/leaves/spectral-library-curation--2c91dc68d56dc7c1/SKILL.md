---
name: spectral-library-curation
description: Use when compiling a reference MS/MS training set (e.g. from LipidMaps,
  MoNA, GNPS .msp files) for models that will be evaluated on spectra annotated by
  other libraries, to avoid leakage and class sparsity.
when_to_use_negative:
- The .msp library will be used directly for library-match annotation of the evaluation
  spectra (no model training), so exclusion would remove needed reference spectra.
- Lipid class coverage is already high and filtering to ≥50 spectra would discard
  classes you need for a rare-class application.
- The goal is in-silico spectrum prediction from canonical SMILES, not curation of
  experimentally measured training spectra.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: IDSL_MINT
  role: Framework whose parser consumes the curated .msp training library; reads comment/comment
    entries, PrecursorMZ, and Fingerprint/InChI/SMILES row entries to build the transformer
    model's training set.
  repo: https://github.com/idslme/IDSL_MINT
- name: RDKit
  role: Used by IDSL_MINT to convert InChI and SMILES row entries into canonical SMILES
    while assembling curated MSP blocks.
  repo: https://www.rdkit.org/docs/Install.html
- name: PyTorch
  role: Backend for the transformer models that consume the curated training library
    during IDSL_MINT training.
  repo: https://pytorch.org/get-started/locally
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:c4723d661c458e0b79312eb5a80eaa2f13bfb3e72ecd9750a1aa48f92d6987a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL_MINT
  - PyTorch
  - RDKit
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spectral-library-curation

## Summary

Compile a defensible MS/MS training library by parsing .msp blocks, dropping in-silico entries (e.g. LipidBlast), excluding compounds already annotated by held-out evaluation libraries (NIST 20), and retaining only well-covered classes.

## When to use

Use when compiling a reference MS/MS training set (e.g. from LipidMaps, MoNA, GNPS .msp files) for models that will be evaluated on spectra annotated by other libraries, to avoid leakage and class sparsity.

## When NOT to use

- The .msp library will be used directly for library-match annotation of the evaluation spectra (no model training), so exclusion would remove needed reference spectra.
- Lipid class coverage is already high and filtering to ≥50 spectra would discard classes you need for a rare-class application.
- The goal is in-silico spectrum prediction from canonical SMILES, not curation of experimentally measured training spectra.

## Inputs

- LipidMaps reference library bundle (lipidmaps_msp.zip from Zenodo) in .msp format
- LipidBlast in-silico library
- NIST 20 library
- Evaluation-set compound identifiers (e.g. study ST002044 annotated against NIST 20 and LipidBlast)

## Outputs

- Curated training set lipidmaps_curated_training.msp (positive and negative modes kept separate)
- Per-lipid-class spectrum counts with retained classes (≥50 spectra)
- Excluded spectra of compounds covered by LipidBlast or NIST 20

## How to apply

Download the reference library bundle (e.g. lipidmaps_msp.zip from Zenodo) and parse each .msp spectral block, which must contain PrecursorMZ row entries. Drop in-silico spectra (e.g. LipidBlast-predicted blocks) so only experimentally measured spectra remain. Cross-reference remaining compound identifiers against the libraries reserved for annotating the independent evaluation set — in the ST002044 case study, NIST 20 and LipidBlast — and exclude any compound covered there, preventing leakage between training and evaluation. Count spectra per lipid class per ion mode and keep only classes with at least 50 spectra (6 of 8 lipid classes passed); keep positive and negative modes separate and write the curated set as lipidmaps_curated_training.msp. IDSL_MINT parses fingerprint or InChI/SMILES row entries from comment/comment entries to build the eventual model training set.

## Related tools

- **IDSL_MINT** (Framework whose parser consumes the curated .msp training library; reads comment/comment entries, PrecursorMZ, and Fingerprint/InChI/SMILES row entries to build the transformer model's training set.) — https://github.com/idslme/IDSL_MINT
- **RDKit** (Used by IDSL_MINT to convert InChI and SMILES row entries into canonical SMILES while assembling curated MSP blocks.) — https://www.rdkit.org/docs/Install.html
- **PyTorch** (Backend for the transformer models that consume the curated training library during IDSL_MINT training.) — https://pytorch.org/get-started/locally

## Examples

```
MINT_workflow --yaml /path/to/yaml/file
```

## Evaluation signals

- In positive mode, the curated set covers 6.96% of all LipidMaps compounds; in negative mode, 4.58%.
- Exactly 6 of 8 lipid classes remain after applying the ≥50-spectra-per-class threshold.
- No spectrum of any compound covered by LipidBlast or NIST 20 remains in the curated training set.
- Every retained .msp block contains a PrecursorMZ row entry and parsable InChI/SMILES or Fingerprint row entries.
- Positive and negative ion mode spectra remain in separate files.

## Limitations

- Excluding compounds covered by the annotation libraries reduces training coverage to a small fraction of the reference database (6.96% positive, 4.58% negative of LipidMaps).
- The ≥50-spectra class cutoff discards 2 of 8 lipid classes, limiting model applicability to rare classes.
- MSP blocks with multiple similar headers rely on IDSL_MINT's longest-content selection rule; unusual headers may be mis-parsed.
- Training library is lipid-class specific (lipid classes with ≥50 spectra); user classes of interest may not be included.

## Evidence

- [results] Drop in-silico blocks and cross-reference against held-out libraries: "We excluded the spectra related to compounds that were covered in the LipidBlast (in-silico) [23] and NIST 20 libraries because we used these two libraries for annotating the experimental MS/MS"
- [results] Compile the training set before training: "Next, we carefully compiled the training dataset for the case study (ST002044) to ensure a high validity of the resulting model (Fig. 3)."
- [results] Case study coverage of LipidMaps compounds: "Our case study training set included 6.96% and 4.58% of all the compounds in the LipidMaps database in positive and negative modes, respectively"
- [results] Lipid class threshold: "Training data has 8 lipid classes and 6 classes having at least 50 spectra (Table 1)."
- [readme] MSP must contain PrecursorMZ and how headers are chosen: "MSP blocks must include `PrecursorMZ: ` row entries."
- [readme] IDSL_MINT reads comment entries from public libraries: "**IDSL_MINT** can extract information from `comment: ` and `comments: ` entries in ***.msp*** files which enables this platform to process MoNA, GNPS, and other public library"
