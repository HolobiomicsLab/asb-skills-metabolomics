---
name: msms-fragment-library-searching
description: Use when an aligned feature table contains m/z and retention-time pairs
  whose monoisotopic precursor masses need compound-level annotation via MS/MS spectral
  matching against an online MassBank library.
when_to_use_negative:
- Input is raw .mzML data that has not yet gone through MSS peak picking (mssmain.peak_pick)
  and alignment — fragment matching expects an aligned feature table
- No MS/MS spectra are available for the features (the workflow relies on MS/MS fragment
  spectra uploaded to MoNA)
- Candidate formula prediction is needed for molecules above 500 Da — MSS formula
  prediction supports small molecules under 500 Da
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Mass-Suite (MSS)
  role: Hosts the fragment module (frag.py) that uploads MS/MS spectra to the online
    database and retrieves matching results
  repo: https://github.com/XiminHu/mass-suite
- name: MoNA MassBank database
  role: Online spectral library endpoint used for all MS/MS fragment searching
- name: MassBank of North America
  role: MS/MS fragment matching database used to facilitate compound identification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:9a419dfe65a5de09d8caca2c8007f27b44efeda25f2f240c0c9f7b8edc8f2345
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiminHu/mass-suite
  tools:
  - Mass-Suite (MSS)
  - MassBank of North America
  - MoNA MassBank database
  - Python
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9047619047619048
n_claims: 42
---

# msms-fragment-library-searching

## Summary

Compound identification by uploading individual or averaged MS/MS spectra through the MSS fragment module (frag.py) to the MassBank of North America (MoNA) online database and retrieving matches following MassBank search criteria, as a stage of MSS's HRMS non-targeted analysis workflow.

## When to use

Use when an aligned feature table contains m/z and retention-time pairs whose monoisotopic precursor masses need compound-level annotation via MS/MS spectral matching against an online MassBank library.

## When NOT to use

- Input is raw .mzML data that has not yet gone through MSS peak picking (mssmain.peak_pick) and alignment — fragment matching expects an aligned feature table
- No MS/MS spectra are available for the features (the workflow relies on MS/MS fragment spectra uploaded to MoNA)
- Candidate formula prediction is needed for molecules above 500 Da — MSS formula prediction supports small molecules under 500 Da

## Inputs

- Aligned feature table exported as .csv containing m/z and retention-time pairs
- Individual or averaged MS/MS spectra for features of interest

## Outputs

- annotated_features.csv with formula assignments, isotopic scores, and MS/MS match results

## How to apply

Take the aligned feature table exported from the upstream peak-detection/alignment stage (a .csv of m/z and retention-time pairs) and select the features targeted for identification. For each feature, upload its individual or averaged MS/MS spectrum through the MSS fragment module (frag.py); all online searching uses the MoNA MassBank database endpoint, following MassBank database searching criteria and protocols. Matches retrieved from MoNA are written into the annotated feature table together with formula assignments (from ChemCalc-based formula prediction for small molecules under 500 Da) and dot-product-based isotopic scores comparing theoretical isotopologue patterns against observed spectra. The final output is an annotated_features.csv containing formula assignments, isotopic scores, and MS/MS match results.

## Related tools

- **Mass-Suite (MSS)** (Hosts the fragment module (frag.py) that uploads MS/MS spectra to the online database and retrieves matching results) — https://github.com/XiminHu/mass-suite
- **MoNA MassBank database** (Online spectral library endpoint used for all MS/MS fragment searching)
- **MassBank of North America** (MS/MS fragment matching database used to facilitate compound identification)

## Examples

```
from mss.frag import Frag; Frag.msms_search('sample_msms.csv', 'annotated_features.csv')
```

## Evaluation signals

- annotated_features.csv is produced and contains, for each queried feature, formula assignments, isotopic scores, and MS/MS match results
- MS/MS matches are retrieved against the MoNA MassBank endpoint and follow MassBank database searching criteria and protocols
- Each annotated feature's formula candidates were generated for small molecules under 500 Da via the ChemCalc API, with dot-product-based isotopic scores recorded
- The input feature table originated from the upstream MSS peak-detection/alignment stage (m/z and RT pairs), confirming correct pipeline ordering

## Limitations

- Formula prediction supports only small molecules under 500 Da
- Feature grouping (merging isotopes, adducts, and in-source fragments into individual chemicals) is not yet implemented in MSS — listed as future work
- All searching is online, so the fragment module depends on availability of the MoNA MassBank database endpoint
- Users are cautioned to develop fundamental understanding about the algorithms, and their limitations and assumptions to avoid misinterpretation

## Evidence

- [methods] Online database searching: "to the MassBank of North America database [68] for MS/MS fragment matching to facilitate identification"
- [methods] MoNA endpoint: "All the online searching is using [MoNA MassBank database](https://mona.fiehnlab.ucdavis.edu/)."
- [readme] README fragment module: "It provided a full capacity including data import, alignment, noise removal, visualization and fragment searching using online database."
- [readme] Formula prediction limit: "Updated formula prediction and isotopic score function, currently support formula prediction for small molecules under 500 Da"
- [abstract] Basic functions list: "including both basic functions (e.g., feature extraction, data reduction, feature annotation, data visualization, and statistical analyses)"
