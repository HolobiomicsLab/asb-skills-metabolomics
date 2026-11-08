---
name: focal-binary-cross-entropy-fine-tuning-of-transformer
description: Use when adapting the pre-trained DreaMS checkpoint (Zenodo 10997887)
  to a binary spectral-property prediction task (e.g. fluorine detection) with class
  imbalance where precision, not just accuracy, is the metric of record.
when_to_use_negative:
- The task is spectral similarity retrieval or fingerprint prediction rather than
  a binary property classification — DreaMS is fine-tuned separately per task and
  the MIST CANOPUS-style retrieval benchmarks do not use focal BCE
- The test molecules share Murcko scaffolds with training molecules — the Murcko-histogram-disjoint
  split is required for honest generalization estimates
- The spectra are in negative ionization mode or multiply charged precursor ions —
  the DreaMS pre-training corpus covered positive-mode, singly charged spectra only
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_3474
tools:
- name: DreaMS
  role: 116M-parameter transformer pre-trained self-supervised on GeMS mass spectra;
    its checkpoint is fine-tuned with focal binary cross-entropy for fluorine detection,
    and dreams.utils.data provides the spectral data loading
  repo: https://github.com/pluskal-lab/DreaMS
- name: SIRIUS 5.6.3
  role: State-of-the-art baseline for fluorine detection whose precision (≤ 0.51)
    the fine-tuned DreaMS classifier is compared against
- name: MoNA
  role: Standard training spectral library whose paired spectra are used for supervised
    fine-tuning with focal BCE loss
- name: NIST20
  role: Standard training spectral library providing additional fine-tuning spectra
    alongside MoNA
- name: HDF5
  role: Compact binary storage format for the GeMS spectra and metadata consumed during
    fine-tuning
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:f71c5bc85e46d95a8885a355714491aed8e8eb0b37ebaa094c01d197ca27b7ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pluskal-lab/DreaMS
  tools:
  - DreaMS
  - HDF5
  - MoNA
  - NIST20
  - SIRIUS 5.6.3
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# focal binary cross-entropy fine-tuning of transformer

## Summary

Fine-tune the 116M-parameter DreaMS transformer by attaching a binary classification head and training with focal binary cross-entropy loss (α=0.8, γ=0.5) to detect a chemical property — fluorine — from tandem mass spectra, where positives are rare and precision matters.

## When to use

Use when adapting the pre-trained DreaMS checkpoint (Zenodo 10997887) to a binary spectral-property prediction task (e.g. fluorine detection) with class imbalance where precision, not just accuracy, is the metric of record.

## When NOT to use

- The task is spectral similarity retrieval or fingerprint prediction rather than a binary property classification — DreaMS is fine-tuned separately per task and the MIST CANOPUS-style retrieval benchmarks do not use focal BCE
- The test molecules share Murcko scaffolds with training molecules — the Murcko-histogram-disjoint split is required for honest generalization estimates
- The spectra are in negative ionization mode or multiply charged precursor ions — the DreaMS pre-training corpus covered positive-mode, singly charged spectra only

## Inputs

- MoNA and NIST20 training spectral libraries (tandem MS/MS spectra in DreaMS compact HDF5-based binary format)
- In-house fluorine test set of 17,052 spectra (MassIVE accession MSV000094528)
- Pre-trained DreaMS checkpoint (116M-parameter transformer, Zenodo record 10997887)
- Murcko-histogram-disjoint train/val/test split assignment

## Outputs

- Fine-tuned DreaMS binary classifier checkpoint selected on the validation split
- Per-spectrum fluorine-detection probability scores for the 17,052-spectrum test set
- Threshold-swept precision/recall curves and summary metrics (DreaMS precision 0.91 / recall 0.57; comparison against SIRIUS baseline precision ≤ 0.51)
- Coverage statistics at 90% precision threshold (95% of predictions classifiable as certain)

## How to apply

Load MoNA and NIST20 training spectral libraries plus the in-house test set (MassIVE MSV000094528, 17,052 spectra) with the DreaMS package's data-loading utilities (dreams.utils.data), stored in the compact HDF5-based binary format. Construct Murcko-histogram-disjoint train/val/test splits so training and evaluation molecules share no scaffolds, then initialize from the 116M-parameter pre-trained checkpoint on Zenodo (record 10997887) and attach a binary classification head for fluorine detection. Train on the MoNA/NIST20 split with focal binary cross-entropy using α=0.8 and γ=0.5 — the focal down-weighting addresses the imbalance between fluorinated and non-fluorinated spectra — and select the checkpoint on the validation split. Run inference on the 17,052-spectrum test set to obtain per-spectrum probability scores, then sweep the decision threshold to report precision, recall, and the fraction of predictions classifiable as certain at a 90% precision threshold.

## Related tools

- **DreaMS** (116M-parameter transformer pre-trained self-supervised on GeMS mass spectra; its checkpoint is fine-tuned with focal binary cross-entropy for fluorine detection, and dreams.utils.data provides the spectral data loading) — https://github.com/pluskal-lab/DreaMS
- **SIRIUS 5.6.3** (State-of-the-art baseline for fluorine detection whose precision (≤ 0.51) the fine-tuned DreaMS classifier is compared against)
- **MoNA** (Standard training spectral library whose paired spectra are used for supervised fine-tuning with focal BCE loss)
- **NIST20** (Standard training spectral library providing additional fine-tuning spectra alongside MoNA)
- **HDF5** (Compact binary storage format for the GeMS spectra and metadata consumed during fine-tuning)

## Examples

```
from dreams.utils.data import load_spectra; spectra = load_spectra('MSV000094528.hdf5')  # fine-tune DreaMS checkpoint with focal BCE (α=0.8, γ=0.5) for fluorine detection
```

## Evaluation signals

- Test-set precision at recall 0.57 should reach 0.91 for the fine-tuned DreaMS classifier, versus SIRIUS precision not exceeding 0.51
- At least 95% of test predictions should fall above a decision threshold that maintains 90% precision (certain predictions)
- Sweeping the decision threshold yields a monotone precision-recall trade-off; the reported operating point (precision 0.91 / recall 0.57) must come from that sweep, not a default 0.5 cutoff
- No Murcko scaffold overlap between train and test molecules — the Murcko-histogram disjunction must be verified before metrics are quoted
- Validation-split checkpoint selection must precede any test-set scoring; test-set metrics are computed once on the 17,052 held-out spectra

## Limitations

- Focal BCE fine-tuning is demonstrated for fluorine detection as a property-prediction task; extending it to other chemical properties or fluorine analogs has not been validated
- The underlying DreaMS model disregards MS1 isotopic patterns and adduct distributions, and only positive-mode, singly charged spectra were used in pre-training
- Only a subset of available mass spectra was used for training; the full potential of training on raw unannotated spectra remains to be unlocked
- The 17,052-spectrum test set is in-house data; precision values may shift on externally sourced fluorine test spectra

## Evidence

- [abstract] Whereas SIRIUS does not exceed a precision value of 0.51, DreaMS achieves a precision of 0.91 with a recall of 0.57: "Whereas SIRIUS does not exceed a precision value of 0.51, DreaMS achieves a precision of 0.91 with a recall of 0.57"
- [abstract] fine-tuned for binary classification on the task of identification of fluorine-containing molecules, we adapted the network to the prediction of ... chemical properties and the identification of fluorine-containing molecules: "we adapted the network to the prediction of spectral similarity, molecular fingerprints, chemical properties and the identification of fluorine-containing molecules"
- [abstract] 116M-parameter transformer pre-trained self-supervised on unannotated MS/MS spectra: "Here we introduce a large self-supervised neural network (with 116 million parameters) trained directly on the repository-scale collection of raw experimental mass spectra"
- [abstract] standard training spectral libraries MoNA and NIST20: "the standard training spectral libraries MoNA36 and NIST20 (ref. 37) cover only a limited subset of known natural molecules"
- [discussion] The dataset is available under MassIVE accession number MSV000094528: "The dataset is available under MassIVE accession number MSV000094528"
- [abstract] we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
- [abstract] DreaMS representations robust to mass spectrometry conditions: "the DreaMS representations (1,024-dimensional real-valued vectors) are organized according to the structural similarity between molecules and are robust to mass spectrometry conditions"
