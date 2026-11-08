---
name: libsvm-bit-prediction-training
description: Use when you need to train or swap per-bit libSVM classifiers that predict
  molecular fingerprints from tandem MS spectra within ChemDistiller's FingerScorer
  scoring pipeline.
when_to_use_negative:
- Input is already a scored candidate list — this skill trains/predicts fingerprints
  upstream of scoring, not the ranking itself
- You only need database searching by MS1 m/z tolerance (--delta_mz); that path does
  not require SVM fingerprint prediction
- Your fingerprint space includes bits not in the 2500-bit occupied set — models are
  only defined for occupied bits
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: ChemDistiller
  role: Engine whose FingerScorer uses the per-bit libSVM models to predict fingerprints
    from tandem MS spectra and rank candidates by Jaccard similarity
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: NumPy
  role: Required dependency for handling spectra feature vectors and bit-vector representations
- name: SciPy
  role: Required dependency for the annotation engine that consumes the SVM models
- name: h5py
  role: Reads the HDF5-format compound databases (*.h5) holding pre-calculated fingerprints
    used for Jaccard ranking
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:e7c66c9b425f17dc7b299f9a002e6a7288896e1a4aebe22bb5f4c63484a7fc46
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDistiller
  - NumPy
  - SciPy
  - h5py
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# libsvm-bit-prediction-training

## Summary

Trains one binary libSVM model per occupied fingerprint bit on merged tandem MS spectra features, so that FingerScorer can predict CDK fingerprint bit vectors from MS/MS data and rank candidate compounds by Jaccard similarity.

## When to use

Use when you need to train or swap per-bit libSVM classifiers that predict molecular fingerprints from tandem MS spectra within ChemDistiller's FingerScorer scoring pipeline.

## When NOT to use

- Input is already a scored candidate list — this skill trains/predicts fingerprints upstream of scoring, not the ranking itself
- You only need database searching by MS1 m/z tolerance (--delta_mz); that path does not require SVM fingerprint prediction
- Your fingerprint space includes bits not in the 2500-bit occupied set — models are only defined for occupied bits

## Inputs

- Merged tandem MS training spectra as feature vectors (chemdistiller.msspectra spectrum/peak/manager modules)
- Pre-calculated CDK fingerprint bit occupancy (2500 occupied bits of 11416-bit union)
- SVM model folders with '1' and '-1' subfolders (Linear_weighted or Radial_weighted, downloaded from the project mediafire links)

## Outputs

- Per-bit binary libSVM models predicting fingerprint bit occupancy from MS/MS features
- Predicted fingerprint bit vectors used by chemdistiller.scorers.fingerprint for Jaccard-similarity ranking of candidate compounds

## How to apply

Load merged MS/MS training spectra as feature vectors via the chemdistiller.msspectra modules (spectrum, peak, manager), restrict targets to the 2500 occupied bits of the 11416-bit CDK fingerprint union (bit-occupancy filter), and train one binary libSVM model per occupied bit. ChemDistiller ships with only a small set of simple SVM models for size reasons; for larger-scale analysis download Linear_weighted or Radial_weighted SVMs — Radial has better prediction power but is larger and slower — and copy the '1' and '-1' subfolders into .../ChemDistiller/SVMs/, deleting both existing subfolders first to avoid confusion. Alternatively point the engine to a custom SVM folder via --svm_folder. The trained/predicted bit vectors are compared against pre-calculated reference fingerprints from the chemdb database, and candidates are ranked by Jaccard similarity.

## Related tools

- **ChemDistiller** (Engine whose FingerScorer uses the per-bit libSVM models to predict fingerprints from tandem MS spectra and rank candidates by Jaccard similarity) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **NumPy** (Required dependency for handling spectra feature vectors and bit-vector representations)
- **SciPy** (Required dependency for the annotation engine that consumes the SVM models)
- **h5py** (Reads the HDF5-format compound databases (*.h5) holding pre-calculated fingerprints used for Jaccard ranking)

## Examples

```
python annotate.py /home/me/my_spectra /home/me/my_results --svm_folder /home/me/my_svms
```

## Evaluation signals

- Every predicted bit falls within the 2500-bit occupied set of the 11416-bit CDK fingerprint union
- Each per-bit libSVM model emits valid binary predictions
- Jaccard rankings are consistent with the reference fingerprints in the chemdb database
- If swapping default SVMs, both '1' and '-1' subfolders under .../ChemDistiller/SVMs/ were replaced together (no mixed sets)
- Spectra processed via annotate.py --test (863 MassBank test spectra with known structures) yield printed retrieval statistics

## Limitations

- The shipped distribution contains only a small set of chemical compound databases and simple SVM models due to size limitations; larger SVMs must be downloaded separately
- Linear SVMs are smaller and faster but have worse prediction power than Radial (which are recommended but larger and slower)
- Mixing SVM subfolders is a failure mode: the README instructs deleting both '1' and '-1' subfolders before copying replacements
- Spectra must be in ChemDistiller's internal hierarchical format; conversions from other formats were not yet available at v0.1

## Evidence

- [other] FingerScorer trains libSVM models on reference spectra to predict molecular fingerprints from tandem MS experimental data; retrieved candidates are then ranked by Jaccard similarity between predicted and actual fingerprints.: [evidence span withheld — anchor status: not_found]
- [readme] SVMs are supplied with Linear_weighted / Radial_weighted variants and folder-swap instructions: [evidence span withheld — anchor status: external_doc]
- [readme] SVM replacement procedure requires deleting both subfolders: [evidence span withheld — anchor status: external_doc]
- [readme] Large-scale SVMs and compound databases must be downloaded separately due to size limits: [evidence span withheld — anchor status: external_doc]
- [abstract] Pre-calculated fingerprints in the compound database are the basis of the database of tens of millions of compounds: "combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints'"
