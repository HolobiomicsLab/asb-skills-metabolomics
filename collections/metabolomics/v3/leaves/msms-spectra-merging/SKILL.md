---
name: msms-spectra-merging
description: Use when tandem MS/MS spectra must be converted into merged training
  feature vectors for per-bit libSVM fingerprint prediction, and candidate compounds
  must be ranked by Jaccard similarity of predicted vs. reference fingerprints from
  chemdb.
when_to_use_negative:
- Input spectra are not in ChemDistiller's internal hierarchical format — format conversion
  was not yet available and other formats require prior conversion.
- The task is element/formula/InChI candidate filtering rather than fingerprint-based
  scoring — use the chemdistiller.filters.element, filters.formula, or filters.inchi
  modules instead.
- Only 1D MS1 peak detection is needed without tandem MS/MS annotation — the FingerScorer
  depends on MS/MS spectra as feature vectors.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ChemDistiller
  role: Annotation engine hosting the msspectra (spectrum, peak, manager) and scorers.fingerprint
    modules used to merge spectra, predict fingerprints, and rank candidates
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: SciPy
  role: Required dependency for loading and processing merged MS/MS spectra as feature
    vectors
- name: NumPy
  role: Required dependency for array-based fingerprint bit vector and SVM feature
    handling
- name: h5py
  role: Required dependency for reading the HDF5 compound databases with pre-calculated
    fingerprints
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:370796a67e782f24043724243e9cf02e3ef431894d6c5c770d1308791ad4e81f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDistiller
  - NumPy
  - SciPy
  - h5py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# msms-spectra-merging

## Summary

Reconstructs ChemDistiller's FingerScorer pipeline: merge tandem MS/MS spectra into training feature vectors, train one libSVM model per occupied CDK fingerprint bit to predict molecular fingerprints from experimental MS/MS data, then rank candidate compounds by Jaccard similarity between predicted and pre-calculated reference fingerprints.

## When to use

Use when tandem MS/MS spectra must be converted into merged training feature vectors for per-bit libSVM fingerprint prediction, and candidate compounds must be ranked by Jaccard similarity of predicted vs. reference fingerprints from chemdb.

## When NOT to use

- Input spectra are not in ChemDistiller's internal hierarchical format — format conversion was not yet available and other formats require prior conversion.
- The task is element/formula/InChI candidate filtering rather than fingerprint-based scoring — use the chemdistiller.filters.element, filters.formula, or filters.inchi modules instead.
- Only 1D MS1 peak detection is needed without tandem MS/MS annotation — the FingerScorer depends on MS/MS spectra as feature vectors.

## Inputs

- Tandem MS/MS spectra in ChemDistiller internal hierarchical format (see int_format.md), e.g. from .../ChemDistiller/data/MassBankTestSpectra/
- Pre-calculated reference molecular fingerprints from the chemdb compound database (HDF5 *.h5 format, copied to .../ChemDistiller/DBs/)
- Pre-trained libSVM FingerScorer models (Linear_weighted or Radial_weighted), organized as '1' and '-1' subfolders in .../ChemDistiller/SVMs/

## Outputs

- Predicted binary fingerprint bit vectors per input spectrum, restricted to the 2500 occupied CDK fingerprint bits
- Ranked candidate compound lists scored by Jaccard similarity between predicted and reference fingerprints
- Annotated_spectra.json, per-spectrum hierarchical text annotations, log.txt, and HTML Report folder (from an annotate.py run)

## How to apply

Load the merged MS/MS training spectra as feature vectors using the chemdistiller.msspectra modules (spectrum, peak, manager). Apply the bit-occupancy filter to restrict the prediction target space to the 2500 occupied bits of the 11416-bit CDK fingerprint union, rather than training on all bits. Train one binary libSVM model per occupied fingerprint bit on the merged-spectra features; choose Linear_weighted SVMs for speed or Radial_weighted SVMs for better prediction power (installed by default, but replaceable via the --svm_folder argument or by copying subfolders '1' and '-1' into .../ChemDistiller/SVMs/). At scoring time, use the chemdistiller.scorers.fingerprint module to compare predicted bit vectors against pre-calculated reference fingerprints from the chemdb database and rank candidates by Jaccard similarity. Spectra must be in ChemDistiller's internal hierarchical format before annotation.

## Related tools

- **ChemDistiller** (Annotation engine hosting the msspectra (spectrum, peak, manager) and scorers.fingerprint modules used to merge spectra, predict fingerprints, and rank candidates) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **SciPy** (Required dependency for loading and processing merged MS/MS spectra as feature vectors)
- **NumPy** (Required dependency for array-based fingerprint bit vector and SVM feature handling)
- **h5py** (Required dependency for reading the HDF5 compound databases with pre-calculated fingerprints)

## Examples

```
python annotate.py /home/me/my_spectra /home/me/my_results --svm_folder /home/me/my_svms
```

## Evaluation signals

- Every predicted bit falls within the 2500-bit occupied subset of the 11416-bit CDK fingerprint union.
- Each per-bit libSVM model emits valid binary (class 1 / class -1) predictions.
- Jaccard-similarity rankings are consistent with the pre-calculated reference fingerprints stored in the chemdb database.
- Merged training spectra load without errors as feature vectors via the chemdistiller.msspectra.spectrum, peak, and manager modules.
- A test-mode run (python annotate.py --test, 863 MassBank spectra with known structures) prints retrieval statistics successfully.

## Limitations

- The shipped default SVM models and compound databases are small due to size limitations; large-scale analysis requires downloading larger (slower) SVMs and databases from the provided MediaFire folders.
- Linear_weighted SVMs are smaller and faster but have worse prediction power than Radial_weighted SVMs, which are larger and slower.
- When replacing default SVMs, both the '1' and '-1' subfolders in .../ChemDistiller/SVMs/ must be deleted first to avoid confusion.
- Commercial use of the derivative compound databases may be restricted by the licenses of the original source databases; consult each database's readme.txt and legal disclaimers.
- No discussion-section content (e.g. detailed caveats on prediction performance) was available in the provided document.

## Evidence

- [other] FingerScorer trains libSVM models on reference spectra to predict molecular fingerprints from tandem MS experimental data; retrieved candidates are then ranked by Jaccard similarity between predicted and actual fingerprints.: [evidence span withheld — anchor status: not_found]
- [methods] msspectra modules: "chemdistiller.msspectra.spectrum module ---------------------------------------"
- [readme] SVM selection tradeoff: [evidence span withheld — anchor status: external_doc]
- [abstract] Database contents: "combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation"
- [readme] SVM replacement procedure: [evidence span withheld — anchor status: external_doc]
