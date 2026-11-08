---
name: svm-metabolite-classifier-training
description: Use when compound databases with pre-calculated fingerprints must be
  classified as metabolite-like vs non-metabolite-like, or when the FingerScorer SVM
  for ChemDistiller annotation must be (re)trained or benchmarked.
when_to_use_negative:
- The goal is annotating tandem MS spectra rather than training/evaluating the metabolite-likeness
  classifier — use the annotate.py annotation pipeline instead
- Input spectra are not in ChemDistiller's internal hierarchical format (format conversion
  was not yet implemented in v0.1)
- Compounds lack pre-calculated fingerprints, so no feature vectors exist for SVM
  training or scoring
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0157
tools:
- name: ChemDistiller
  role: Source package providing the fingerprint compound database and the SVM scoring
    machinery used for training and metabolite-likeness prediction
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: Python 2 and 3
  role: Runtime environment (64-bit recommended) in which the SVM training and evaluation
    scripts are executed
- name: NumPy
  role: Numerical library required by ChemDistiller, used for fingerprint vector handling
    during SVM training
- name: SciPy
  role: Required scientific library for the SVM/scoring computations
- name: RDKit
  role: Optional; used to generate 2D chemical structure images for the annotation
    report
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:6c9d5c7c76a86333810e9e553b7666614b9a3e640c6c2b1148d39c2b3c1b278f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDistiller
  - NumPy
  - Python
  - Python 2 and 3
  - RDKit
  - SciPy
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# svm-metabolite-classifier-training

## Summary

Trains ChemDistiller's two-class metabolite-likeness SVM on pre-calculated compound fingerprints (metabolites from HMDB/ChEBI/LipidMaps/YMDB vs ZINC non-metabolites) and applies it to estimate the metabolite-like fraction of each compound database.

## When to use

Use when compound databases with pre-calculated fingerprints must be classified as metabolite-like vs non-metabolite-like, or when the FingerScorer SVM for ChemDistiller annotation must be (re)trained or benchmarked.

## When NOT to use

- The goal is annotating tandem MS spectra rather than training/evaluating the metabolite-likeness classifier — use the annotate.py annotation pipeline instead
- Input spectra are not in ChemDistiller's internal hierarchical format (format conversion was not yet implemented in v0.1)
- Compounds lack pre-calculated fingerprints, so no feature vectors exist for SVM training or scoring

## Inputs

- Pre-calculated fingerprint compound database in HDF5 (*.h5) format from ChemDistiller's DBs folder
- Positive-class fingerprint vectors: metabolite compounds from HMDB, ChEBI, LipidMaps, YMDB
- Negative-class fingerprint vectors: ZINC non-metabolite compounds

## Outputs

- Serialized trained SVM model (metabolite_likeness_svm_model.pkl)
- Cross-validation accuracy and per-database metabolite-likeness fractions (svm_cv_metrics.json)
- Per-database predicted metabolite-like fractions (HMDB/LipidMaps/YMDB >90%, PubChem ~35%, ZINC ~20%)

## How to apply

Obtain ChemDistiller from the Bitbucket repository, which provides the pre-calculated fingerprint compound database. Load fingerprint vectors for the positive class (metabolites from HMDB, ChEBI, LipidMaps, YMDB) and negative class (ZINC non-metabolites) via the chemdistiller.chemdb.database module, assigning binary labels. Train the two-class SVM with the chemdistiller.scorers.fingerprint machinery under Python 2/3 (64-bit) with NumPy and SciPy, evaluating with 5-fold cross-validation against the reported ~95% accuracy. Then apply the trained model to each compound database and compute the fraction predicted metabolite-like. When only pre-built SVMs are used, pick Linear_weighted (smaller, faster, weaker) vs Radial_weighted (better, recommended) models and supply them via --svm_folder.

## Related tools

- **ChemDistiller** (Source package providing the fingerprint compound database and the SVM scoring machinery used for training and metabolite-likeness prediction) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **Python 2 and 3** (Runtime environment (64-bit recommended) in which the SVM training and evaluation scripts are executed)
- **NumPy** (Numerical library required by ChemDistiller, used for fingerprint vector handling during SVM training)
- **SciPy** (Required scientific library for the SVM/scoring computations)
- **RDKit** (Optional; used to generate 2D chemical structure images for the annotation report)

## Examples

```
python annotate.py /home/me/my_spectra /home/me/my_results --svm_folder /home/me/my_svms
```

## Evaluation signals

- Reproduced 5-fold cross-validation accuracy matches the reported ~95%
- Predicted metabolite-like fractions exceed 90% for HMDB, YMDB and LipidMaps
- Predicted metabolite-like fraction is ~35% for PubChem and ~20% for ZINC
- Trained model serializes to metabolite_likeness_svm_model.pkl and metrics to svm_cv_metrics.json without error
- ChemDistiller --test run on the 863 MassBank test spectra completes with retrieval statistics printed

## Limitations

- Linear SVMs are smaller and faster but have worse prediction power than Radial_weighted SVMs, which are larger and slower
- The default distribution ships only a small set of compound databases and simple SVM models due to size limitations; larger databases/SVMs must be downloaded separately
- When replacing default SVMs, both '1' and '-1' subfolders in .../ChemDistiller/SVMs/ must be deleted to avoid confusion

## Evidence

- [abstract] combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints': "combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints'"
- [methods] chemdistiller.chemdb.database module: "chemdistiller.chemdb.database module
------------------------------------

.. automodule:: chemdistiller.chemdb.database"
- [readme] You can choose Linear_weighted (already installed) or Radial_weighted SVMs here. Linear are smaller, faster, but prediction power is worse than for Radial.: [evidence span withheld — anchor status: not_found]
- [intro] ChemDistiller supports Python 2 and 3 (64-bit version recommended) and requires SciPy, NumPy, h5py libraries.: "__ChemDistiller__ supports Python 2 and 3 (64-bit version recommended) and requires _SciPy_, _NumPy_, _h5py_ libraries."
