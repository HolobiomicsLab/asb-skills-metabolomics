---
name: in-silico-mass-spectral-library-construction
description: Use when an experimental spectral library (e.g., NIST 2017 with only
  0.27 million EI spectra) lacks coverage and you need large-scale predicted spectra
  from molecular structures to support compound identification by spectrum matching.
when_to_use_negative:
- The goal is predicting spectra for ionization modes or fragmentation physics beyond
  EI, e.g. when quantum-chemistry-level accuracy (QCEIMS-style BOMD sampling) or collision-induced
  dissociation spectra are required.
- Molecules fall outside the filtering scope (mass ≥ 1000 Da, ionic compounds, elements
  beyond H, C, O, N, P, S, Cl, F, Br, I, Si), since NEIMS was trained and applied
  only on the filtered subsets.
- You need experimentally measured reference spectra (e.g., NIST 2017 library entries)
  rather than predicted spectra for definitive identification.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: NEIMS
  role: Pre-trained fully connected neural network that takes ECFPs as inputs to predict
    EI mass spectra, used without re-training
  repo: https://storage.googleapis.com/deep-molecular-massspec/massspec_weights/massspec_weights.zip
- name: RDKit (v2022.03.3)
  role: Reads SMILES strings from parsed SDF structures and generates extended circular
    fingerprints (ECFPs) for every molecule
- name: FastEI
  role: Downstream consumer of the in-silico library for ultra-fast spectrum matching
    via Word2vec embedding and HNSW
  repo: https://github.com/Qiong-Yang/FastEI
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:7f4920757cb37f61183ddb436d17afd904faaa41f952fb41e5af69189da13fc3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FastEI
  - NEIMS
  - RDKit (v2022.03.3)
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# in-silico-mass-spectral-library-construction

## Summary

Builds a million-scale in-silico electron ionization mass spectral library by predicting EI-MS spectra for filtered molecular structure sets (f-NIST, f-ChEMBL) with the pre-trained NEIMS neural network, producing IN_SILICO_LIBRARY.db with 2,146,690 molecule entries.

## When to use

Use when an experimental spectral library (e.g., NIST 2017 with only 0.27 million EI spectra) lacks coverage and you need large-scale predicted spectra from molecular structures to support compound identification by spectrum matching.

## When NOT to use

- The goal is predicting spectra for ionization modes or fragmentation physics beyond EI, e.g. when quantum-chemistry-level accuracy (QCEIMS-style BOMD sampling) or collision-induced dissociation spectra are required.
- Molecules fall outside the filtering scope (mass ≥ 1000 Da, ionic compounds, elements beyond H, C, O, N, P, S, Cl, F, Br, I, Si), since NEIMS was trained and applied only on the filtered subsets.
- You need experimentally measured reference spectra (e.g., NIST 2017 library entries) rather than predicted spectra for definitive identification.

## Inputs

- f-NIST molecule set (255,821 compounds, union of NEIMS training, validation, and test splits)
- ChEMBL 28 gzipped structure-data file (chembl_28.sdf.gz)
- NEIMS pre-trained model weights (massspec_weights.zip)
- RDKit v2022.03.3 for SMILES parsing and ECFP generation

## Outputs

- Predicted EI mass spectra for f-NIST and f-ChEMBL molecules
- IN_SILICO_LIBRARY.db — a single in-silico spectral library containing 2,146,690 molecule entries

## How to apply

Start from molecule sets that passed five filtering steps (molecular mass < 1000 Da; only the 11 elements H, C, O, N, P, S, Cl, F, Br, I, Si; no ionic compounds; LogP between −12 and 24; deduplication), specifically f-NIST (255,821 compounds from the NEIMS training/validation/test split union) and f-ChEMBL from ChEMBL 28 SDF. Parse structures and read SMILES with RDKit (v2022.03.3), generate extended circular fingerprints (ECFPs) for every molecule, and feed the ECFPs into the NEIMS fully connected network using the pre-trained weights from the official deep-molecular-massspec repository without any re-training. Combine the predicted spectra of f-NIST and f-ChEMBL into a single in-silico spectral library and store it as IN_SILICO_LIBRARY.db (also distributed via Zenodo record 7293922). The rationale is that machine-learning prediction from structures quickly extends the chemical space beyond what experimental or quantum-chemistry (QCEIMS) spectra generation can cover, solving the library-coverage bottleneck in GC-MS compound identification.

## Related tools

- **NEIMS** (Pre-trained fully connected neural network that takes ECFPs as inputs to predict EI mass spectra, used without re-training) — https://storage.googleapis.com/deep-molecular-massspec/massspec_weights/massspec_weights.zip
- **RDKit (v2022.03.3)** (Reads SMILES strings from parsed SDF structures and generates extended circular fingerprints (ECFPs) for every molecule)
- **FastEI** (Downstream consumer of the in-silico library for ultra-fast spectrum matching via Word2vec embedding and HNSW) — https://github.com/Qiong-Yang/FastEI

## Examples

```
curl -o https://storage.googleapis.com/deep-molecular-massspec/massspec_weights/massspec_weights.zip; unzip massspec_weights.zip; python make_spectra_prediction.py --input_file=chembl_28.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- IN_SILICO_LIBRARY.db exists and contains exactly 2,146,690 molecule entries, matching the library size reported in the article.
- The f-NIST input set contains exactly 255,821 compounds (the union of NEIMS training, validation, and test splits).
- Predicted spectra are generated from ECFPs via NEIMS pre-trained weights, with no re-training performed.
- Library entries derive only from molecules passing all five filter steps (mass < 1000 Da, 11 allowed elements, non-ionic, LogP in [−12, 24], deduplicated).
- Every molecule entry has an associated predicted spectrum generated from its ECFP input.

## Limitations

- The NEIMS model was downloaded from its official repository and used without re-training, so prediction quality depends entirely on its original training corpus.
- Spectra are in-silico predictions, not measured spectra; identification accuracy on molecules beyond the NIST 2017 library drops to 50% recall@1 (90% recall@20 on the 10-compound extra test set).
- Molecules excluded by the five filtering steps (ionic compounds, heavy or exotic-element molecules, extreme LogP) are not represented in the library.

## Evidence

- [discussion] Library built from predicted f-NIST and f-ChEMBL spectra: "the predicted spectra of molecules in f-NIST and f-ChEMBL were used to build the in-silico library, which included 2,146,690 molecules"
- [discussion] ECFPs fed into NEIMS to obtain predicted spectra: "The ECFPs of all molecules in f-NIST, f-ChEMBL, f-HMDB, and the extra test set were fed into the NEIMS model to obtain their predicted mass spectra"
- [discussion] NEIMS used without re-training from official weights: "The NEIMS model was downloaded from its official repository (https://storage.googleapis.com/deep-molecular-massspec/massspec_weights/massspec_weights.zip) without re-training"
- [discussion] f-NIST composition: "The f-NIST (255,821) was obtained by combining the training set, the validation set, and the test set"
- [readme] NEIMS input representation from README: [evidence span withheld — anchor status: not_found]
