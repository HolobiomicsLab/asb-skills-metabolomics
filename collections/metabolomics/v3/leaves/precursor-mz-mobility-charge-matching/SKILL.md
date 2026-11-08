---
name: precursor-mz-mobility-charge-matching
description: Use when detected isotope patterns from DDA-PASEF TDF/TIMS data must
  be validated against vendor-selected precursor lists, e.g. to measure how many of
  the N=6902 Bruker-selected peptides an LSH pipeline recovers.
when_to_use_negative:
- Input data is not ion-mobility timsTOF (TDF) raw data with vendor precursor annotations,
  since the 3D match dimensions (m/z, ion-mobility apex, charge) are undefined for
  plain LC-MS data.
- You need peak-level ground-truth evaluation of real data — no gold-standard peak-level
  annotation was available; vendor-selected DDA peaks are a biased reference covering
  only fragmented, likely high-intensity peptides.
- Peaks are already identified/annotated; the matching step is only meaningful for
  detections emerging from the LSH signal-classification cascade.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: pyproteolizard-data
  role: Data-access layer over the OpenTIMS low-level C++ API used to load the TDF/TIMS
    subset and expose frames, scans, spectra and slices for precursor matching
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: pyproteolizard-algorithm
  role: Provides the LSH hashing and reference-based isotope-pattern detection whose
    confirmed detections are matched to vendor precursors
  repo: https://github.com/theGreatHerrLebert/proteolizard-algorithm
- name: OpenTIMS
  role: Low-level C++ API underlying data access to the bruker TDF raw files during
    the matching workflow
  repo: https://github.com/michalsta/opentims
- name: Python
  role: Implementation language for the matching script that aggregates k1/k2/k3 cascade
    counts and writes match_cascade.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:e454e25a6f51c580cde9703e317b95ae62df04e89ea83bf53cfabcabfb440ce5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theGreatHerrLebert/proteolizard-data
  tools:
  - OpenTIMS
  - Python
  - TensorFlow 2
  - pyproteolizard-algorithm
  - pyproteolizard-data
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# precursor-mz-mobility-charge-matching

## Summary

Match LSH-based isotope-pattern detections from timsTOF DDA-PASEF raw data against vendor-selected precursors in 3 dimensions (m/z, ion-mobility apex, charge) using a Manhattan distance criterion, to quantify how many of the N=6902 vendor-selected precursor peptides are recovered by the reference-based approach.

## When to use

Use when detected isotope patterns from DDA-PASEF TDF/TIMS data must be validated against vendor-selected precursor lists, e.g. to measure how many of the N=6902 Bruker-selected peptides an LSH pipeline recovers.

## When NOT to use

- Input data is not ion-mobility timsTOF (TDF) raw data with vendor precursor annotations, since the 3D match dimensions (m/z, ion-mobility apex, charge) are undefined for plain LC-MS data.
- You need peak-level ground-truth evaluation of real data — no gold-standard peak-level annotation was available; vendor-selected DDA peaks are a biased reference covering only fragmented, likely high-intensity peptides.
- Peaks are already identified/annotated; the matching step is only meaningful for detections emerging from the LSH signal-classification cascade.

## Inputs

- Bruker timsTOF TDF/TIMS raw data subset (.d folder) from Zenodo record 5036526, loaded via PyTimsDataHandle
- Vendor-selected DDA-PASEF precursor list (N=6902 peptides with m/z, ion-mobility apex, charge)
- LSH-confirmed isotope-pattern detections from pyproteolizard-algorithm (cosine similarity >= 0.6)
- Averagine synthetic reference database (150–1800 Da, 1e-5 sampling, 1e-2 Da bins, charges 1–5)

## Outputs

- Per-detection precursor match assignments in 3D (m/z, ion-mobility apex, charge) with Manhattan distance <= 10
- Match cascade counts (k1 candidate matches, k2 confirmed matches, k3 precursor-matched detections) written to match_cascade.csv
- Matched fraction statistic (e.g. 6200/6902 ≈ 91% of N)

## How to apply

First run the LSH reference search: hash unidentified signal windows with pyproteolizard-algorithm against the averagine-based synthetic reference database (masses 150–1800 Da, charges 1–5, mass axis sampled at 1e-5 and binned to 1e-2 Da), then score candidates explicitly with cosine similarity and retain those at or above the 0.6 threshold. Load the raw subset (TDF/TIMS from Zenodo record 5036526) via pyproteolizard-data on top of the OpenTIMS C++ API to obtain detected peaks with their frame, scan, m/z and mobility coordinates. Match the confirmed detections to the N=6902 vendor-selected DDA-PASEF precursors in the 3 dimensions m/z, ion-mobility apex and charge, accepting matches with Manhattan distance <= 10. Aggregate the match cascade counts (k1 candidate matches, k2 confirmed matches, k3 precursor-matched detections) and write them to match_cascade.csv. The rationale is that vendor-selected precursors are high-confidence annotated masses, so the matched fraction (6200 of 6902, ≈91% of N, with appropriate m and n settings) serves as an accuracy measure for initial matching, charge state assignment, and monoisotopic m/z.

## Related tools

- **pyproteolizard-data** (Data-access layer over the OpenTIMS low-level C++ API used to load the TDF/TIMS subset and expose frames, scans, spectra and slices for precursor matching) — https://github.com/theGreatHerrLebert/proteolizard-data
- **pyproteolizard-algorithm** (Provides the LSH hashing and reference-based isotope-pattern detection whose confirmed detections are matched to vendor precursors) — https://github.com/theGreatHerrLebert/proteolizard-algorithm
- **OpenTIMS** (Low-level C++ API underlying data access to the bruker TDF raw files during the matching workflow) — https://github.com/michalsta/opentims
- **Python** (Implementation language for the matching script that aggregates k1/k2/k3 cascade counts and writes match_cascade.csv)

## Examples

```
from proteolizarddata.data import PyTimsDataHandle; data_handle = PyTimsDataHandle('path/to/experiment.d')
```

## Evaluation signals

- Matched-precursor count falls near the reported 6200 of N=6902 vendor-selected DDA-PASEF peptides (≈91%) when m and n settings are appropriate.
- Every accepted match satisfies Manhattan distance <= 10 across m/z, ion-mobility apex and charge simultaneously.
- Only detections confirmed with cosine similarity >= 0.6 enter the precursor-matching step (k2 stage of the cascade).
- match_cascade.csv contains the full k1/k2/k3 aggregation with monotonically decreasing counts (k3 <= k2 <= k1).
- Vendor precursor count in the evaluation equals N=6902, matching the DDA-PASEF selections reported for the subset.

## Limitations

- Charge state identification sometimes fails, which can invalidate the 3D match even when m/z and mobility agree.
- The monoisotopic m/z can be shifted by multiples of the isotope peak spacing (m_n/z) for some cases, so the reference search is not yet perfect and only prototypic.
- Vendor-selected precursors are a biased reference: they correspond only to peptides selected for fragmentation and likely stem from high-intensity signals, so the ≈91% match rate overestimates performance on low-intensity species.
- Noise peaks within signal windows (noise_2) are not removed by the approach and require subsequent processing steps before matching.

## Evidence

- [methods] Precursor matching in 3D with Manhattan distance cutoff: [evidence span withheld — anchor status: not_found]
- [methods] Cosine similarity confirmation threshold before matching: [evidence span withheld — anchor status: not_found]
- [results] Matched fraction of vendor-selected precursors: [evidence span withheld — anchor status: not_found]
- [results] Explicit cosine scoring after candidate selection: "When an unidentified window is now hashed, its key set can be used to select a set of candidate isotope patterns, for which cosine similarity is calculated explicitly."
- [discussion] Charge state failure caveat: "Furthermore, charge state identification sometimes fails as well."
- [readme] Data loading via PyTimsDataHandle: [evidence span withheld — anchor status: external_doc]
