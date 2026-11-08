---
name: metabolite-database-compound-curation
description: Use when you need to quantify what fraction of a compound database (HMDB,
  LipidMaps, YMDB, PubChem, ZINC) is metabolite-like, or to filter annotation candidates
  using the fingerprint-SVM metabolite-likeness scores.
when_to_use_negative:
- Your input is already raw tandem MS spectra needing annotation rather than compound-database
  curation — run the annotate.py annotation pipeline instead
- You need to prioritize annotation candidates by m/z or fragmentation evidence rather
  than fingerprint-based metabolite-likeness (use the delta_mz candidate search and
  scoring filters instead)
- Compound structures/fingerprints are unavailable for your database, since the SVM
  operates on pre-calculated fingerprint vectors
edam_operation: http://edamontology.org/operation_2990
tools:
- name: ChemDistiller
  role: Provides the chemdb.database module for loading pre-calculated fingerprint
    compound databases and the fingerprint scorer/SVM machinery used to train and
    apply the metabolite-likeness classifier
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: Python 2 and 3
  role: Runtime environment (64-bit recommended) in which the SVM training, cross-validation
    and per-database scoring scripts are executed
- name: SciPy
  role: Required dependency for ChemDistiller's fingerprint/SVM numerical computations
    during training and prediction
- name: NumPy
  role: Required dependency for handling fingerprint vector arrays loaded from the
    HDF5 compound databases
- name: RDKit
  role: Optional dependency used to generate 2D chemical structure images of curated
    compounds in annotation reports
  repo: http://www.rdkit.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:8d198182d29700629c6fbbb846eac2ed09463ea149101891b8bcae1d13e8db97
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# metabolite-database-compound-curation

## Summary

Curates and triages ChemDistiller's pre-calculated fingerprint compound databases by training/applying the two-class metabolite-likeness SVM (FingerScorer) so databases and candidate compounds can be labelled metabolite-like vs non-metabolite-like before tandem MS annotation.

## When to use

Use when you need to quantify what fraction of a compound database (HMDB, LipidMaps, YMDB, PubChem, ZINC) is metabolite-like, or to filter annotation candidates using the fingerprint-SVM metabolite-likeness scores.

## When NOT to use

- Your input is already raw tandem MS spectra needing annotation rather than compound-database curation — run the annotate.py annotation pipeline instead
- You need to prioritize annotation candidates by m/z or fragmentation evidence rather than fingerprint-based metabolite-likeness (use the delta_mz candidate search and scoring filters instead)
- Compound structures/fingerprints are unavailable for your database, since the SVM operates on pre-calculated fingerprint vectors

## Inputs

- Pre-calculated fingerprint compound databases in HDF5 (*.h5) format (HMDB, ChEBI, LipidMaps, YMDB, PubChem, ZINC)
- Binary metabolite/non-metabolite class labels for training compounds
- ChemDistiller SVM model folders with '1' and '-1' subfolders (Linear_weighted or Radial_weighted)

## Outputs

- Serialized metabolite-likeness SVM model (e.g. metabolite_likeness_svm_model.pkl)
- Cross-validation accuracy and per-database metabolite-likeness fractions (e.g. svm_cv_metrics.json)
- Filtered/labelled compound databases or candidate lists ranked by metabolite-likeness for tandem MS annotation

## How to apply

Download the ChemDistiller source from the Bitbucket repository and, optionally, the full HDF5 (*.h5) compound databases and Linear_weighted/Radial_weighted SVM folders (Radial is recommended as it has better prediction power, at the cost of size and speed). Load pre-calculated fingerprint vectors from the compound databases via the chemdistiller.chemdb.database module, labelling metabolites (HMDB, ChEBI, LipidMaps, YMDB) as the positive class and ZINC non-metabolites as the negative class. Train the two-class SVM using the chemdistiller.scorers.fingerprint scoring machinery under Python 2/3 with SciPy and NumPy, and evaluate with 5-fold cross-validation against the reported ~95% accuracy. Then apply the trained model to each compound database and compute the metabolite-like fraction, expecting >90% for HMDB/YMDB/LipidMaps, ~35% for PubChem and ~20% for ZINC; serialize the model and metrics (e.g. metabolite_likeness_svm_model.pkl, svm_cv_metrics.json) so the SVM can also be pointed to via --svm_folder during annotate.py runs.

## Related tools

- **ChemDistiller** (Provides the chemdb.database module for loading pre-calculated fingerprint compound databases and the fingerprint scorer/SVM machinery used to train and apply the metabolite-likeness classifier) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **Python 2 and 3** (Runtime environment (64-bit recommended) in which the SVM training, cross-validation and per-database scoring scripts are executed)
- **SciPy** (Required dependency for ChemDistiller's fingerprint/SVM numerical computations during training and prediction)
- **NumPy** (Required dependency for handling fingerprint vector arrays loaded from the HDF5 compound databases)
- **RDKit** (Optional dependency used to generate 2D chemical structure images of curated compounds in annotation reports) — http://www.rdkit.org/

## Examples

```
python annotate.py /home/me/my_spectra /home/me/my_results --svm_folder /home/me/my_svms
```

## Evaluation signals

- 5-fold cross-validation accuracy of the metabolite-likeness SVM reproduces the reported ~95%
- Predicted metabolite-like fraction exceeds 90% for metabolite databases HMDB, LipidMaps and YMDB
- Predicted metabolite-like fraction is approximately 35% for PubChem and approximately 20% for ZINC
- Trained model serializes cleanly (e.g. metabolite_likeness_svm_model.pkl) and reloads to give identical per-database fractions
- When the curated SVMs are used in annotate.py, SVM subfolders '1' and '-1' are both present in the --svm_folder target or default .../ChemDistiller/SVMs/

## Limitations

- The shipped repository contains only a small set of compound databases and simple SVM models due to size limitations; full HDF5 databases and larger SVMs must be downloaded separately
- Linear_weighted SVMs are smaller and faster but have worse prediction power than Radial_weighted SVMs, which are larger and slower
- Some source compound databases have unclear commercial-use restrictions; consult the readme.txt distributed with the DBs folder and cite the original databases and ChemDistiller

## Evidence

- [abstract] database scale with fingerprints: [evidence span withheld — anchor status: not_found]
- [readme] runtime dependencies: "supports Python 2 and 3 (64-bit version recommended) and requires _SciPy_, _NumPy_, _h5py_ libraries."
- [readme] SVM variants available: [evidence span withheld — anchor status: not_found]
- [methods] loading fingerprint databases via package module: "chemdistiller.chemdb.database module ------------------------------------"
- [results] reported per-database metabolite-likeness fractions: [evidence span withheld — anchor status: not_found]
