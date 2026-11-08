---
name: cdk-fingerprint-bit-occupancy-filtering
description: Use when training or applying FingerScorer-style SVM fingerprint predictors
  over a 11416-bit CDK fingerprint space, and you must prune to the 2500 occupied
  bits before per-bit model training or Jaccard ranking.
when_to_use_negative:
- Your reference fingerprint scheme is not the CDK 11416-bit union (e.g. a different
  fingerprint type), so the 2500-bit occupancy mask would be invalid
- Input spectra are not in ChemDistiller's internal hierarchical format and no conversion
  is available yet
- You need to predict fingerprints for bits never observed in the reference database
  — training models on unoccupied bits is exactly what this filter prevents
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: ChemDistiller
  role: Provides the msspectra modules (spectrum, peak, manager) for building feature
    vectors, the chemdb database with pre-calculated fingerprints, and the scorers.fingerprint
    module for Jaccard ranking after bit filtering
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: libSVM
  role: Trains one binary classification model per occupied fingerprint bit after
    the occupancy filter restricts the target space
- name: SciPy
  role: Numerical dependency required by ChemDistiller for handling spectral feature
    vectors
- name: NumPy
  role: Numerical dependency required by ChemDistiller for bit-vector and spectrum
    array operations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:f9553ddabac3714fd7a414c7b950dab14ed50c1a24d52348600a3a908b2384aa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDistiller
  - NumPy
  - SciPy
  - libSVM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cdk-fingerprint-bit-occupancy-filtering

## Summary

Restricts FingerScorer's fingerprint prediction target space from the full 11416-bit CDK fingerprint union down to the 2500 occupied bits, so one binary libSVM model is trained per occupied bit only. This cuts model count and training cost while keeping every bit that actually occurs in the reference compound database.

## When to use

Use when training or applying FingerScorer-style SVM fingerprint predictors over a 11416-bit CDK fingerprint space, and you must prune to the 2500 occupied bits before per-bit model training or Jaccard ranking.

## When NOT to use

- Your reference fingerprint scheme is not the CDK 11416-bit union (e.g. a different fingerprint type), so the 2500-bit occupancy mask would be invalid
- Input spectra are not in ChemDistiller's internal hierarchical format and no conversion is available yet
- You need to predict fingerprints for bits never observed in the reference database — training models on unoccupied bits is exactly what this filter prevents

## Inputs

- Merged MS/MS training spectra converted to ChemDistiller internal hierarchical format feature vectors
- Pre-calculated CDK fingerprints for tens of millions of compounds from the chemdb database (HDF5 *.h5 files)
- CDK fingerprint definition covering the 11416-bit union space

## Outputs

- Bit-occupancy mask identifying the 2500 occupied bits of the 11416-bit CDK fingerprint union
- One binary libSVM model per occupied fingerprint bit (organized in '1' and '-1' subfolders under .../ChemDistiller/SVMs/)
- Predicted binary fingerprint bit vectors restricted to the 2500-bit occupied set, used for Jaccard-similarity candidate ranking

## How to apply

Load merged MS/MS training spectra as feature vectors using the chemdistiller.msspectra modules (spectrum, peak, manager), then compute the union of CDK fingerprints across the reference compounds in the chemdb database (supplied as HDF5 *.h5 files) to identify which of the 11416 bits are ever set. Apply the bit-occupancy filter: keep only the 2500 occupied bit indices and discard the remaining unused bits, so that one binary libSVM model is trained per occupied bit rather than per bit of the full space. Predicted bit vectors must lie entirely within this 2500-bit occupied set; downstream, the chemdistiller.scorers.fingerprint module compares predicted against pre-calculated reference fingerprints via Jaccard similarity to rank candidates. If swapping in larger SVM sets (Linear_weighted vs Radial_weighted), clear both the '1' and '-1' subfolders in .../ChemDistiller/SVMs/ before copying new models to avoid mixing predictors.

## Related tools

- **ChemDistiller** (Provides the msspectra modules (spectrum, peak, manager) for building feature vectors, the chemdb database with pre-calculated fingerprints, and the scorers.fingerprint module for Jaccard ranking after bit filtering) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **libSVM** (Trains one binary classification model per occupied fingerprint bit after the occupancy filter restricts the target space)
- **SciPy** (Numerical dependency required by ChemDistiller for handling spectral feature vectors)
- **NumPy** (Numerical dependency required by ChemDistiller for bit-vector and spectrum array operations)

## Examples

```
python annotate.py /home/me/my_spectra /home/me/my_results --svm_folder /home/me/my_svms
```

## Evaluation signals

- Every predicted fingerprint bit falls within the 2500-bit occupied set of the 11416-bit CDK union
- Each per-bit libSVM model emits valid binary predictions (no intermediate or missing values)
- Jaccard-similarity rankings are consistent with the pre-calculated reference fingerprints stored in the chemdb database
- If SVMs were replaced, both '1' and '-1' subfolders exist under .../ChemDistiller/SVMs/ and no stale models from the previous set remain

## Limitations

- The shipped SVM models are simplified 'due to size limitations'; larger, more accurate Radial_weighted SVMs must be downloaded separately and are slower and larger
- Linear_weighted SVMs are smaller and faster but have worse prediction power than Radial_weighted
- Generated databases and SVMs are provided AS IS, with possible commercial-use restrictions inherited from the original compound databases — check readme.txt and source database licenses
- Input spectra must be converted to ChemDistiller's internal format; conversions from other formats were not yet implemented at v0.1

## Evidence

- [other] bit-occupancy filter restricts target space to 2500 occupied bits: [evidence span withheld — anchor status: not_found]
- [readme] SVM model organization: [evidence span withheld — anchor status: external_doc]
- [readme] Linear vs Radial SVM trade-off: [evidence span withheld — anchor status: external_doc]
- [readme] spectra must be in internal format: [evidence span withheld — anchor status: external_doc]
- [methods] chemdb database module: "chemdistiller.chemdb.database module
------------------------------------

.. automodule:: chemdistiller.chemdb.database"
