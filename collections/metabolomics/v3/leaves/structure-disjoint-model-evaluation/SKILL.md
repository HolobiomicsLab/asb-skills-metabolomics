---
name: structure-disjoint-model-evaluation
description: Use when assessing whether a metabolite annotation or fingerprint-prediction
  model generalizes beyond its training compounds, e.g. before publishing accuracy
  claims for an MS/MS spectrum predictor.
when_to_use_negative:
- When the test compounds share first InChIKey blocks with training compounds - the
  evaluation would overestimate generalization by testing on scaffolds the model has
  already seen
- When comparing against training performance on merged multi-collision-energy spectra
  - the paper showed merging degrades accuracy, so merged testing spectra are not
  comparable to separately modeled collision energies
- When evaluating spectral-library matching tools that do not train a model on compound
  structures - structure-disjoint splitting is specific to learned spectrum-to-fingerprint
  predictors
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: MetFID
  role: Trained ANN whose evaluation is the subject of this skill; loads the .h5 model
    and predicts the 528-bit fingerprint per test spectrum
  repo: https://github.com/aohongyu/MetFID
- name: PyFingerprint
  role: Computes OpenBabel MACCS/FP3/FP4 fingerprints for candidate compounds to score
    against predicted fingerprints
  repo: https://github.com/hcji/PyFingerprint
- name: PubChemPy
  role: Queries PubChem to retrieve candidate compounds by molecular formula for annotation
    evaluation
- name: TensorFlow
  role: Backend that loads the saved .h5 Keras model for inference on testing spectra
provenance:
  source_task_ids:
  - task_003
  - task_007
  - task_001
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:de1808e80505451f2d708791c2803770925d17ff76cfa24c70fed7349d3a236e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/aohongyu/MetFID
  tools:
  - MetFID
  - MetaboQuest
  - MoNA
  - NIST 17
  - PubChemPy
  - PyFingerprint
  - RDKit
  - TensorFlow
  - tabulate
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# Structure-disjoint model evaluation

## Summary

Evaluates a spectrum-to-fingerprint prediction model on test compounds sharing no first InChIKey block with training compounds, so reported accuracy/F1 reflect generalization to novel structures rather than memorization of seen scaffolds.

## When to use

Use when assessing whether a metabolite annotation or fingerprint-prediction model generalizes beyond its training compounds, e.g. before publishing accuracy claims for an MS/MS spectrum predictor.

## When NOT to use

- When the test compounds share first InChIKey blocks with training compounds - the evaluation would overestimate generalization by testing on scaffolds the model has already seen
- When comparing against training performance on merged multi-collision-energy spectra - the paper showed merging degrades accuracy, so merged testing spectra are not comparable to separately modeled collision energies
- When evaluating spectral-library matching tools that do not train a model on compound structures - structure-disjoint splitting is specific to learned spectrum-to-fingerprint predictors

## Inputs

- Training spectra (MoNA and NIST 17 MS/MS) with compound InChIKeys and binned peak vectors
- Testing spectra from compounds sharing no first InChIKey block with training compounds (e.g. NIST, CASMI 2016, or in-house cancer biomarker MS/MS)
- Candidate InChIKey lists per test spectrum (e.g. from MetaboQuest or PubChemPy)
- Pretrained model file (e.g. MetFID_CNN_40088_5618.h5)

## Outputs

- Macro-averaged accuracy and F1 scores for fingerprint prediction on the structure-disjoint test set
- Ranked candidate lists with Tanimoto-similarity-style scores (testing_compound_prediction.txt)
- Top-k annotation accuracy statistics on previously unseen compounds

## How to apply

Split the dataset by compound identity rather than by spectrum: exclude from testing any compound whose first InChIKey block (the skeleton formula identifier) matches a training compound, and for NIST-style multi-collision-energy libraries keep testing to single collision energies without the spectrum-merging used in training. In MetFID this means training the Keras/TensorFlow ANN (two ReLU hidden layers, sigmoid 528-bit output with binary cross-entropy, Adam optimizer, 30 epochs, batch size 100) on MoNA plus NIST 17 spectra, then feeding the binned, denoised testing spectra of previously unseen compounds into the saved .h5 model. Score fingerprint prediction with macro-averaged accuracy and F1 (the paper's Eqs. 2-3) on the structure-disjoint test set. For end-to-end annotation evaluation, predict the 528-bit fingerprint for each spectrum, compute candidate fingerprints with OpenBabel via PyFingerprint, score candidates by delta(f',f) = 1 - sum|f'-f|/n, rank in descending order, and report top-k accuracy by locating the true InChIKey in the ranked list. Also stratify evaluation by collision energy (30 eV cutoff) and instrument (HCD/QqQ/Q-TOF vs IT) when training data mix them, since the paper found collision-energy merging degrades prediction versus separate low/high-energy models.

## Related tools

- **MetFID** (Trained ANN whose evaluation is the subject of this skill; loads the .h5 model and predicts the 528-bit fingerprint per test spectrum) — https://github.com/aohongyu/MetFID
- **PyFingerprint** (Computes OpenBabel MACCS/FP3/FP4 fingerprints for candidate compounds to score against predicted fingerprints) — https://github.com/hcji/PyFingerprint
- **PubChemPy** (Queries PubChem to retrieve candidate compounds by molecular formula for annotation evaluation)
- **TensorFlow** (Backend that loads the saved .h5 Keras model for inference on testing spectra)

## Examples

```
python3 main.py
```

## Evaluation signals

- The set of first InChIKey blocks in testing compounds has zero overlap with the training set's first InChIKey blocks (verify programmatically before scoring)
- Macro-averaged accuracy and F1 on the test set fall within the range reported for structure-disjoint testing (approximately 50% top-1 annotation accuracy with MetaboQuest candidates)
- testing_compound_prediction.txt exists and contains per-block tables of InChIKeys with Tanimoto similarity scores ranked in descending order
- Tested spectra are single collision energy and not merged, matching the no-merging condition described for testing
- Top-k accuracy recomputed on structure-disjoint compounds shows the expected >5% improvement over MetFrag, ChemDistiller, and CSI:FingerID baselines

## Limitations

- Collision-energy merging used during training is inconsistent with separate evaluation, so merging cannot be applied uniformly across both phases; the combined model was slightly less accurate than separate low/high-energy models
- Evaluation depends on candidate list quality from external databases (MetaboQuest 10 PPM precursor m/z, H+ adduct; PubChemPy formula queries), so errors in candidate retrieval propagate to annotation accuracy
- The paper did not release pre-trained model weights beyond the README-linked MetFID_CNN_40088_5618.h5 and did not discuss target-decoy FDR estimation for ranking specificity

## Evidence

- [results] Test on previously unseen compounds with single collision energy: [evidence span withheld — anchor status: not_found]
- [results] Evaluation metrics on fingerprint prediction: [evidence span withheld — anchor status: not_found]
- [results] Top-1 annotation accuracy on MetaboQuest candidates: [evidence span withheld — anchor status: not_found]
- [results] Comparison against baseline tools: [evidence span withheld — anchor status: not_found]
