---
name: compound-level-data-split-stratification
description: Use when a dataset contains multiple spectra (e.g., EI-MS and ESI-MS/MS
  from MassBank) per compound and a classifier such as Random Forest must be evaluated
  without compound-level spectral leakage.
when_to_use_negative:
- Each row in the dataset is an independent compound with exactly one spectrum, so
  no compound-level grouping is needed
- The goal is external validation on spectra from new publications or laboratory measurements
  rather than an internal test estimate
- The task is prediction on individual EI-MS-only or ESI-MS/MS-only sets, which the
  article reports gave poor results compared with the combined set
tools:
- name: BitterMasS
  role: Random Forest classifier retrained on each of the 20 compound-level 80/20
    training partitions and evaluated on the held-out compounds
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:f29fc532afdad59dd87e2f89d1085f086022762621749efa844f97eb510bfac3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BitterMasS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# compound-level-data-split-stratification

## Summary

Partition mass-spectral training data so that all spectra of the same compound stay within a single fold, using an 80/20 compound-level train/test split repeated 20 times. This prevents spectra from one compound leaking into both train and test partitions, which would inflate classifier performance.

## When to use

Use when a dataset contains multiple spectra (e.g., EI-MS and ESI-MS/MS from MassBank) per compound and a classifier such as Random Forest must be evaluated without compound-level spectral leakage.

## When NOT to use

- Each row in the dataset is an independent compound with exactly one spectrum, so no compound-level grouping is needed
- The goal is external validation on spectra from new publications or laboratory measurements rather than an internal test estimate
- The task is prediction on individual EI-MS-only or ESI-MS/MS-only sets, which the article reports gave poor results compared with the combined set

## Inputs

- Experimental mass spectra (EI-MS and ESI-MS/MS positive ion mode, e.g., retrieved from MassBank) labeled bitter/non-bitter, with a compound identifier mapping each spectrum to its parent compound
- Fixed-length spectral descriptors (0.1 m/z bins with peak intensities as bin values, plus instrument-type descriptors)

## Outputs

- 20 repeated 80/20 compound-level train/test partitions with no compound shared between partitions
- Aggregated precision, recall, and F1 over the 20 repetitions (target: precision = 0.83, recall = 0.90) versus the naive baseline (precision/recall = 0.67, F1 = 0.67, BA = 0.51)

## How to apply

Group the experimental EI-MS and ESI-MS/MS positive ion mode spectra by parent compound rather than by spectrum. Assign compounds (not spectra) to an 80% training and 20% internal test partition, ensuring all spectra of one compound remain in the same fold. Repeat this split 20 times, each time retraining the Random Forest (BitterMasS) on the binned 0.1 m/z spectral descriptors of the training compounds. Aggregate precision, recall, and F1 across the 20 repetitions and compare against the naive proportion-based baseline (baseline precision/recall = 0.67, F1 = 0.67, BA = 0.51). Note that combining EI-MS and ESI-MS/MS spectra in one model outperformed models trained on either instrument type alone, so the split should be performed over the combined spectral set.

## Related tools

- **BitterMasS** (Random Forest classifier retrained on each of the 20 compound-level 80/20 training partitions and evaluated on the held-out compounds)

## Evaluation signals

- No compound ID appears in both the training and internal test partitions in any of the 20 repetitions
- Aggregated internal-test precision and recall reproduce ~0.83 and ~0.90 and clearly exceed the naive baseline (precision/recall = 0.67, F1 = 0.67, BA = 0.51)
- Every spectrum of a given compound carries the same instrument-type descriptor as its fold assignment, and combined EI-MS + ESI-MS/MS training is used rather than single-technique subsets
- Performance is reported as an average over all 20 repetitions, not a single split

## Limitations

- Only about 30% of the compounds in the initial bitter and non-bitter sets had at least one mass spectrum, limiting the number of compounds available for the split and overall training data
- Nonbitter compound spectral representation in the training set is insufficient, so compound-level splits may leave the test fold under-representing nonbitter compounds
- Splitting at the spectrum level instead of the compound level would leak information, since multiple spectra of the same compound share its chemical identity

## Evidence

- [other] 80/20 compound-level split repeated 20 times: [evidence span withheld — anchor status: not_found]
- [discussion] Combined instrument sets outperform individual sets: "Prediction on the individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set."
- [discussion] Limited spectral coverage of training compounds: "only 30% of the compounds in the initial bitter and nonbitter sets in the training set had at least one mass spectrum identified"
