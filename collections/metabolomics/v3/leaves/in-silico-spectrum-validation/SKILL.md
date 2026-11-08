---
name: in-silico-spectrum-validation
description: Use when you need to benchmark a machine-learned (e.g., NEIMS) EI-MS
  predictor against experimental spectra before trusting its outputs for library building
  or matching.
when_to_use_negative:
- The goal is fast library matching / compound identification ranking rather than
  assessing spectrum prediction quality — use FastEI's HNSW + Word2vec matching instead
- Spectra arise from ionization methods other than electron ionization (EI), since
  NEIMS was trained only for EI-MS prediction from ECFPs
- The molecules lack valid SMILES or cannot be parsed into ECFP fingerprints by RDKit
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: NEIMS
  role: Fully connected neural network that takes ECFP fingerprints as input and predicts
    the EI mass spectrum for each test molecule; weights used without re-training
  repo: https://github.com/brain-research/deep-molecular-massspec
- name: RDKit (v2022.03.3)
  role: Reads the SMILES strings of the test molecules and generates their ECFP fingerprints
    used as NEIMS input
  repo: https://www.rdkit.org/docs/Install.html
- name: faiss.IndexFlatL2
  role: Exact nearest-neighbor search on normalized spectral vectors, useful as the
    exact-search reference against which prediction-quality scores can be compared
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:1ec5f4655c98f23dad382f84fb2102bd9c5d2da2f3bd857b9b3fdafeb780155e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/brain-research/deep-molecular-massspec
  tools:
  - FastEI
  - NEIMS
  - RDKit (v2022.03.3)
  - faiss.IndexFlatL2
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# in-silico-spectrum-validation

## Summary

Validates the quality of in-silico EI mass spectra predicted by the unmodified NEIMS model by computing weighted cosine similarity (WCS) between each predicted spectrum and its experimental counterpart, confirming that ~78% of the 11,499 NIST 2017 test molecules achieve WCS > 0.7.

## When to use

Use when you need to benchmark a machine-learned (e.g., NEIMS) EI-MS predictor against experimental spectra before trusting its outputs for library building or matching.

## When NOT to use

- The goal is fast library matching / compound identification ranking rather than assessing spectrum prediction quality — use FastEI's HNSW + Word2vec matching instead
- Spectra arise from ionization methods other than electron ionization (EI), since NEIMS was trained only for EI-MS prediction from ECFPs
- The molecules lack valid SMILES or cannot be parsed into ECFP fingerprints by RDKit

## Inputs

- NIST 2017 test-split molecules: SMILES strings paired with measured (experimental) EI-MS spectra (11,499 molecules)
- Pre-trained NEIMS model weights (massspec_weights.zip from deep-molecular-massspec, used without re-training)

## Outputs

- Predicted EI mass spectrum for each of the 11,499 test molecules
- wcs_scores.csv: per-molecule weighted cosine similarity scores between predicted and experimental spectra
- Fraction of test molecules with WCS > 0.7 (expected ~78%)

## How to apply

Download the pre-trained NEIMS weights from the official deep-molecular-massspec repository (massspec_weights.zip) without re-training, along with the NIST 2017 test split (SMILES and measured spectra) for the 11,499 test molecules. Read the SMILES strings with RDKit (v2022.03.3) and generate their ECFP fingerprints, then feed the ECFPs into the NEIMS fully connected neural network to obtain a predicted EI mass spectrum for every test molecule. Compute the weighted cosine similarity (WCS) between each predicted spectrum and its experimental counterpart, writing all per-molecule scores to wcs_scores.csv. Threshold the WCS distribution at 0.7 and compute the fraction of molecules above the threshold; a successful reproduction matches the reported ~78%, consistent with Supplementary Fig. 2 of the NEIMS evaluation.

## Related tools

- **NEIMS** (Fully connected neural network that takes ECFP fingerprints as input and predicts the EI mass spectrum for each test molecule; weights used without re-training) — https://github.com/brain-research/deep-molecular-massspec
- **RDKit (v2022.03.3)** (Reads the SMILES strings of the test molecules and generates their ECFP fingerprints used as NEIMS input) — https://www.rdkit.org/docs/Install.html
- **faiss.IndexFlatL2** (Exact nearest-neighbor search on normalized spectral vectors, useful as the exact-search reference against which prediction-quality scores can be compared)

## Examples

```
python make_spectra_prediction.py --input_file=test_molecules.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- The fraction of the 11,499 test molecules with WCS > 0.7 is ~78%, reproducing the reported value in Supplementary Fig. 2
- wcs_scores.csv contains one weighted cosine similarity score per test molecule (row count = 11,499) with values in [0, 1]
- The WCS score distribution's shape (median, spread) is qualitatively consistent with the NEIMS paper's published distribution
- Predicted spectra use the same m/z binning as the experimental library so cosine similarities are computed over aligned vectors

## Limitations

- The NEIMS model was downloaded without re-training, and the version/build date of the pre-trained weights is not stated, which may affect exact reproducibility of the ~78% figure
- WCS is a similarity metric only; the article notes the WCS baseline is not defined in the visible text and that exact nearest-neighbor search by WCS is time-consuming at million-scale library sizes
- Performance applies to EI-MS of small organic molecules; extrapolation to other ionization modes or very large molecules is not validated

## Evidence

- [discussion] The NEIMS model was downloaded from its official repository (https://storage.googleapis.com/deep-molecular-massspec/massspec_weights/massspec_weights.zip) without re-training: "The NEIMS model was downloaded from its official repository (https://storage.googleapis.com/deep-molecular-massspec/massspec_weights/massspec_weights.zip) without re-training"
- [discussion] The SMILES strings of molecules were read by RDKit (v2022.03.3): "The SMILES strings of molecules were read by RDKit (v2022.03.3)"
- [discussion] The ECFPs of all molecules in f-NIST, f-ChEMBL, f-HMDB, and the extra test set were fed into the NEIMS model to obtain their predicted mass spectra: "The ECFPs of all molecules in f-NIST, f-ChEMBL, f-HMDB, and the extra test set were fed into the NEIMS model to obtain their predicted mass spectra"
- [intro] The neural electron-ionization mass spectrometry (NEIMS) method takes the extended circular fingerprints (ECFPs) of molecules as inputs and applies fully connected neural networks to predict the: "The neural electron-ionization mass spectrometry (NEIMS) method takes the extended circular fingerprints (ECFPs) of molecules as inputs and applies fully connected neural networks to predict the"
- [intro] We have demonstrated the performance of FastEI in terms of identification accuracy and spectrum matching speed on a test set and an extra test set: "We have demonstrated the performance of FastEI in terms of identification accuracy and spectrum matching speed on a test set and an extra test set"
