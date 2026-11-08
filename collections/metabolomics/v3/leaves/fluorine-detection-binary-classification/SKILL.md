---
name: fluorine-detection-binary-classification
description: Use when tandem MS/MS spectra (positive mode, singly charged) must be
  screened for fluorine-containing molecules, and a DreaMS embedding-based classifier
  is preferred over isotope/annotation tools like SIRIUS.
when_to_use_negative:
- The task is molecular structure identification/annotation (fingerprint retrieval,
  formula prediction) rather than a single binary chemical-property label
- Spectra are in negative ionization mode or have multiply charged precursor ions
  — outside DreaMS's demonstrated scope
- Inputs already include confident structure annotations from spectral libraries,
  making the classifier redundant
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: DreaMS
  role: Pre-trained 116M transformer providing MS/MS embeddings and the base network
    fine-tuned with a binary classification head for fluorine detection
  repo: https://github.com/pluskal-lab/DreaMS
- name: SIRIUS
  role: Baseline binary fluorine-classification method (v5.6.3) whose precision is
    compared against DreaMS
- name: MoNA
  role: Training spectral library used for fine-tuning the fluorine classifier
- name: NIST20
  role: Training spectral library used for fine-tuning the fluorine classifier
- name: HDF5
  role: Compact binary storage format for spectra and LC-MS/MS metadata used by the
    DreaMS package
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:3b4ce40a822dc32e215aab31aa92cab05cf71c65d08668ceaf77b54bb96fb5d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pluskal-lab/DreaMS
  tools:
  - DreaMS
  - HDF5
  - MoNA
  - NIST20
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# fluorine-detection-binary-classification

## Summary

Fine-tune the pre-trained 116M-parameter DreaMS transformer as a binary fluorine-containing vs. non-fluorinated classifier from tandem mass spectra, evaluating against the SIRIUS baseline on a large in-house test set.

## When to use

Use when tandem MS/MS spectra (positive mode, singly charged) must be screened for fluorine-containing molecules, and a DreaMS embedding-based classifier is preferred over isotope/annotation tools like SIRIUS.

## When NOT to use

- The task is molecular structure identification/annotation (fingerprint retrieval, formula prediction) rather than a single binary chemical-property label
- Spectra are in negative ionization mode or have multiply charged precursor ions — outside DreaMS's demonstrated scope
- Inputs already include confident structure annotations from spectral libraries, making the classifier redundant

## Inputs

- LC-MS/MS tandem mass spectra (positive ionization mode, singly charged precursors) in DreaMS HDF5 format, e.g. MoNA, NIST20 training libraries
- In-house fluorine test set: 17,052 spectra from MassIVE accession MSV000094528
- Pre-trained DreaMS checkpoint (116M parameters, Zenodo record 10997887)

## Outputs

- Binary fluorine-detection predictions with probability scores for each test spectrum
- Per-threshold precision/recall metrics; summary table (DreaMS precision 0.91 at recall 0.57; SIRIUS precision ≤ 0.51)
- Coverage statistic: fraction of predictions classifiable as certain at a 90% precision threshold (95% of DreaMS predictions)
- fluorine_detection_results.csv with per-model comparison against SIRIUS

## How to apply

Load spectral libraries and the test set from MassIVE accession MSV000094528 (17,052 in-house spectra) using DreaMS data-loading utilities (dreams.utils.data), storing spectra in DreaMS's compact HDF5 binary format. Build Murcko-histogram-disjoint train/val/test splits so no Murcko scaffold is shared between training and evaluation molecules, which tests generalization to structurally unseen chemistry. Initialize from the pre-trained DreaMS checkpoint (116M-parameter transformer, Zenodo record 10997887), attach a binary classification head for fluorine detection, and train with focal binary cross-entropy (α=0.8, γ=0.5) on the MoNA/NIST20 training split to handle class imbalance. Select the checkpoint on the validation split, run inference on the test set to obtain probability scores, and sweep the decision threshold to compute precision/recall plus the coverage at a 90% precision threshold; compare against the SIRIUS 5.6.3 baseline.

## Related tools

- **DreaMS** (Pre-trained 116M transformer providing MS/MS embeddings and the base network fine-tuned with a binary classification head for fluorine detection) — https://github.com/pluskal-lab/DreaMS
- **SIRIUS** (Baseline binary fluorine-classification method (v5.6.3) whose precision is compared against DreaMS)
- **MoNA** (Training spectral library used for fine-tuning the fluorine classifier)
- **NIST20** (Training spectral library used for fine-tuning the fluorine classifier)
- **HDF5** (Compact binary storage format for spectra and LC-MS/MS metadata used by the DreaMS package)

## Examples

```
from dreams.utils.data import load_spectra; spectra = load_spectra('MSV000094528.hdf5'); model.predict(spectra)  # binary fluorine probabilities
```

## Evaluation signals

- Test set is Murcko-histogram-disjoint from training/val: no shared Murcko scaffolds between splits
- Test set size matches 17,052 spectra from MSV000094528
- DreaMS precision ≈ 0.91 at recall ≈ 0.57 on the in-house test set; SIRIUS does not exceed precision 0.51
- 95% of DreaMS predictions fall above the 90%-precision certainty threshold
- Predictions table saved as fluorine_detection_results.csv with DreaMS-vs-SIRIUS comparison rows

## Limitations

- Only positive-mode, singly charged spectra were demonstrated; negative mode and multiply charged precursors are untested scope extensions
- The task disregards MS1 isotopic patterns and adduct distributions, which could otherwise help fluorine detection
- Recall at high precision is moderate (0.57), so low-confidence spectra should be filtered by the 90%-precision threshold rather than treated as definitive
- Spectrum-based prediction is a proof of concept; structural inference tools may offer complementary evidence

## Evidence

- [abstract] Fluorine detection results: "Whereas SIRIUS does not exceed a precision value of 0.51, DreaMS achieves a precision of 0.91 with a recall of 0.57"
- [methods] Training and evaluation protocol: [evidence span withheld — anchor status: not_found]
- [methods] Data storage: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
- [abstract] Pre-training: "large self-supervised neural network (with 116 million parameters) trained directly on the repository-scale collection of raw experimental mass spectra"
