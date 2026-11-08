---
name: hrms-chemical-formula-prediction
description: Use when you already have an aligned HRMS feature table (.csv of m/z
  and retention-time pairs) and need formula assignments, isotopic scores, and MS/MS-based
  compound identification for each feature.
when_to_use_negative:
- The feature masses exceed 500 Da — MSS formula prediction currently supports only
  small molecules under 500 Da
- Input is raw .mzML data that has not yet gone through MSS peak picking and alignment
  — the annotation stage expects an already-aligned m/z–RT feature table
- No MS/MS spectra are available — the workflow will yield only formula candidates
  and isotopic scores, not compound identifications
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Mass-Suite (MSS)
  role: Provides the annotation function that submits precursor masses to ChemCalc
    and the fragment module that uploads MS/MS spectra to MoNA
  repo: https://github.com/XiminHu/mass-suite
- name: Python
  role: Implementation language for the MSS annotation and fragment modules; installed
    via pip
  repo: https://github.com/XiminHu/mass-suite
- name: MassBank of North America
  role: Target database for MS/MS fragment matching to facilitate compound identification
- name: MoNA MassBank database
  role: Online endpoint used by all MSS online fragment searching
  repo: https://mona.fiehnlab.ucdavis.edu/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:574958ec9b6e3ba47c8aee514d0e9ceb0c1dfcbf4eb00dda104394aac032b372
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiminHu/mass-suite
  tools:
  - Mass-Suite (MSS)
  - MassBank of North America
  - MoNA MassBank database
  - Python
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# hrms-chemical-formula-prediction

## Summary

Assigns candidate chemical formulae to HRMS features by submitting monoisotopic precursor masses to ChemCalc via the Mass-Suite (MSS) annotation function, scoring candidates with a dot-product isotopic-pattern comparison, and adding MS/MS matches from the MoNA MassBank database.

## When to use

Use when you already have an aligned HRMS feature table (.csv of m/z and retention-time pairs) and need formula assignments, isotopic scores, and MS/MS-based compound identification for each feature.

## When NOT to use

- The feature masses exceed 500 Da — MSS formula prediction currently supports only small molecules under 500 Da
- Input is raw .mzML data that has not yet gone through MSS peak picking and alignment — the annotation stage expects an already-aligned m/z–RT feature table
- No MS/MS spectra are available — the workflow will yield only formula candidates and isotopic scores, not compound identifications

## Inputs

- Aligned feature table (.csv) containing m/z and retention-time pairs from the upstream MSS peak-detection/alignment stage
- Monoisotopic precursor masses for each feature (< 500 Da)
- Individual or averaged MS/MS spectra per feature
- Raw HRMS data in .mzML format (for regenerating spectra if needed)

## Outputs

- annotated_features.csv containing formula assignments, dot-product isotopic scores, and MoNA MS/MS match results

## How to apply

Load the aligned feature table exported from the upstream MSS peak-detection/alignment stage as a .csv containing m/z and retention-time pairs. For each feature, submit its monoisotopic precursor mass to the ChemCalc online API through the MSS annotation function; formula prediction supports small molecules under 500 Da, so restrict to features below that mass. Evaluate each candidate formula's prediction accuracy with a dot-product-based isotopic score that compares theoretical isotopologue patterns against the observed spectra. For compound identification, upload individual or averaged MS/MS spectra through the MSS fragment module to the MassBank of North America database (MoNA endpoint) and retrieve matches following MassBank database searching criteria and protocols. Finally, write the annotated feature table — formula assignments, isotopic scores, and MS/MS match results — to annotated_features.csv.

## Related tools

- **Mass-Suite (MSS)** (Provides the annotation function that submits precursor masses to ChemCalc and the fragment module that uploads MS/MS spectra to MoNA) — https://github.com/XiminHu/mass-suite
- **Python** (Implementation language for the MSS annotation and fragment modules; installed via pip) — https://github.com/XiminHu/mass-suite
- **MassBank of North America** (Target database for MS/MS fragment matching to facilitate compound identification)
- **MoNA MassBank database** (Online endpoint used by all MSS online fragment searching) — https://mona.fiehnlab.ucdavis.edu/

## Examples

```
pip install mass-suite
import mss
```

## Evaluation signals

- annotated_features.csv is produced with columns covering formula assignments, isotopic scores, and MS/MS match results
- Every formula candidate row carries a dot-product-based isotopic score comparing theoretical isotopologue patterns to the observed spectrum
- Formula predictions are only present for features with monoisotopic mass under 500 Da (the documented ChemCalc prediction limit)
- MS/MS matches correspond to MoNA/MassBank of North America search results and follow MassBank database searching criteria and protocols
- Annotated features can be joined back to the upstream aligned table on m/z and retention-time pairs without dropping rows

## Limitations

- Formula prediction supports only small molecules under 500 Da, so larger features cannot be annotated
- Feature grouping (merging isotopes, adducts, and in-source fragments as individual chemicals) is not yet implemented in MSS and is listed as future work
- Online searching depends on the external MoNA MassBank database endpoint availability
- The MSS package version is not stated in the article and no changelog or archived release (e.g., Zenodo DOI) is cited, complicating reproducibility of the annotation results

## Evidence

- [abstract] feature annotation stage: "including both basic functions (e.g., feature extraction, data reduction, feature annotation, data visualization, and statistical analyses)"
- [methods] MoNA as search target: "All the online searching is using [MoNA MassBank database](https://mona.fiehnlab.ucdavis.edu/)."
- [methods] MassBank MS/MS fragment matching: "to the MassBank of North America database [68] for MS/MS fragment matching to facilitate identification"
- [readme] 500 Da formula prediction limit and isotopic score: "Updated formula prediction and isotopic score function, currently support formula prediction for small molecules under 500 Da"
