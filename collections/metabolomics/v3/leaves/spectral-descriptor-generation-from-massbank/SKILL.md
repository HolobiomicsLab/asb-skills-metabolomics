---
name: spectral-descriptor-generation-from-massbank
description: Use when raw experimental mass spectra (EI-MS or ESI-MS/MS, positive
  ion mode) must be turned into fixed-length numeric feature vectors — e.g., to train
  or apply a classifier like BitterMasS on spectral input rather than chemical structures.
when_to_use_negative:
- The chemical structure is known and you want bitterness from structure — use BitterPredict
  instead, since the direct spectrum route is specifically recommended when the structure
  is unknown
- Input spectra are in negative ion mode — the current BitterMasS implementation makes
  predictions based on EI-MS or ESI-MS/MS spectra obtained in positive ion mode only
- Input is already a fixed-length descriptor/feature table — regenerating binned descriptors
  would be redundant
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: BitterMasS
  role: Random Forest classifier trained on the generated spectral descriptors (5414
    experimental mass spectra) to predict bitterness from the spectrum
- name: MassBank
  role: Source database from which the experimental EI-MS and ESI-MS/MS positive ion
    mode spectra are extracted for descriptor generation
- name: BitterPredict1
  role: Related Niv-Lab code release for the structure-based BitterPredict classifier;
    provided as adjacent repository context, not for spectral descriptor generation
  repo: https://github.com/Niv-Lab/BitterPredict1
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:95ba8d82e608d7cfc86d1927c4a51f225e861a4bcd0002d9041af21fe9d2b064
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Niv-Lab/BitterPredict1
  tools:
  - BitterMasS
  - BitterPredict1
  - MassBank
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# spectral-descriptor-generation-from-massbank

## Summary

Converts experimental EI-MS and ESI-MS/MS positive ion mode mass spectra retrieved from MassBank into fixed-length spectral descriptors by binning peaks into 0.1 m/z bins (peak intensities as bin values) and appending instrument-type descriptors. These descriptors are the feature representation that BitterMasS, a Random Forest classifier, trains on to predict bitterness directly from the spectrum.

## When to use

Use when raw experimental mass spectra (EI-MS or ESI-MS/MS, positive ion mode) must be turned into fixed-length numeric feature vectors — e.g., to train or apply a classifier like BitterMasS on spectral input rather than chemical structures.

## When NOT to use

- The chemical structure is known and you want bitterness from structure — use BitterPredict instead, since the direct spectrum route is specifically recommended when the structure is unknown
- Input spectra are in negative ion mode — the current BitterMasS implementation makes predictions based on EI-MS or ESI-MS/MS spectra obtained in positive ion mode only
- Input is already a fixed-length descriptor/feature table — regenerating binned descriptors would be redundant

## Inputs

- Experimental EI-MS mass spectra from MassBank (positive ion mode)
- Experimental ESI-MS/MS mass spectra from MassBank (positive ion mode)
- Compound-level bitter/non-bitter labels (e.g., from BitterDB-derived sets)
- Instrument-type annotation for each spectrum

## Outputs

- Fixed-length spectral descriptor vectors per spectrum (0.1 m/z bin intensities plus instrument-type descriptors)
- Compound-level 80/20 train/test splits (20 repetitions, compound-level grouping)

## How to apply

Retrieve experimental EI-MS and ESI-MS/MS positive ion mode spectra for each compound from the MassBank database. Convert each spectrum into a fixed-length descriptor by binning all peaks into 0.1 m/z bins, using the peak intensities as the bin values, and append instrument-type descriptors to the vector. Keep all spectra of a given compound together when partitioning data: split 80% training / 20% internal test at the compound level (repeating the split 20 times) so spectra never leak across folds. Train the downstream Random Forest (BitterMasS) on the training partition per repetition and aggregate precision/recall/F1 over the 20 repetitions, comparing against the naive proportion-based baseline (precision/recall = 0.67, F1 = 0.67, BA = 0.51). Use the combined EI-MS + ESI-MS/MS set, since prediction on the individual instrument sets showed poor results compared with the combined set.

## Related tools

- **BitterMasS** (Random Forest classifier trained on the generated spectral descriptors (5414 experimental mass spectra) to predict bitterness from the spectrum)
- **MassBank** (Source database from which the experimental EI-MS and ESI-MS/MS positive ion mode spectra are extracted for descriptor generation)
- **BitterPredict1** (Related Niv-Lab code release for the structure-based BitterPredict classifier; provided as adjacent repository context, not for spectral descriptor generation) — https://github.com/Niv-Lab/BitterPredict1

## Evaluation signals

- Descriptor vectors are fixed-length and comparable across spectra: every spectrum is binned on the same 0.1 m/z grid with instrument-type descriptors appended
- Compound-level leakage check: all spectra of one compound stay within the same fold across the 20 repeated 80/20 splits
- Downstream BitterMasS internal-test performance reaches precision = 0.83 and recall = 0.90, clearly above the naive baseline (precision/recall = 0.67, F1 = 0.67, BA = 0.51)
- Aggregated metrics (precision, recall, F1) are reported over all 20 training/test repetitions rather than a single split
- Combined EI-MS + ESI-MS/MS descriptors are used, since prediction on the individual instrument sets showed poor results

## Limitations

- Only about 30% of the compounds in the initial bitter and nonbitter training sets had at least one mass spectrum identified, limiting the training data
- Nonbitter compound spectral representation in the training set is insufficient; more would likely improve performance
- Only positive ion mode ESI-MS/MS and EI-MS are supported by the current BitterMasS implementation
- Only a small fraction (5–10%) of mass spectra can be assigned to known structures, and most articles reporting experimental mass spectra do not include the actual spectra; standardized mass spectral deposition is lacking

## Evidence

- [abstract] We present BitterMasS, a Random Forest classifier that was trained on 5414 experimental mass spectra: "We present BitterMasS, a Random Forest classifier that was trained on 5414 experimental mass spectra"
- [discussion] Experimental EI-MS and ESI-MS/MS positive ion mode mass spectra for these compounds were extracted from the MassBank database: "Experimental EI-MS and ESI-MS/MS positive ion mode mass spectra for these compounds were extracted from the MassBank database"
- [other] Convert each spectrum into a fixed-length spectral descriptor by binning peaks into 0.1 m/z bins with peak intensities as bin values, and append instrument-type descriptors.: [evidence span withheld — anchor status: not_found]
- [other] Partition the compounds into 80% training and 20% internal test sets at the compound level, repeating the split 20 times so all spectra of one compound stay in the same fold.: [evidence span withheld — anchor status: not_found]
