---
name: neims-spectrum-prediction
description: Use when you need EI mass spectra for molecules lacking measured spectra
  — e.g., building an in-silico library from SMILES/SDF sets like f-NIST (255,821)
  and ChEMBL 28 to expand coverage beyond the 0.27M compounds in NIST 2017.
when_to_use_negative:
- When you only need to search/match query spectra against a library — use FastEI
  (Word2vec embedding + HNSW) instead of re-predicting spectra
- When the ready-made IN_SILICO_LIBRARY.db (Zenodo record 7293922) suffices, avoiding
  redundant prediction of 2,146,690 spectra
- When molecules fall outside the article's filter scope (mass ≥ 1000 Da, ionic compounds,
  elements beyond the 11 common ones) since NEIMS predictions are not validated there
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: NEIMS
  role: Pre-trained fully connected neural network that maps ECFP fingerprints to
    predicted EI mass spectra; used without re-training
  repo: https://github.com/brain-research/deep-molecular-massspec
- name: RDKit (v2022.03.3)
  role: Reads SMILES/SDF structures and generates the extended circular fingerprints
    (ECFPs) that serve as NEIMS inputs
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:dc3d9cdb03dc8389627c793c06dec85950c332f93917832d4b74477d08fa7b15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/brain-research/deep-molecular-massspec
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
yes_rate: 0.7
n_claims: 10
---

# neims-spectrum-prediction

## Summary

Predict electron-ionization mass spectra for arbitrary molecules by feeding RDKit-generated ECFP fingerprints into the pre-trained NEIMS fully connected neural network, enabling construction of million-scale in-silico libraries without any experimental measurement or re-training.

## When to use

Use when you need EI mass spectra for molecules lacking measured spectra — e.g., building an in-silico library from SMILES/SDF sets like f-NIST (255,821) and ChEMBL 28 to expand coverage beyond the 0.27M compounds in NIST 2017.

## When NOT to use

- When you only need to search/match query spectra against a library — use FastEI (Word2vec embedding + HNSW) instead of re-predicting spectra
- When the ready-made IN_SILICO_LIBRARY.db (Zenodo record 7293922) suffices, avoiding redundant prediction of 2,146,690 spectra
- When molecules fall outside the article's filter scope (mass ≥ 1000 Da, ionic compounds, elements beyond the 11 common ones) since NEIMS predictions are not validated there

## Inputs

- SMILES strings or SDF structure files of molecules (e.g., ChEMBL 28 SDF from ftp.ebi.ac.uk, HMDB 5.0 structures.zip, f-NIST splits)
- Pre-trained NEIMS model weights (massspec_weights.zip from storage.googleapis.com/deep-molecular-massspec)
- RDKit-generated ECFP fingerprints of the molecules

## Outputs

- Predicted EI mass spectrum per molecule (e.g., annotated SDF via make_spectra_prediction.py)
- Million-scale in-silico spectral library (IN_SILICO_LIBRARY.db with exactly 2,146,690 molecule entries)
- Per-molecule WCS scores against experimental spectra when evaluated on the NIST 2017 test split (wcs_scores.csv)

## How to apply

Download the pre-trained NEIMS weights from the official deep-molecular-massspec repository (massspec_weights.zip) without re-training. Read each molecule's structure with RDKit (v2022.03.3) and generate its extended circular fingerprint (ECFP); NEIMS uses ECFPs as inputs to fully connected neural networks. Feed the ECFPs of every molecule (e.g., all of f-NIST, f-ChEMBL, f-HMDB, and the extra test set) through the network to obtain predicted EI mass spectra. For library construction, combine predictions of f-NIST and f-ChEMBL into a single database (IN_SILICO_LIBRARY.db, 2,146,690 molecules). Apply the article's filtering criteria first (mass < 1000 Da, only the 11 elements H/C/O/N/P/S/Cl/F/Br/I/Si, non-ionic, LogP −12 to 24, deduplication) before predicting. Validate by reproducing NEIMS benchmark behavior: ~78% of NIST test molecules should reach weighted cosine similarity (WCS) > 0.7 between predicted and experimental spectra.

## Related tools

- **NEIMS** (Pre-trained fully connected neural network that maps ECFP fingerprints to predicted EI mass spectra; used without re-training) — https://github.com/brain-research/deep-molecular-massspec
- **RDKit (v2022.03.3)** (Reads SMILES/SDF structures and generates the extended circular fingerprints (ECFPs) that serve as NEIMS inputs)

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- On the NIST 2017 test split (11,499 molecules), ~78% of predicted-vs-experimental pairs reach weighted cosine similarity (WCS) > 0.7, reproducing Supplementary Fig. 2
- Library construction yields IN_SILICO_LIBRARY.db containing exactly 2,146,690 molecule entries
- Predictions are generated from the unmodified official massspec_weights.zip — any re-training or modified weights invalidate comparability
- Every predicted molecule must pass the five filters beforehand: mass < 1000 Da, only the 11 common elements, non-ionic, LogP in [−12, 24], deduplicated
- make_spectra_prediction.py produces an annotated SDF output file containing predicted spectra for the input structures

## Limitations

- Prediction quality depends on the NEIMS training distribution; molecules beyond NIST 2017 (e.g., synthetic compounds in the extra test set) show lower identification performance (recall@1 = 50%)
- NEIMS weights were used as downloaded with no version/build date stated, complicating reproducibility
- Molecules excluded by the five filtering steps (Supplementary Data 1) should not be predicted without additional validation

## Evidence

- [intro] ECFP inputs to fully connected networks: "The neural electron-ionization mass spectrometry (NEIMS) method takes the extended circular fingerprints (ECFPs) of molecules as inputs and applies fully connected neural networks"
- [discussion] NEIMS weights downloaded without re-training: "The NEIMS model was downloaded from its official repository (https://storage.googleapis.com/deep-molecular-massspec/massspec_weights/massspec_weights.zip) without re-training"
- [discussion] All molecule sets fed into NEIMS for predicted spectra: "The ECFPs of all molecules in f-NIST, f-ChEMBL, f-HMDB, and the extra test set were fed into the NEIMS model to obtain their predicted mass spectra"
- [readme] README prediction invocation and weights download: [evidence span withheld — anchor status: not_found]
- [other] WCS validation threshold from source card workflow: [evidence span withheld — anchor status: not_found]
