---
name: ecfp-fingerprint-generation
description: Use when you have molecular structures (SMILES strings or SDF files)
  and need fixed-length ECFP representations as model inputs for in-silico EI mass
  spectrum prediction with NEIMS or similar fingerprint-based models.
when_to_use_negative:
- 'Molecules failing the article''s filters: mass >= 1000 Da, elements outside the
  11 common elements (H, C, O, N, P, S, Cl, F, Br, I, Si), ionic compounds, or LogP
  outside -12 to 24'
- When experimental EI mass spectra are already available — direct library matching
  against measured spectra (e.g., NIST 2017) is preferable
- Non-Windows environments for the packaged FastEI GUI, which the README states only
  supports Windows 64-bit (tested on Windows 7 and 10)
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: RDKit (v2022.03.3)
  role: Reads SMILES strings / parses SDF files and generates the ECFP fingerprints
    for every molecule
  repo: https://www.rdkit.org/docs/Install.html
- name: NEIMS
  role: Fully connected neural network that consumes the generated ECFPs to predict
    EI mass spectra; official weights used without re-training
  repo: https://github.com/brain-research/deep-molecular-massspec
- name: FastEI
  role: Downstream consumer of NEIMS-predicted spectra; packages the in-silico library,
    Word2vec embedding, and HNSW index for spectrum matching
  repo: https://github.com/Qiong-Yang/FastEI
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:4ce99dfc1d12bfd07813d7ab4f500839f3ecd24d8e45f1ef5a58d116c1e699bd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FastEI
  - NEIMS
  - RDKit (v2022.03.3)
  - faiss.IndexFlatL2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# ecfp-fingerprint-generation

## Summary

Generate extended circular fingerprints (ECFPs) from molecular structures with RDKit so they can be fed into the NEIMS fully connected neural network to predict EI-MS spectra, enabling large-scale in-silico spectral library construction.

## When to use

Use when you have molecular structures (SMILES strings or SDF files) and need fixed-length ECFP representations as model inputs for in-silico EI mass spectrum prediction with NEIMS or similar fingerprint-based models.

## When NOT to use

- Molecules failing the article's filters: mass >= 1000 Da, elements outside the 11 common elements (H, C, O, N, P, S, Cl, F, Br, I, Si), ionic compounds, or LogP outside -12 to 24
- When experimental EI mass spectra are already available — direct library matching against measured spectra (e.g., NIST 2017) is preferable
- Non-Windows environments for the packaged FastEI GUI, which the README states only supports Windows 64-bit (tested on Windows 7 and 10)

## Inputs

- SMILES strings of molecules (e.g., 11,499 NIST 2017 test-split molecules)
- Gzipped structure-data file of ChEMBL 28 (chembl_28.sdf.gz)
- SDF file of HMDB 5.0 (structures.zip)
- Pre-trained NEIMS model weights (massspec_weights.zip, used without re-training)

## Outputs

- Per-molecule ECFP fingerprint vectors
- Predicted EI mass spectra for every fingerprinted molecule
- In-silico spectral library (IN_SILICO_LIBRARY.db with 2,146,690 molecule entries)
- Per-molecule similarity scores (e.g., wcs_scores.csv of weighted cosine similarities)

## How to apply

Read SMILES strings or parse structure-data files (e.g., chembl_28.sdf.gz, HMDB structures.zip) with RDKit (v2022.03.3). Generate extended circular fingerprints (ECFPs) for every molecule; these vectors are the required input format for NEIMS, which applies fully connected neural networks to predict EI mass spectra. Apply the article's filtering steps first (molecular mass < 1000 Da, only the 11 elements H/C/O/N/P/S/Cl/F/Br/I/Si, non-ionic, LogP between -12 and 24, deduplication) before fingerprinting. Feed the ECFPs of all molecules (e.g., the 11,499 NIST 2017 test molecules or the f-NIST + f-ChEMBL set totaling 2,146,690 compounds) into the pre-trained NEIMS weights from the official deep-molecular-massspec repository without re-training. Write per-molecule predicted spectra or similarity scores (e.g., weighted cosine similarity to wcs_scores.csv) for downstream thresholding.

## Related tools

- **RDKit (v2022.03.3)** (Reads SMILES strings / parses SDF files and generates the ECFP fingerprints for every molecule) — https://www.rdkit.org/docs/Install.html
- **NEIMS** (Fully connected neural network that consumes the generated ECFPs to predict EI mass spectra; official weights used without re-training) — https://github.com/brain-research/deep-molecular-massspec
- **FastEI** (Downstream consumer of NEIMS-predicted spectra; packages the in-silico library, Word2vec embedding, and HNSW index for spectrum matching) — https://github.com/Qiong-Yang/FastEI

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- Fingerprint generation succeeds for all molecules in the split — e.g., ECFPs for all 11,499 NIST test molecules reproduce the reported ~78% of molecules with predicted-vs-experimental WCS > 0.7
- IN_SILICO_LIBRARY.db exists and contains exactly 2,146,690 molecule entries, matching the library size reported in the article
- Predicted spectra are obtainable for every fingerprinted molecule from f-NIST, f-ChEMBL, f-HMDB, and the extra test set
- NEIMS weights are used exactly as downloaded from storage.googleapis.com/deep-molecular-massspec/massspec_weights.zip without any re-training
- Filtering precedes fingerprinting: excluded molecules (Supplementary Data 1) correspond to the five filter rules, so no filtered molecule yields a spectrum in the library

## Limitations

- NEIMS quality degrades outside the fingerprint space it was trained on; only ~20% of GC-MS compounds are identifiable by current libraries and molecules beyond NIST 2017 (extra test set) reach only 50% recall@1
- Performance claims on the 10-molecule extra test set rely on a very small sample
- Version or build date of the NEIMS pre-trained weights is not stated, and the Word2vec hyperparameter table is only in Supplementary Table 1

## Evidence

- [intro] The NEIMS method takes the extended circular fingerprints (ECFPs) of molecules as inputs and applies fully connected neural networks to predict the: [evidence span withheld — anchor status: not_found]
- [discussion] The SMILES strings of molecules were read by RDKit (v2022.03.3): "The SMILES strings of molecules were read by RDKit (v2022.03.3)"
- [discussion] The ECFPs of all molecules in f-NIST, f-ChEMBL, f-HMDB, and the extra test set were fed into the NEIMS model to obtain their predicted mass spectra: "The ECFPs of all molecules in f-NIST, f-ChEMBL, f-HMDB, and the extra test set were fed into the NEIMS model to obtain their predicted mass spectra"
- [readme] python make_spectra_prediction.py \
--input_file=examples/pentachlorobenzene.sdf \
--output_file=/tmp/annotated.sdf \
--weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights: "$ python make_spectra_prediction.py \
--input_file=examples/pentachlorobenzene.sdf \
--output_file=/tmp/annotated.sdf \
--weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights"
