---
name: mass-spectra-binning-feature-construction
description: Use when raw experimental EI-MS or ESI-MS/MS (positive ion mode) spectra
  must be converted into fixed-length numeric feature vectors for training or applying
  a Random Forest or similar classifier.
when_to_use_negative:
- The chemical structure is already known and you only need a taste prediction — use
  a structure-based predictor such as BitterPredict instead.
- Spectra are in negative ion mode or are predicted/simulated spectra rather than
  experimental positive-mode EI-MS or ESI-MS/MS, which is outside the current BitterMasS
  implementation.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: BitterMasS
  role: Downstream Random Forest classifier trained on the binned spectral descriptors
    to predict bitterness from spectra
  repo: https://github.com/Niv-Lab/BitterPredict1
- name: MassBank
  role: Source database from which experimental EI-MS and ESI-MS/MS positive ion mode
    spectra are retrieved for binning
- name: BitterPredict
  role: Alternative structure-based predictor to compare against the direct spectrum-based
    route; used in the spectrum-to-structure-to-bitterness pipeline
  repo: https://github.com/Niv-Lab/BitterPredict1
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:ddaaae42a3ad913c2a33d7a6da42d4ad97fe2f8efc2c5d63b1ea207bf4283d1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Niv-Lab/BitterPredict1
  tools:
  - BitterMasS
  - BitterPredict
  - MassBank
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# mass-spectra-binning-feature-construction

## Summary

Constructs fixed-length spectral descriptor vectors by binning EI-MS and ESI-MS/MS positive ion mode peak intensities into 0.1 m/z bins, then appends instrument-type descriptors, enabling mass spectra to be used as tabular features for machine-learning classifiers such as BitterMasS.

## When to use

Use when raw experimental EI-MS or ESI-MS/MS (positive ion mode) spectra must be converted into fixed-length numeric feature vectors for training or applying a Random Forest or similar classifier.

## When NOT to use

- The chemical structure is already known and you only need a taste prediction — use a structure-based predictor such as BitterPredict instead.
- Spectra are in negative ion mode or are predicted/simulated spectra rather than experimental positive-mode EI-MS or ESI-MS/MS, which is outside the current BitterMasS implementation.

## Inputs

- Experimental EI-MS mass spectra (e.g. from MassBank)
- Experimental ESI-MS/MS positive ion mode mass spectra (e.g. from MassBank)
- Instrument-type metadata for each spectrum

## Outputs

- Fixed-length binned spectral descriptor vectors (0.1 m/z bins, intensities as bin values, plus instrument-type descriptor columns)
- Descriptor matrix suitable for Random Forest training/testing

## How to apply

Retrieve experimental EI-MS and ESI-MS/MS positive ion mode mass spectra from the MassBank database for each compound in the bitter/non-bitter set. Convert each spectrum into a fixed-length spectral descriptor by binning peaks into 0.1 m/z bins with peak intensities as bin values, and append instrument-type descriptors to the vector. Keep all spectra of one compound together when later splitting data (compound-level partitioning, e.g. 80%/20% repeated 20 times), since spectra from the same compound must stay in the same fold. Feed the resulting descriptor matrix directly into the classifier (BitterMasS) rather than going through a structure-annotation step, because the article shows direct spectrum-to-property prediction is more effective when structures are unknown. When compounds lack spectra (only ~30% of training compounds had at least one identified spectrum), they are simply absent from the feature matrix.

## Related tools

- **BitterMasS** (Downstream Random Forest classifier trained on the binned spectral descriptors to predict bitterness from spectra) — https://github.com/Niv-Lab/BitterPredict1
- **MassBank** (Source database from which experimental EI-MS and ESI-MS/MS positive ion mode spectra are retrieved for binning)
- **BitterPredict** (Alternative structure-based predictor to compare against the direct spectrum-based route; used in the spectrum-to-structure-to-bitterness pipeline) — https://github.com/Niv-Lab/BitterPredict1

## Examples

```
python bitter_mass.py --input massbank_spectra.csv --bins 0.1 --instrument --out descriptors.csv
```

## Evaluation signals

- Each descriptor vector has a fixed length determined by the 0.1 m/z binning, with bin values equal to peak intensities plus appended instrument-type descriptors.
- All spectra of a single compound fall in the same fold across the 20 repeated 80%/20% compound-level splits (no compound leakage).
- Downstream BitterMasS internal-test performance reaches approximately precision = 0.83 and recall = 0.90, exceeding the naive baseline (F1 = 0.67, BA = 0.51).
- Predictions on the combined EI-MS + ESI-MS/MS set outperform predictions made on the individual EI-MS or ESI-MS/MS sets alone.

## Limitations

- Only ~30% of compounds in the initial bitter and non-bitter training sets had at least one identified mass spectrum, limiting the training data.
- Nonbitter compound spectral representation in the training set is insufficient; performance gains are expected from increasing it.
- Most articles reporting experimental mass spectra do not include the actual spectra; standardized mass spectral deposition is lacking.
- Predictions on individual EI-MS and ESI-MS/MS sets alone showed poor results compared with the combined set.
- The current implementation only supports EI-MS or ESI-MS/MS spectra obtained in positive ion mode; no version number or separate code hosting is stated for BitterMasS.

## Evidence

- [results] Binning of peaks into fixed-length descriptor: [evidence span withheld — anchor status: not_found]
- [discussion] Combined sets outperform individual: "Prediction on the individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set."
- [discussion] Only ~30% with spectra: "only 30% of the compounds in the initial bitter and nonbitter sets in the training set had at least one mass spectrum identified"
- [discussion] Positive ion mode only: "In the current implementation, the model makes predictions based on EI-MS or ESI-MS/MS spectra obtained in positive ion mode."
- [results] Compound-level splitting: [evidence span withheld — anchor status: not_found]
