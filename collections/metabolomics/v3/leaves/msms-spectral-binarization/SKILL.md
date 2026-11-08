---
name: msms-spectral-binarization
description: 'Use when preparing MS/MS spectra for per-fragment binary prediction:
  instead of regressing peak intensities, spectra are binarized by m/z bin and a separate
  model predicts presence/absence per bin.'
when_to_use_negative:
- The goal is to predict quantitative peak intensities or full high-resolution spectra
  rather than peak presence/absence — use an intensity-regression model such as 3DMolMS
  instead
- Only integer-mass spectra are available (e.g. HMDB spectra listed at whole-number
  m/z); SingleFrag needs 2-decimal fragment m/z resolution
- Spectra were acquired in negative mode or with precursor adducts outside the supported
  set — binarization alone does not fix adduct/charge mismatch
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: SingleFrag
  role: Deep learning framework in which spectra are binarized per m/z bin and a separate
    classifier is trained per bin; provides ANNmodel.py, GNNmodel.py, COMmodel.py
    and per-bin threshold pickles
  repo: https://github.com/MaribelPR/SingleFrag
- name: mol2vec
  role: Generates the 300-dimensional molecular embedding that serves as input to
    the per-bin ANN binary classifiers after binarization
- name: 3DMolMS
  role: 'Contrast/reference approach: predicts full MS/MS spectra from 3D conformations
    rather than binarized per-fragment labels'
  repo: https://github.com/JosieHong/3DMolMS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:08cd71e752b09c64f2faf114b6c1cc12681104b1cc9668aad303b240865482c7
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - 3DMolMS
  - SingleFrag
  - mol2vec
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# MS/MS spectral binarization

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Converts tandem MS spectra into per-m/z-bin binary labels (peak present/absent) so that a separate binary classifier can be trained for each of the 1,000 most frequented m/z positions, as done in SingleFrag.

## When to use

Use when preparing MS/MS spectra for per-fragment binary prediction: instead of regressing peak intensities, spectra are binarized by m/z bin and a separate model predicts presence/absence per bin.

## When NOT to use

- The goal is to predict quantitative peak intensities or full high-resolution spectra rather than peak presence/absence — use an intensity-regression model such as 3DMolMS instead
- Only integer-mass spectra are available (e.g. HMDB spectra listed at whole-number m/z); SingleFrag needs 2-decimal fragment m/z resolution
- Spectra were acquired in negative mode or with precursor adducts outside the supported set — binarization alone does not fix adduct/charge mismatch

## Inputs

- MS/MS spectra in positive mode with fragment m/z values at 2-decimal resolution (e.g. SMILES-linked spectra from HMDB or the SingleFrag Figshare deposit)
- Molecule identifiers/SMILES paired with each experimental spectrum
- Training/validation/test split prepared by the authors in the SingleFrag Figshare dataset

## Outputs

- Binary peak-presence label per m/z bin per molecule (1,000 most frequented bins)
- Frequency table of peaks per m/z bin (AllMostFreqMolGeneral_rep_dades1.pickle)
- Per-bin decision threshold files (thresholdsANN.pickle, thresholdsGNN.pickle, thresholdsCOM.pickle)

## How to apply

Rather than predicting the full spectrum or peak intensities, assign each observed peak to an m/z bin (mz*100 encoding used in model file names) and label it present (1) or absent (0). Rank all m/z bins by how frequently peaks occur across the training set — SingleFrag uses the 1,000 most frequented positions, stored in AllMostFreqMolGeneral_rep_dades1.pickle — and restrict modeling to those bins. The output is one binary label per bin per molecule, which feeds per-bin ANN (mol2vec 300-dim embedding input), GNN (molecular graph input), or combined models, each trained with its own decision threshold calibrated on validation spectra. This framing is deliberately intensity-free so the classifier is robust across collision energies and instruments, unlike intensity-regression approaches such as 3DMolMS.

## Related tools

- **SingleFrag** (Deep learning framework in which spectra are binarized per m/z bin and a separate classifier is trained per bin; provides ANNmodel.py, GNNmodel.py, COMmodel.py and per-bin threshold pickles) — https://github.com/MaribelPR/SingleFrag
- **mol2vec** (Generates the 300-dimensional molecular embedding that serves as input to the per-bin ANN binary classifiers after binarization)
- **3DMolMS** (Contrast/reference approach: predicts full MS/MS spectra from 3D conformations rather than binarized per-fragment labels) — https://github.com/JosieHong/3DMolMS

## Examples

```
python ANNmodel.py --mz_position 4500  # train the per-bin binary classifier for m/z 45.00; repeat across the 1,000 most frequented bins
```

## Evaluation signals

- The frequency table (AllMostFreqMolGeneral_rep_dades1.pickle) should contain exactly the 1,000 most frequented m/z bins from the training set
- Each bin's label distribution should retain both classes; a bin with zero positives in training cannot be trained as a binary classifier
- Per-bin validation AUC/threshold calibration should yield thresholds files that reproduce the reported mean F1 of 0.411 (ANN), 0.399 (combined), and 0.290 (GNN) on the 10% test split
- Binarized predictions above a bin's threshold should align with peaks observed at that m/z in the experimental spectrum (within 2-decimal m/z tolerance)

## Limitations

- Intensity information is discarded by design, so binarized models cannot rank relative fragment abundance or reproduce intensity-dependent spectral similarity
- Only the 1,000 most frequented m/z bins are modeled; rare fragments outside these bins are never predicted
- Peak presence can depend on collision energy and instrument type; binarization mitigates but does not eliminate cross-instrument variance

## Evidence

- [readme] Models are per-MZ-position binary predictors: [evidence span withheld — anchor status: not_found]
- [other] Mean F1 across the 1,000 bins is the evaluation metric: [evidence span withheld — anchor status: not_found]
