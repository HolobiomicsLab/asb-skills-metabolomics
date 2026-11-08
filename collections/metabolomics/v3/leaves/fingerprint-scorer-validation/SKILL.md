---
name: fingerprint-scorer-validation
description: Use when tandem MS/MS spectra must be converted into predicted CDK molecular
  fingerprints via per-bit libSVM models, and candidate compounds ranked by Jaccard
  similarity against pre-calculated database fingerprints.
when_to_use_negative:
- You need to predict across all 11416 CDK fingerprint bits — libSVM models only emit
  predictions within the 2500 occupied-bit subset defined by the bit-occupancy filter
- Input spectra are not in ChemDistiller's internal hierarchical format and cannot
  be converted — format converters were not yet implemented at time of writing
- The analysis is formula/element filtering rather than fingerprint-based candidate
  ranking — chemdistiller.filters.element, .formula, and .inchi modules are the appropriate
  entry points instead
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2258
tools:
- name: ChemDistiller
  role: Hosts the scorers.fingerprint module that compares predicted bit vectors to
    pre-calculated reference fingerprints and ranks candidates by Jaccard similarity
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: libSVM
  role: One binary classification model per occupied fingerprint bit, trained on merged-spectra
    features to emit {1,-1} bit predictions
- name: h5py
  role: Reads the HDF5 compound databases (*.h5) holding pre-calculated fingerprints
    used for Jaccard ranking
  repo: http://www.h5py.org/
- name: SciPy
  role: Numerical dependency required by ChemDistiller for the spectral feature-vector
    computation
- name: NumPy
  role: Numerical dependency for array-based bit-vector and Jaccard-similarity computation
- name: RDKit
  role: Optional; used only to render 2D chemical structure images in the HTML annotation
    report
  repo: http://www.rdkit.org/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:eea4d4034f0491e2ebbccee6b62baeda387af228c76acb75272bef076f92b808
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDistiller
  - NumPy
  - RDKit
  - SciPy
  - h5py
  - libSVM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# fingerprint-scorer-validation

## Summary

FingerScorer-style validation: per-bit binary libSVM models predict CDK molecular fingerprints from tandem MS spectra, and candidate compounds are ranked by Jaccard similarity against pre-calculated fingerprints in the ChemDistiller compound database.

## When to use

Use when tandem MS/MS spectra must be converted into predicted CDK molecular fingerprints via per-bit libSVM models, and candidate compounds ranked by Jaccard similarity against pre-calculated database fingerprints.

## When NOT to use

- You need to predict across all 11416 CDK fingerprint bits — libSVM models only emit predictions within the 2500 occupied-bit subset defined by the bit-occupancy filter
- Input spectra are not in ChemDistiller's internal hierarchical format and cannot be converted — format converters were not yet implemented at time of writing
- The analysis is formula/element filtering rather than fingerprint-based candidate ranking — chemdistiller.filters.element, .formula, and .inchi modules are the appropriate entry points instead

## Inputs

- Tandem MS spectra in ChemDistiller internal hierarchical format (see int_format.md)
- Merged MS/MS training spectra as feature vectors loaded via chemdistiller.msspectra.spectrum / peak / manager modules
- Compound databases in HDF5 format (*.h5) containing pre-calculated fingerprints, placed in .../ChemDistiller/DBs/
- libSVM model folders with '1' and '-1' subfolders (Linear_weighted or Radial_weighted) under .../ChemDistiller/SVMs/ or a --svm_folder path

## Outputs

- Predicted binary fingerprint bit vectors restricted to the 2500-bit occupied subset of the 11416-bit CDK fingerprint union
- Jaccard-similarity-ranked candidate compound lists per MS1 peak
- annotated_spectra.json, log.txt, individual annotated spectra files, and an HTML Report with index.html

## How to apply

Load the merged MS/MS spectra through the chemdistiller.msspectra modules (spectrum, peak, manager) as feature vectors, then apply the bit-occupancy filter that restricts the prediction target space to the 2500 occupied bits of the 11416-bit CDK fingerprint union. Train one binary libSVM model per occupied fingerprint bit; choose between Linear_weighted SVMs (smaller, faster, weaker prediction) and Radial_weighted SVMs (larger, slower, strongly recommended). At annotation time, use the chemdistiller.scorers.fingerprint module to compare the predicted bit vectors against pre-calculated reference fingerprints stored in the HDF5 compound databases (*.h5 in .../ChemDistiller/DBs/) from chemdb, ranking candidates by Jaccard similarity. SVM models live in '1' and '-1' subfolders under .../ChemDistiller/SVMs/ or a folder supplied via --svm_folder; if replaced, delete both old subfolders first to avoid confusion.

## Related tools

- **ChemDistiller** (Hosts the scorers.fingerprint module that compares predicted bit vectors to pre-calculated reference fingerprints and ranks candidates by Jaccard similarity) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **libSVM** (One binary classification model per occupied fingerprint bit, trained on merged-spectra features to emit {1,-1} bit predictions)
- **h5py** (Reads the HDF5 compound databases (*.h5) holding pre-calculated fingerprints used for Jaccard ranking) — http://www.h5py.org/
- **SciPy** (Numerical dependency required by ChemDistiller for the spectral feature-vector computation)
- **NumPy** (Numerical dependency for array-based bit-vector and Jaccard-similarity computation)
- **RDKit** (Optional; used only to render 2D chemical structure images in the HTML annotation report) — http://www.rdkit.org/

## Examples

```
python annotate.py /home/me/my_spectra /home/me/my_results --svm_folder /home/me/my_svms
```

## Evaluation signals

- Every predicted bit index falls within the 2500-bit occupied subset of the 11416-bit CDK fingerprint union (bit-occupancy filter invariant)
- Each per-bit libSVM model emits valid binary predictions ('1' or '-1'), never fractional or missing values
- Jaccard rankings are consistent with the pre-calculated reference fingerprints stored in the chemdb database
- Running 'python annotate.py --test' on the 863 MassBank test spectra prints retrieval statistics, confirming the SVM + DB pairing is intact
- After swapping SVM models, both '1' and '-1' subfolders exist under the active SVM folder and stale copies were deleted

## Limitations

- The default distribution ships only a small set of compound databases and simple linear-weighted SVM models due to size limitations; larger HDF5 DBs and radial SVMs must be downloaded separately
- Radial_weighted SVMs give better prediction power but are larger and slower than Linear_weighted SVMs
- Spectra must be in ChemDistiller's internal hierarchical format; conversion from other formats was still pending at the time of the README
- Derivative compound databases may inherit commercial-use restrictions from their original sources; consult the readme.txt bundled with the DB downloads and cite original databases
- RDKit support is optional and only affects 2D structure images in reports, not fingerprint prediction

## Evidence

- [abstract] combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation: "combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation"
- [intro] __ChemDistiller__ is the high-throughput annotation engine for tandem MS spectra annotation.: "__ChemDistiller__ is the high-throughput annotation engine for tandem MS spectra annotation."
- [other] Apply the bit-occupancy filter, restricting the target space to the 2500 occupied bits of the 11416-bit CDK fingerprint union.: [evidence span withheld — anchor status: not_found]
- [other] Use the chemdistiller.scorers.fingerprint module to compare predicted bit vectors against pre-calculated reference fingerprints from the chemdb database and rank candidates by Jaccard similarity.: [evidence span withheld — anchor status: not_found]
- [readme] _Linear_ are smaller, faster, but prediction power is worse than for _Radial_. _Radial_ are better (and thus strongly recommended), but larger and slower.: [evidence span withheld — anchor status: external_doc]
- [readme] If replacing the default __SVMs__ - make sure to delete both __"1"__ and __"-1"__ subfolders in .../ChemDistiller/SVMs/. to avoid confusion prior to copying the new ones.: [evidence span withheld — anchor status: external_doc]
