---
name: locality-sensitive-hashing-reference-matching
description: Use when timsTOF/TDF raw DDA-PASEF frames must be screened for peptide
  isotope patterns without a classical peak-picking pipeline, e.g., to match vendor-selected
  precursors from a .d folder via hashed averagine references.
when_to_use_negative:
- 'When the sample is dominated by repeated chemical contaminants: self-similar chemical
  noise would be misclassified as ''true'' signal by the collision-based approach.'
- When the target signals are low relative intensity compared to neighboring noise
  peaks — the similarity measure on which the LSH is based drops and such signals
  can be lost.
- When a validated peak-level gold-standard ground truth or a production-grade feature
  detector is required; the reference search is only prototypic and evaluation on
  real data lacked ground-truth annotation.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: pyproteolizard-algorithm
  role: Implements the LSH used to hash spectrum windows and the averagine synthetic
    reference database for candidate retrieval via the collision table
  repo: https://github.com/theGreatHerrLebert/proteolizard-algorithm
- name: pyproteolizard-data
  role: Object-oriented data access layer exposing timsTOF frames, spectra and slices
    (PyTimsDataHandle) to Python for window extraction and binning
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: OpenTIMS
  role: Low-level C++ API underlying proteolizard-data for TDF raw-data access
  repo: https://github.com/michalsta/opentims
- name: TensorFlow 2
  role: Provides GPU acceleration for the LSH implementation
- name: mzBucket
  role: Original repository containing createSynthetics (reference pattern generation)
    and scalability scripts; superseded by the proteolizard packages
  repo: https://github.com/hildebrandtlab/mzBucket
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:f8efe0192dee5572f50df63cbf7f896c43e87acb9d03c613ee1b5f099ade915d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theGreatHerrLebert/proteolizard-algorithm
  tools:
  - OpenTIMS
  - Python
  - TensorFlow 2
  - mzBucket
  - pyproteolizard-algorithm
  - pyproteolizard-data
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7368421052631579
n_claims: 19
---

# locality-sensitive-hashing-reference-matching

## Summary

Detects peptide isotope patterns in timsTOF raw data by hashing unidentified signal windows against a hashed averagine-modeled synthetic reference database (built with createSynthetics/pyproteolizard-algorithm), then confirming candidates via explicit cosine similarity and matching detections to vendor-selected DDA-PASEF precursors in m/z, ion-mobility apex, and charge.

## When to use

Use when timsTOF/TDF raw DDA-PASEF frames must be screened for peptide isotope patterns without a classical peak-picking pipeline, e.g., to match vendor-selected precursors from a .d folder via hashed averagine references.

## When NOT to use

- When the sample is dominated by repeated chemical contaminants: self-similar chemical noise would be misclassified as 'true' signal by the collision-based approach.
- When the target signals are low relative intensity compared to neighboring noise peaks — the similarity measure on which the LSH is based drops and such signals can be lost.
- When a validated peak-level gold-standard ground truth or a production-grade feature detector is required; the reference search is only prototypic and evaluation on real data lacked ground-truth annotation.

## Inputs

- Bruker timsTOF TDF/TIMS raw data (.d folder) from Zenodo record 5036526, opened via PyTimsDataHandle
- List of N=6902 vendor-selected DDA-PASEF precursor peptides (m/z, ion-mobility apex, charge)
- Averagine-model synthetic isotope pattern reference database (masses 150–1800 Da, charge states 1–5, 1e-2 Da bins)

## Outputs

- Confirmed isotope-pattern detections with charge state and monoisotopic m/z (cosine similarity >= 0.6)
- Match cascade counts k1 (candidate matches), k2 (confirmed matches), k3 (precursor-matched detections), written to match_cascade.csv
- Precursor match verdicts per vendor-selected peptide (Manhattan distance <= 10 over m/z, ion-mobility apex, charge)

## How to apply

Generate synthetic isotope patterns from the averagine model covering 150–1800 Da at 1e-5 sampling binned to 1e-2 Da with charge states 1–5 using the createSynthetics module, hash them to form the reference database. Download the raw subset from Zenodo record 5036526, unzip it into the data folder, and open the TDF/TIMS experiment via pyproteolizard-data (PyTimsDataHandle, built on OpenTIMS). Hash unidentified signal windows with the LSH in pyproteolizard-algorithm (tunable m OR / n AND connectives; (m,n)=(64,32) discriminates well) and retrieve candidate reference patterns via the hash collision table. Explicitly score candidates with cosine similarity and retain those at or above the 0.6 threshold. Match confirmed detections to the N=6902 vendor-selected precursors in 3 dimensions (m/z, ion-mobility apex, charge), accepting Manhattan distance <= 10, and aggregate the match cascade counts k1/k2/k3 (≈6200 of 6902, ≈91%, matched with appropriate m and n) to match_cascade.csv.

## Related tools

- **pyproteolizard-algorithm** (Implements the LSH used to hash spectrum windows and the averagine synthetic reference database for candidate retrieval via the collision table) — https://github.com/theGreatHerrLebert/proteolizard-algorithm
- **pyproteolizard-data** (Object-oriented data access layer exposing timsTOF frames, spectra and slices (PyTimsDataHandle) to Python for window extraction and binning) — https://github.com/theGreatHerrLebert/proteolizard-data
- **OpenTIMS** (Low-level C++ API underlying proteolizard-data for TDF raw-data access) — https://github.com/michalsta/opentims
- **TensorFlow 2** (Provides GPU acceleration for the LSH implementation)
- **mzBucket** (Original repository containing createSynthetics (reference pattern generation) and scalability scripts; superseded by the proteolizard packages) — https://github.com/hildebrandtlab/mzBucket

## Examples

```
from proteolizarddata.data import PyTimsDataHandle; dh = PyTimsDataHandle('path/to/experiment.d'); frame = dh.get_frame(1)
```

## Evaluation signals

- With appropriate m and n settings, ~6200 of N=6902 vendor-selected DDA-PASEF peptides (≈91% of N) should be considered matched by the initial reference-based metric.
- All retained candidate patterns must pass the explicit cosine similarity threshold of 0.6.
- Precursor matches must satisfy Manhattan distance <= 10 jointly over m/z, ion-mobility apex, and charge dimensions.
- The match cascade must be internally consistent: k3 (precursor-matched detections) <= k2 (confirmed matches) <= k1 (candidate matches), recorded in match_cascade.csv.
- Stricter discrimination settings such as (m,n)=(64,32) should measurably improve true-positive vs. false-positive rates relative to (m,n)=(32,16).

## Limitations

- Charge state identification sometimes fails.
- The monoisotopic m/z can be shifted by multiples of m/z in some cases; the reference search is not yet perfect and only prototypic (no released/finished implementation).
- Self-similar chemical noise (repeated contaminants) would be classified as 'true' signal.
- Signals with low relative intensity to noise peaks could be lost because the similarity measure drops.
- Vendor-selected DDA peaks are a biased reference — only fragmented, likely high-intensity peptides — and no gold-standard peak-level annotation of real data was available.

## Evidence

- [results] Synthetic reference hashing: "These synthetic isotope patterns are hashed to form a reference database, which then can be used to detect isotope patterns by collision of hashes in sets of spectrum windows."
- [discussion] Cosine similarity confirmation: "When an unidentified window is now hashed, its key set can be used to select a set of candidate isotope patterns, for which cosine similarity is calculated explicitly."
- [discussion] Signal/noise classification via collision table: "A window in question is called “signal” if at least one of its hash values can be found in the collision table, otherwise it is called “noise”."
- [results] Vendor precursor ground truth: [evidence span withheld — anchor status: not_found]
- [readme] Data access stack: [evidence span withheld — anchor status: not_found]
- [readme] Dataset placement for reproducibility: "Please unzip it into the data folder to ensure reproducible runs of the scripts."
