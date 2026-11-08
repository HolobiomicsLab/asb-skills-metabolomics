---
name: annotated-feature-list-export
description: Use when an aligned peak table (m/z, retention-time pairs) exported from
  a peak-detection/alignment stage needs formula assignments, isotopic-pattern scores,
  and MS/MS library identifications.
when_to_use_negative:
- Input is still raw .mzML data that has not been peak-picked and aligned — run MSS
  feature extraction (mssmain.peak_pick) and alignment first
- Features have precursor masses above 500 Da, for which the current formula prediction
  function does not support candidate generation
- No MS/MS spectra were acquired, since the compound-identification step relies on
  uploading spectra to the MoNA MassBank database
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: Mass-Suite (MSS)
  role: Python package whose annotation/fragment module submits precursor masses to
    ChemCalc and MS/MS spectra to MoNA, then writes the annotated feature table
  repo: https://github.com/XiminHu/mass-suite
- name: MoNA MassBank database
  role: Online endpoint searched with MS/MS spectra for compound identification following
    MassBank searching criteria
- name: MassBank of North America
  role: MS/MS fragment-matching database used to facilitate compound identification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:595958ccc8634aede1c985bdf3992dcd586d99aa1be5e18abff45e35712fa88b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiminHu/mass-suite
  tools:
  - Mass-Suite (MSS)
  - MassBank of North America
  - MoNA MassBank database
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# annotated-feature-list-export

## Summary

Annotates an aligned HRMS feature table by assigning candidate molecular formulae via ChemCalc prediction plus dot-product isotopic scoring, and matching MS/MS spectra against the MoNA MassBank database, producing an annotated_features.csv output.

## When to use

Use when an aligned peak table (m/z, retention-time pairs) exported from a peak-detection/alignment stage needs formula assignments, isotopic-pattern scores, and MS/MS library identifications.

## When NOT to use

- Input is still raw .mzML data that has not been peak-picked and aligned — run MSS feature extraction (mssmain.peak_pick) and alignment first
- Features have precursor masses above 500 Da, for which the current formula prediction function does not support candidate generation
- No MS/MS spectra were acquired, since the compound-identification step relies on uploading spectra to the MoNA MassBank database

## Inputs

- Aligned feature table as .csv containing m/z and retention-time pairs from the upstream peak-detection/alignment stage
- Individual or averaged MS/MS spectra for features to be identified

## Outputs

- annotated_features.csv with formula assignments, isotopic scores, and MS/MS match results

## How to apply

Load the aligned feature table as a .csv containing m/z and retention-time pairs, then submit each feature's monoisotopic precursor mass to the ChemCalc online API through the MSS annotation function; formula prediction supports small molecules under 500 Da. Rank each candidate formula with a dot-product-based isotopic score that compares theoretical isotopologue patterns against the observed spectra, and keep the best-scoring assignment. For identification, upload individual or averaged MS/MS spectra through the MSS fragment module to the MoNA MassBank database endpoint, retrieving matches that follow MassBank database searching criteria and protocols. Write the final annotated feature table — formula assignments, isotopic scores, and MS/MS match results — to annotated_features.csv.

## Related tools

- **Mass-Suite (MSS)** (Python package whose annotation/fragment module submits precursor masses to ChemCalc and MS/MS spectra to MoNA, then writes the annotated feature table) — https://github.com/XiminHu/mass-suite
- **MoNA MassBank database** (Online endpoint searched with MS/MS spectra for compound identification following MassBank searching criteria)
- **MassBank of North America** (MS/MS fragment-matching database used to facilitate compound identification)

## Examples

```
python -m mss_processor.py
```

## Evaluation signals

- annotated_features.csv is produced and contains columns for formula assignments, isotopic dot-product scores, and MS/MS match results for every input feature
- Every feature receives formula candidates only if its monoisotopic precursor mass is under 500 Da (documented limit of the prediction function)
- Isotopic scores are dot-product-based comparisons of theoretical vs observed isotopologue patterns, so assignments should carry a quantitative isotopic score
- MS/MS identifications trace back to MoNA MassBank database searches, so match results should reference the MassBank searching criteria and protocols
- Feature m/z and retention-time values in the output must match the input aligned feature table 1:1

## Limitations

- Formula prediction currently supports small molecules only under 500 Da (README update 2020/09/07)
- Feature grouping to merge isotopes, adducts, and in-source fragments into individual chemicals is not yet implemented and is listed as future work
- MS/MS searching depends on online access to the MoNA MassBank database; results follow MassBank criteria and are subject to library coverage

## Evidence

- [abstract] Feature annotation stage of MSS workflow: "including both basic functions (e.g., feature extraction, data reduction, feature annotation, data visualization, and statistical analyses)"
- [methods] MS/MS matching against MassBank of North America: "to the MassBank of North America database [68] for MS/MS fragment matching to facilitate identification"
- [readme] MoNA endpoint for online searching: "All the online searching is using [MoNA MassBank database](https://mona.fiehnlab.ucdavis.edu/)."
- [readme] Formula prediction mass limit: "Updated formula prediction and isotopic score function, currently support formula prediction for small molecules under 500 Da"
