---
name: molecular-fingerprint-tanimoto-label-generation
description: Use when building training data for ML models that predict chemical structural
  similarity from MS/MS spectra pairs, and each spectrum's compound has (or can be
  resolved to) an InChIKey/SMILES so a fingerprint can be computed.
when_to_use_negative:
- Spectra lack InChIKey/SMILES (or the structure cannot be recovered from PubChem),
  so no fingerprint can be computed — MS2DeepScore relies solely on peak m/z and intensity,
  but its labels require known structures
- You need a spectral similarity measure for library matching at inference time (e.g.,
  modified Cosine or Spec2Vec) rather than supervised structural-similarity training
  labels
- You need deterministic, explainable chemical similarity — the article notes 'a score
  itself is not deterministic but might change when a new model is trained'
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: RDKit
  role: Computes the topological Daylight (and Morgan-2/3) fingerprints from which
    Tanimoto labels are derived
- name: ms2deepscore
  role: Consumes the Tanimoto labels as regression targets for Siamese network training;
    provides SettingsMS2Deepscore and training wrapper
  repo: https://github.com/matchms/ms2deepscore
- name: pubchempy
  role: Automated search against PubChem to recover missing compound structures for
    fingerprint computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:79687fdc340b90b8632f7ce07541388654d3a5eb35f1f854f4cf01306b5f6000
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore
  - RDKit
  - ms2deepscore
  - pubchempy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Molecular fingerprint Tanimoto label generation

## Summary

Compute structural similarity labels (Tanimoto scores) for spectrum pairs from chemical structure annotations (InChIKey/SMILES) using RDKit fingerprints, providing the ground-truth targets for training the MS2DeepScore Siamese neural network.

## When to use

Use when building training data for ML models that predict chemical structural similarity from MS/MS spectra pairs, and each spectrum's compound has (or can be resolved to) an InChIKey/SMILES so a fingerprint can be computed.

## When NOT to use

- Spectra lack InChIKey/SMILES (or the structure cannot be recovered from PubChem), so no fingerprint can be computed — MS2DeepScore relies solely on peak m/z and intensity, but its labels require known structures
- You need a spectral similarity measure for library matching at inference time (e.g., modified Cosine or Spec2Vec) rather than supervised structural-similarity training labels
- You need deterministic, explainable chemical similarity — the article notes 'a score itself is not deterministic but might change when a new model is trained'

## Inputs

- Annotated MS/MS spectra with InChIKey and/or SMILES metadata (e.g., the cleaned GNPS dataset of 210,407 spectra from Zenodo record 4699300)
- Molecule structure representations (SMILES/InChI) resolvable to RDKit molecule objects
- Sampled spectrum pairs drawn from the training set (~102,536 spectra pairs from 14,062 unique training molecules)

## Outputs

- Tanimoto similarity label (float in [0,1]) per spectrum pair, for training/validation/test
- Molecule-level split assignment by unique InChIKey (train n=14,062, validation n=500, test n=500)
- Per-bin RMSE evaluation of predicted vs. true Tanimoto scores on the held-out 3601-spectrum test set

## How to apply

Retrieve or parse the chemical structure for each annotated spectrum — the GNPS-derived dataset used here carries InChIKey and SMILES annotations (184,698 of 210,407 spectra annotated; unresolved cases were searched against PubChem via pubchempy). Compute topological Daylight fingerprints in RDKit (2048-bit; Morgan-2/Morgan-3 ECFP-style fingerprints were also explored) for each compound, then compute the Tanimoto similarity for every sampled spectrum pair to produce the regression label. Crucially, split compounds — not spectra — into train/validation/test by unique InChIKeys (14,062 / 500 / 500 of 15,062 total) so that no molecule appears in more than one split, preventing structural leakage. Pairs should span the Tanimoto range 0–1 (MS2DeepScore evaluates predictions between 0.1 and 0.9), since a model trained only on high-similarity pairs cannot calibrate low-similarity scores.

## Related tools

- **RDKit** (Computes the topological Daylight (and Morgan-2/3) fingerprints from which Tanimoto labels are derived)
- **ms2deepscore** (Consumes the Tanimoto labels as regression targets for Siamese network training; provides SettingsMS2Deepscore and training wrapper) — https://github.com/matchms/ms2deepscore
- **pubchempy** (Automated search against PubChem to recover missing compound structures for fingerprint computation)

## Evaluation signals

- Every Tanimoto label falls within [0,1] and sampled pairs cover a range of similarities (the trained model is only validated for predictions between 0.1 and 0.9)
- No InChIKey (or first-block InChIKey collision) appears in more than one of the train/validation/test splits (14,062/500/500)
- Fingerprints are the intended type and size (Daylight 2048-bit; alternatives Morgan-2/Morgan-3) and are computed from validated SMILES/InChI, not precursor m/z
- Downstream check: a model trained on these labels should reach RMSE ~0.13–0.2 on the 3601-spectrum test set of 500 unseen compounds, with accuracy lowest for extreme similarity bins

## Limitations

- Some fingerprint features may simply not be represented in any of the fragments, capping achievable prediction accuracy
- No consensus on how to best quantify chemical relatedness — results depend on the fingerprint/label choice (Daylight vs. Morgan variants)
- GNPS-derived spectral libraries are biased toward certain compound classes, so labels are unbalanced across chemical space; mixed instrumentation and varying spectrum quality limit the maximum achievable precision

## Evidence

- [results] topological fingerprints: "circular fingerprints (we used Morgan-2 and Morgan-3, the RDKit [23] pendants of ECFP-4 and ECFP-6) as well as topological fingerprints (we used Daylight fingerprints in RDKit)"
- [discussion] split by unique molecules: "The set of 15,062 InChIKeys was split into a training (n = 14,062), validation (n = 500), and test set (n = 500)."
- [discussion] annotated dataset source: "The dataset was retrieved from GNPS (25/01/2021) and contains a total of 210,407 MS/MS spectra."
- [discussion] structure recovery: "We then ran an automated search against PubChem [42] using pubchempy [43]"
- [readme] Tanimoto training target: "`ms2deepscore` provides a Siamese neural network that is trained to predict molecular structural similarities (Tanimoto scores) 
from pairs of mass spectrometry spectra."
