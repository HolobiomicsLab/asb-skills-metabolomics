---
name: cosine-similarity-candidate-scoring
description: Use when LSH hash collisions have produced a set of candidate isotope
  patterns for a window in timsTOF/TDF raw data and a definitive signal/noise or identity
  decision is needed before precursor matching.
when_to_use_negative:
- When input is already a processed feature table with charge and monoisotopic m/z
  assigned — candidate scoring via LSH collisions adds no value
- When the target signals are low-intensity relative to noise peaks, since the similarity
  measure the LSH is based on drops and such signals can be lost
- When self-similar chemical noise (e.g. repeated contaminants) dominates, as it would
  be scored as 'true' signal by this approach
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pyproteolizard-algorithm
  role: Provides the LSH implementation whose collision table yields candidate reference
    isotope patterns to be cosine-scored
  repo: https://github.com/theGreatHerrLebert/proteolizard-algorithm
- name: pyproteolizard-data
  role: Provides frame/spectrum access and binning of timsTOF raw data that feed the
    scoring step
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: mzBucket (createSynthetics)
  role: Generates the synthetic averagine isotope patterns that are hashed to form
    the reference database being scored against
  repo: https://github.com/hildebrandtlab/mzBucket
- name: TensorFlow 2
  role: GPU acceleration backend for the LSH/hashing computations underlying candidate
    generation
- name: OpenTIMS
  role: Low-level C++ API on which raw-data access for window extraction is built
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:24de342b4f5a8ac37570303dbc127318d61a50020690b0be97b44b2812a7d4dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theGreatHerrLebert/proteolizard-algorithm
  tools:
  - OpenTIMS
  - Python
  - TensorFlow 2
  - mzBucket (createSynthetics)
  - pyproteolizard-algorithm
  - pyproteolizard-data
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# cosine-similarity-candidate-scoring

## Summary

After LSH collision-based candidate selection, explicitly score unidentified spectrum windows against candidate averagine-modeled reference isotope patterns using cosine similarity, retaining candidates at or above a 0.6 threshold for downstream precursor matching.

## When to use

Use when LSH hash collisions have produced a set of candidate isotope patterns for a window in timsTOF/TDF raw data and a definitive signal/noise or identity decision is needed before precursor matching.

## When NOT to use

- When input is already a processed feature table with charge and monoisotopic m/z assigned — candidate scoring via LSH collisions adds no value
- When the target signals are low-intensity relative to noise peaks, since the similarity measure the LSH is based on drops and such signals can be lost
- When self-similar chemical noise (e.g. repeated contaminants) dominates, as it would be scored as 'true' signal by this approach

## Inputs

- Bruker timsTOF TDF/TIMS raw data (loaded via pyproteolizard-data / OpenTIMS low-level C++ API)
- Synthetic averagine reference isotope-pattern database (masses 150–1800 Da, charges 1–5, binned at 1e-2 Da)
- Hash-collision candidate sets from unidentified spectrum windows
- Vendor-selected DDA-PASEF precursor list (N=6902) for post-scoring matching

## Outputs

- Confirmed isotope-pattern detections (cosine similarity >= 0.6)
- Charge-state and monoisotopic m/z assignments per detection
- Match cascade counts (k1, k2, k3) written to match_cascade.csv

## How to apply

Hash synthetic averagine isotope patterns (masses 150–1800 Da, charge states 1–5, sampled at 1e-5 and binned to 1e-2 Da via the createSynthetics module) into a reference database, then hash unidentified windows with pyproteolizard-algorithm's LSH. The window's key set selects candidate reference patterns from the collision table — this step is deliberately approximate, so an exact scoring pass is required. Compute cosine similarity explicitly between each candidate pattern and the window's binned signal, and retain candidates scoring at or above 0.6. Confirmed detections are then matched against the N=6902 vendor-selected DDA-PASEF precursors in 3 dimensions (m/z, ion-mobility apex, charge), accepting matches with Manhattan distance <= 10, and cascade counts (k1 candidates, k2 confirmed, k3 precursor-matched) are aggregated. LSH parameters (m,n) trade recall against precision — e.g. (32,16) is permissive while (32,32) acts as a strong filter — so tune them before scoring.

## Related tools

- **pyproteolizard-algorithm** (Provides the LSH implementation whose collision table yields candidate reference isotope patterns to be cosine-scored) — https://github.com/theGreatHerrLebert/proteolizard-algorithm
- **pyproteolizard-data** (Provides frame/spectrum access and binning of timsTOF raw data that feed the scoring step) — https://github.com/theGreatHerrLebert/proteolizard-data
- **mzBucket (createSynthetics)** (Generates the synthetic averagine isotope patterns that are hashed to form the reference database being scored against) — https://github.com/hildebrandtlab/mzBucket
- **TensorFlow 2** (GPU acceleration backend for the LSH/hashing computations underlying candidate generation)
- **OpenTIMS** (Low-level C++ API on which raw-data access for window extraction is built)

## Examples

```
from proteolizarddata.data import PyTimsDataHandle; dh = PyTimsDataHandle('path/to/experiment.d'); frame = dh.get_frame(1)
```

## Evaluation signals

- Retained candidates have cosine similarity at or above the 0.6 threshold
- Confirmed detections match vendor-selected precursors with 3D Manhattan distance <= 10 (m/z, ion-mobility apex, charge)
- Match cascade counts k1 >= k2 >= k3 are internally consistent and written to match_cascade.csv
- On the reference setup, roughly 6200 of N=6902 vendor-selected DDA-PASEF peptides (~91%) should be matched, providing a sanity benchmark
- Choice of (m,n) parameters should show the documented trade-off, e.g. (32,16) yields high true-positive and false-positive rates while (32,32) lowers both

## Limitations

- Charge state identification sometimes fails during reference-based matching
- Monoisotopic m/z can be shifted by multiples of the m/z ratio in some cases; the reference search is only prototypic and not yet a finished release
- Chemical noise is self-similar and would be classified as 'true' signal by this approach
- Vendor-selected DDA peaks are a biased reference (only fragmented, likely high-intensity peptides), and no gold-standard peak-level annotation of real data was available
- Signals with low relative intensity to noise peaks can be lost as the underlying similarity measure drops

## Evidence

- [discussion] Candidate selection precedes explicit scoring: "When an unidentified window is now hashed, its key set can be used to select a set of candidate isotope patterns, for which cosine similarity is calculated explicitly."
- [results] Reference database construction from synthetic patterns: "These synthetic isotope patterns are hashed to form a reference database, which then can be used to detect isotope patterns by collision of hashes in sets of spectrum windows."
- [results] LSH parameter tuning affects downstream scoring quality: [evidence span withheld — anchor status: not_found]
- [discussion] Known failure modes of the reference search: "Furthermore, charge state identification sometimes fails as well."
- [readme] Modular tooling for data access: "the components [pyproteolizard-data](https://github.com/theGreatHerrLebert/proteolizard-data) for data-access and [pyproteolizard-algorithm](https://github.com/theGreatHerrLebert/proteolizard-algorith"
