---
name: cross-validation-performance-estimation
description: Use when a fingerprint-based binary classifier (e.g. the metabolite-likeness
  SVM over compounds from HMDB, ChEBI, LipidMaps, YMDB vs ZINC non-metabolites) needs
  an unbiased accuracy estimate before it is applied to score compound databases.
when_to_use_negative:
- The task is tandem MS spectra annotation with the FingerScorer rather than classifying
  whole compound databases as metabolite-like
- Input compounds lack the pre-calculated fingerprints that ChemDistiller's fingerprint
  scoring machinery expects (e.g. raw SMILES without fingerprint generation)
- Only a small set of compound databases/SVMs is installed and large-scale training
  or evaluation is intended — larger databases and SVMs must first be downloaded from
  the mediafire DBs/SVMs folders
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2258
tools:
- name: ChemDistiller
  role: Source of the metabolite-likeness SVM and fingerprint scoring machinery (chemdistiller.scorers.fingerprint)
    used to train and evaluate the classifier
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: Python
  role: Execution environment (Python 2 and 3 supported, 64-bit recommended) in which
    the SVM is trained and 5-fold cross-validation is run
- name: NumPy
  role: Array library for manipulating the labelled fingerprint vectors fed into the
    SVM
- name: SciPy
  role: Required scientific computing dependency for the SVM training/evaluation pipeline
- name: RDKit
  role: Optional ChemDistiller dependency used to generate 2D chemical structure images
    in reports; not required for cross-validation itself
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:0d84a8662866329b9ccb0f6b85d9737b6ae3879cc167ff2b4bab620f4d661251
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDistiller
  - NumPy
  - Python
  - RDKit
  - SciPy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# cross-validation-performance-estimation

## Summary

Estimates the predictive performance of ChemDistiller's two-class metabolite-likeness SVM over pre-calculated compound fingerprint vectors via 5-fold cross-validation, before applying the model to score compound databases as metabolite-like or not.

## When to use

Use when a fingerprint-based binary classifier (e.g. the metabolite-likeness SVM over compounds from HMDB, ChEBI, LipidMaps, YMDB vs ZINC non-metabolites) needs an unbiased accuracy estimate before it is applied to score compound databases.

## When NOT to use

- The task is tandem MS spectra annotation with the FingerScorer rather than classifying whole compound databases as metabolite-like
- Input compounds lack the pre-calculated fingerprints that ChemDistiller's fingerprint scoring machinery expects (e.g. raw SMILES without fingerprint generation)
- Only a small set of compound databases/SVMs is installed and large-scale training or evaluation is intended — larger databases and SVMs must first be downloaded from the mediafire DBs/SVMs folders

## Inputs

- Pre-calculated compound fingerprint vectors from ChemDistiller's compound database (HDF5 *.h5 format from the ChemDistiller DBs download)
- Positive-class compound lists: HMDB, ChEBI, LipidMaps, YMDB metabolites
- Negative-class compound list: ZINC non-metabolites
- SVM model folder (Linear_weighted or Radial_weighted, subfolders '1' and '-1')

## Outputs

- 5-fold cross-validation accuracy metric (~95% expected)
- Trained two-class metabolite-likeness SVM model (serialized e.g. metabolite_likeness_svm_model.pkl)
- Cross-validation accuracy and per-database metabolite-likeness fractions (e.g. svm_cv_metrics.json)

## How to apply

Load pre-calculated fingerprint vectors for the positive class (metabolite compounds from HMDB, ChEBI, LipidMaps and YMDB) and the negative class (ZINC non-metabolites) using the chemdistiller.chemdb.database module, and assign binary metabolite/non-metabolite labels. Train the two-class SVM with ChemDistiller's fingerprint scoring machinery (chemdistiller.scorers.fingerprint) under Python with NumPy and SciPy, choosing Linear_weighted (smaller, faster, weaker) or Radial_weighted (larger, slower, more accurate) SVMs per the README. Partition the labelled fingerprint vectors into 5 folds, holding out each fold in turn to estimate cross-validation accuracy, targeting the reported ~95% classification accuracy. Only once CV accuracy is acceptable, apply the trained model to each compound database and compute the fraction of entries predicted metabolite-like; rationalise model choice against the Linear/Radial accuracy-size trade-off described in the README.

## Related tools

- **ChemDistiller** (Source of the metabolite-likeness SVM and fingerprint scoring machinery (chemdistiller.scorers.fingerprint) used to train and evaluate the classifier) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **Python** (Execution environment (Python 2 and 3 supported, 64-bit recommended) in which the SVM is trained and 5-fold cross-validation is run)
- **NumPy** (Array library for manipulating the labelled fingerprint vectors fed into the SVM)
- **SciPy** (Required scientific computing dependency for the SVM training/evaluation pipeline)
- **RDKit** (Optional ChemDistiller dependency used to generate 2D chemical structure images in reports; not required for cross-validation itself)

## Examples

```
python annotate.py /home/me/my_spectra /home/me/my_results --svm_folder /home/me/my_svms --ncpu 5 --delta_mz 20 --max_results 100
```

## Evaluation signals

- Reproduced 5-fold cross-validation accuracy matches the reported ~95% classification accuracy
- Predicted metabolite-likeness fractions match reported values: >90% for HMDB, LipidMaps and YMDB, ~35% for PubChem, ~20% for ZINC
- Trained SVM serializes successfully to metabolite_likeness_svm_model.pkl and metrics serialize to svm_cv_metrics.json
- SVM folder contains both '1' and '-1' subfolders (positive and negative class models) as required by ChemDistiller
- Class composition check: positive folds drawn from HMDB/ChEBI/LipidMaps/YMDB metabolites, negative folds from ZINC non-metabolites, with no compound leakage between folds

## Limitations

- Linear_weighted SVMs are smaller and faster but have worse prediction power than Radial_weighted SVMs, which are strongly recommended but larger and slower
- ChemDistiller ships by default with only a small set of compound databases and simple SVM models due to size limitations; larger databases and more complex SVMs must be downloaded separately
- Performance estimates are specific to the metabolite vs ZINC non-metabolite class split and the pre-calculated fingerprint representation; the accuracy metric does not report per-class precision/recall
- Derivative compound databases inherit usage restrictions from original databases, so consult the readme.txt in the DBs folder and source-database legal disclaimers before commercial use

## Evidence

- [readme] SVM model choice trade-off: [evidence span withheld — anchor status: external_doc]
- [abstract] Fingerprint database context: "combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation"
- [other] Task-card finding on per-database metabolite-likeness fractions: [evidence span withheld — anchor status: not_found]
- [methods] chemdb.database module used to load fingerprint compound databases: "chemdistiller.chemdb.database module
------------------------------------

.. automodule:: chemdistiller.chemdb.database"
