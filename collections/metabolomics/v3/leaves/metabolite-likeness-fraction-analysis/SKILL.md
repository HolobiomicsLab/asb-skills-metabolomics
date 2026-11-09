---
name: metabolite-likeness-fraction-analysis
description: Use when you must reproduce ChemDistiller's metabolite-likeness SVM filter
  step or estimate what fraction of entries in compound databases (HMDB, LipidMaps,
  YMDB, PubChem, ZINC) is metabolite-like.
when_to_use_negative:
- Input is tandem MS spectra to be annotated — ChemDistiller's main engine (python
  annotate.py) handles spectra annotation; this skill only addresses the metabolite-likeness
  SVM filter over compound databases.
- The task is real-time candidate scoring during annotation runs rather than database-level
  estimation of metabolite-like entry fractions.
- No pre-calculated fingerprint database is available (ChemDistiller supplies only
  a small set due to size limitations, and full databases must be downloaded separately).
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3174
tools:
- name: ChemDistiller
  role: Provides the metabolite-likeness two-class SVM filter machinery, the pre-calculated
    fingerprint compound database, and the chemdistiller.chemdb / chemdistiller.scorers.fingerprint
    modules used to load fingerprints and train/apply the SVM.
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: Python
  role: Runtime for loading fingerprints, training the two-class SVM, running 5-fold
    cross-validation and computing per-database metabolite-like fractions; ChemDistiller
    supports Python 2 and 3 (64-bit recommended).
- name: NumPy
  role: Required library for handling the fingerprint feature vectors used as SVM
    training input.
- name: SciPy
  role: Required library underpinning ChemDistiller's fingerprint scoring and SVM
    computations.
- name: RDKit
  role: Optional dependency of ChemDistiller (used for 2D chemical structure images
    in reports); listed as part of the skill's toolchain.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:e880379c326172cdc505e7b3aad85520906609f8113b1cda421450cff167845b
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# metabolite-likeness-fraction-analysis

## Summary

Reconstructs ChemDistiller's metabolite-likeness two-class SVM filter: fingerprints of known metabolites (HMDB, ChEBI, LipidMaps, YMDB) vs. ZINC non-metabolites train a classifier whose per-database predicted metabolite-like fractions quantify how 'metabolite-like' each compound database is.

## When to use

Use when you must reproduce ChemDistiller's metabolite-likeness SVM filter step or estimate what fraction of entries in compound databases (HMDB, LipidMaps, YMDB, PubChem, ZINC) is metabolite-like.

## When NOT to use

- Input is tandem MS spectra to be annotated — ChemDistiller's main engine (python annotate.py) handles spectra annotation; this skill only addresses the metabolite-likeness SVM filter over compound databases.
- The task is real-time candidate scoring during annotation runs rather than database-level estimation of metabolite-like entry fractions.
- No pre-calculated fingerprint database is available (ChemDistiller supplies only a small set due to size limitations, and full databases must be downloaded separately).

## Inputs

- Pre-calculated fingerprint compound databases in HDF5 (*.h5) format (e.g. HMDB, ChEBI, LipidMaps, YMDB, PubChem, ZINC)
- Positive-class compound labels: metabolite entries from HMDB, ChEBI, LipidMaps, YMDB
- Negative-class compound labels: ZINC non-metabolite entries
- ChemDistiller fingerprint scoring machinery (chemdistiller.scorers.fingerprint) and chemdistiller.chemdb.database module

## Outputs

- Trained two-class metabolite-likeness SVM serialized to metabolite_likeness_svm_model.pkl
- svm_cv_metrics.json containing the 5-fold cross-validation accuracy (~95%) and per-database metabolite-like fractions
- Per-database metabolite-likeness fractions (HMDB, LipidMaps, YMDB, PubChem, ZINC)

## How to apply

Obtain the ChemDistiller source from the Bitbucket repository, which ships pre-calculated fingerprint compound databases in HDF5 (*.h5) format that must be placed in .../ChemDistiller/DBs/. Load pre-calculated fingerprint vectors via the chemdistiller.chemdb.database module, labelling metabolite compounds from HMDB, ChEBI, LipidMaps and YMDB as the positive class and ZINC non-metabolites as the negative class. Train a two-class SVM on the labelled fingerprint vectors using the ChemDistiller fingerprint scoring machinery (chemdistiller.scorers.fingerprint) under Python with NumPy and SciPy, mirroring the package's supplied Linear_weighted and Radial_weighted SVM setups (Radial is recommended but larger and slower). Validate with 5-fold cross-validation against the reported ~95% accuracy, then apply the trained model to every compound database and compute the fraction predicted metabolite-like. Check the fractions against the reported expectations (>90% for HMDB/YMDB/LipidMaps, ~35% for PubChem, ~20% for ZINC), serialize the SVM to metabolite_likeness_svm_model.pkl, and write cross-validation accuracy plus per-database fractions to svm_cv_metrics.json.

## Related tools

- **ChemDistiller** (Provides the metabolite-likeness two-class SVM filter machinery, the pre-calculated fingerprint compound database, and the chemdistiller.chemdb / chemdistiller.scorers.fingerprint modules used to load fingerprints and train/apply the SVM.) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **Python** (Runtime for loading fingerprints, training the two-class SVM, running 5-fold cross-validation and computing per-database metabolite-like fractions; ChemDistiller supports Python 2 and 3 (64-bit recommended).)
- **NumPy** (Required library for handling the fingerprint feature vectors used as SVM training input.)
- **SciPy** (Required library underpinning ChemDistiller's fingerprint scoring and SVM computations.)
- **RDKit** (Optional dependency of ChemDistiller (used for 2D chemical structure images in reports); listed as part of the skill's toolchain.)

## Examples

```
python annotate.py /home/me/my_spectra /home/me/my_results --svm_folder /home/me/my_svms
```

## Evaluation signals

- 5-fold cross-validation accuracy of the reconstructed SVM matches the reported ~95% classification accuracy.
- Predicted metabolite-like fractions per database match reported values: >90% for HMDB, YMDB and LipidMaps, ~35% for PubChem, ~20% for ZINC.
- Serialized model metabolite_likeness_svm_model.pkl reloads and emits binary metabolite/non-metabolite predictions on held-out fingerprint vectors.
- svm_cv_metrics.json parses and contains both the CV accuracy and a per-database fraction entry for each database evaluated.
- Class structure matches ChemDistiller's SVM convention of separate '1' and '-1' class folders/models.

## Limitations

- The distributed version ships only a small set of chemical compound databases and simple SVM models due to size limitations; larger SVMs must be downloaded separately.
- Radial_weighted SVMs predict better but are larger and slower than Linear_weighted ones — a speed/accuracy trade-off when applying or retraining the model.
- ChemDistiller-generated databases and SVMs carry legal caveats: some source databases may restrict commercial use, so the readme's legal disclaimers of the source compound databases must be consulted.
- Replacing the default SVMs requires deleting both '1' and '-1' subfolders in .../ChemDistiller/SVMs/ to avoid confusion.

## Evidence

- [abstract] combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation: "a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation"
- [readme] You can choose _Linear_weighted_ (already installed) or _Radial_weighted_ __SVMs__ here. _Linear_ are smaller, faster, but prediction power is worse than for _Radial_.: [evidence span withheld — anchor status: not_found]
- [readme] When unpacked, __SVMs__ (subfolders "1" and "-1") should be copied into .../ChemDistiller/SVMs/. to be used by default.: [evidence span withheld — anchor status: external_doc]
- [readme] ChemDistiller supports Python 2 and 3 (64-bit version recommended) and requires SciPy, NumPy, h5py libraries.: "requires _SciPy_, _NumPy_, _h5py_ libraries."
