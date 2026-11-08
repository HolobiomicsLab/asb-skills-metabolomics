---
name: cheminformatics-fingerprint-generation
description: Use when a compound database must be featurized into fingerprint vectors
  for ChemDistiller's FingerScorer SVM — e.g. reproducing the metabolite-likeness
  classifier or rebuilding/repackaging an HDF5 compound database.
when_to_use_negative:
- Input is raw tandem MS spectra to be annotated — use `python annotate.py` on the
  existing pre-calculated fingerprint databases rather than regenerating fingerprints
- The compound database already ships with pre-calculated fingerprints in HDF5 (*.h5)
  format downloaded from the ChemDistiller mediafire DBs folder — reuse rather than
  recompute
- Structures lack a parseable chemical representation (no InChI/formula), so fingerprint
  calculation cannot proceed without prior curation
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: ChemDistiller
  role: Provides the chemdb.database module for loading pre-calculated fingerprint
    compound databases and the fingerprint scoring machinery (SVM subfolders '1' and
    '-1') that consumes the vectors
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: NumPy
  role: Required library for manipulating fingerprint feature vectors used to train
    the SVM
- name: SciPy
  role: Required library underlying the fingerprint/SVM computations in ChemDistiller
- name: RDKit
  role: Optional ChemDistiller dependency, used to generate 2D chemical structure
    images for the annotation report
  repo: http://www.rdkit.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:37ea50835e9052dddb5bd1ec14151b3c8f21bd327764de364d3e45cfddf48692
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
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cheminformatics-fingerprint-generation

## Summary

Generates pre-calculated molecular fingerprint vectors for chemical compound databases (HMDB, ChEBI, LipidMaps, YMDB, PubChem, ZINC) so they can be scored by ChemDistiller's fingerprint-based SVM machinery, e.g. for the metabolite-likeness two-class classifier.

## When to use

Use when a compound database must be featurized into fingerprint vectors for ChemDistiller's FingerScorer SVM — e.g. reproducing the metabolite-likeness classifier or rebuilding/repackaging an HDF5 compound database.

## When NOT to use

- Input is raw tandem MS spectra to be annotated — use `python annotate.py` on the existing pre-calculated fingerprint databases rather than regenerating fingerprints
- The compound database already ships with pre-calculated fingerprints in HDF5 (*.h5) format downloaded from the ChemDistiller mediafire DBs folder — reuse rather than recompute
- Structures lack a parseable chemical representation (no InChI/formula), so fingerprint calculation cannot proceed without prior curation

## Inputs

- Parsed compound structures/identifiers from source databases (HMDB, ChEBI, LipidMaps, YMDB as metabolites; ZINC as non-metabolites; PubChem)
- Pre-calculated fingerprint compound database in HDF5 format (*.h5) from ChemDistiller/DBs/
- Binary class labels (metabolite = 1, non-metabolite = -1) for supervised fingerprint scoring

## Outputs

- Per-compound fingerprint feature vectors aligned with the compound database entries
- Trained two-class SVM serialized as metabolite_likeness_svm_model.pkl
- Cross-validation accuracy and per-database metabolite-likeness fractions written to svm_cv_metrics.json
- Repackaged compound database in HDF5 (*.h5) format for ChemDistiller/DBs/

## How to apply

Obtain the pre-calculated fingerprint compound database from the ChemDistiller Bitbucket repository (https://bitbucket.org/iAnalytica/chemdistillerpython) or regenerate fingerprints for source compound databases using the chemdistiller.chemdb.database module with NumPy/SciPy. Load fingerprint vectors for the positive class (metabolites from HMDB, ChEBI, LipidMaps, YMDB) and negative class (ZINC non-metabolites), assigning binary labels (1 / -1, matching the '1' and '-1' subfolder convention ChemDistiller uses for its SVMs). Choose the SVM type per the README: 'Linear_weighted' SVMs are smaller and faster but have worse prediction power, while 'Radial_weighted' SVMs are recommended despite being larger and slower; when replacing default SVMs, delete both '1' and '-1' subfolders in .../ChemDistiller/SVMs/ before copying the new ones. Train and evaluate with 5-fold cross-validation, targeting the reported ~95% accuracy, then apply the trained model per database. Store the large HDF5 (*.h5) databases in .../ChemDistiller/DBs/ so they are picked up automatically on the next run.

## Related tools

- **ChemDistiller** (Provides the chemdb.database module for loading pre-calculated fingerprint compound databases and the fingerprint scoring machinery (SVM subfolders '1' and '-1') that consumes the vectors) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **NumPy** (Required library for manipulating fingerprint feature vectors used to train the SVM)
- **SciPy** (Required library underlying the fingerprint/SVM computations in ChemDistiller)
- **RDKit** (Optional ChemDistiller dependency, used to generate 2D chemical structure images for the annotation report) — http://www.rdkit.org/

## Examples

```
from chemdistiller.chemdb import database; from chemdistiller.scorers import fingerprint; X, y = database.load_labels(labels=(1,-1))  # task_006 workflow
```

## Evaluation signals

- 5-fold cross-validation of the fingerprint-based two-class SVM reaches the reported ~95% classification accuracy
- Applying the trained model per database yields >90% metabolite-like entries for HMDB, YMDB and LipidMaps, ~35% for PubChem, and ~20% for ZINC
- Fingerprint vectors have a consistent length across all database entries and carry labels matching the '1' and '-1' SVM subfolder convention
- The repackaged HDF5 (*.h5) database placed in .../ChemDistiller/DBs/ is picked up automatically and `python annotate.py --test` (863 MassBank test spectra) prints retrieval statistics
- Serialized model (metabolite_likeness_svm_model.pkl) reloads and reproduces the same per-database fractions on a held-out re-run

## Limitations

- ChemDistiller ships with only a small set of chemical compound databases and simple SVM models due to size limitations; larger databases and SVMs must be downloaded separately (mediafire DBs and SVMs folders)
- Linear weighted SVMs are smaller and faster but have worse prediction power than Radial weighted SVMs, which are larger and slower
- Derivative databases may carry individual restrictions on commercial use inherited from the original compound databases — consult the DBs readme and original database disclaimers
- RDKit is optional and only used for 2D structure images in the report, so fingerprint reproducibility does not depend on it being installed

## Evidence

- [abstract] a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation: "a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation"
- [readme] Linear are smaller, faster, but prediction power is worse than for Radial.: [evidence span withheld — anchor status: not_found]
- [readme] Current version of ChemDistiller is supplied with only a small set of chemical compound databases and simple SVM models due to size limitations.: "Current version of __ChemDistiller__ is supplied with only a small set of __chemical compound databases__ and
simple __SVM models__ due to size limitations."
- [intro] requires SciPy, NumPy, h5py libraries.: "requires _SciPy_, _NumPy_, _h5py_ libraries."
