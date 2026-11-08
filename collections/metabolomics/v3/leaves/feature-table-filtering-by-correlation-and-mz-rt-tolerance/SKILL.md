---
name: feature-table-filtering-by-correlation-and-mz-rt-tolerance
description: Use when an LC-MS metabolite feature table contains suspected in-source
  fragment features that must be de novo recognized and filtered by correlation and
  m/z/RT tolerance, e.g. before metabolite annotation from DDA/DIA/fullscan mzXML
  data.
when_to_use_negative:
- When the goal is standard adduct/isotope annotation rather than ISF recognition
  — CAMERA adduct and isotope annotation can only be used for XCMS ONLY ISFrag analysis
  and is not a substitute for the ISF cascade.
- When only MS2 peak lists are available and no MS1 feature table (XCMS or custom
  csv with mz, rt, rtmin, rtmax, intensity columns) can be produced.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0091
tools:
- name: ISFrag
  role: R package whose find.level3()/find.level2()/find.level1() functions perform
    the correlation- and m/z/RT-tolerance-based ISF filtering cascade
  repo: https://github.com/HuanLab/ISFrag
- name: xcms
  role: Extracts the MS1 feature table (mz, rt, rtmin, rtmax, maxo/intensity) that
    the ISF cascade filters
- name: MSnbase
  role: Provides the spectral object output by XCMS.featuretable()/generate.featuretable()
    used for MS2 assignment
- name: CAMERA
  role: Optional adduct/isotope annotation, restricted to XCMS-only ISFrag analyses
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c01644
    title: 'ISFrag: De Novo Recognition of In-Source Fragments for Liquid Chromatography–Mass
      Spectrometry Data'
  source_content_hash: sha256:d64696f22d516d35b247a7279d99aedf1e2b5cdfe4a2482fae0c52ded024ea1d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/ISFrag
  tools:
  - CAMERA
  - ISFrag
  - MSnbase
  - xcms
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# feature-table-filtering-by-correlation-and-mz-rt-tolerance

## Summary

Applies ISFrag's three-level in-source fragment (ISF) recognition cascade — find.level3(), find.level2(), find.level1() — to filter an LC-MS metabolite feature table into ISF precursor-fragment relationships using co-elution/correlation patterns and m/z and retention-time tolerance. Benchmarked on metabolite standards at 100% level 1 and >80% level 2 correct recognition.

## When to use

Use when an LC-MS metabolite feature table contains suspected in-source fragment features that must be de novo recognized and filtered by correlation and m/z/RT tolerance, e.g. before metabolite annotation from DDA/DIA/fullscan mzXML data.

## When NOT to use

- When the goal is standard adduct/isotope annotation rather than ISF recognition — CAMERA adduct and isotope annotation can only be used for XCMS ONLY ISFrag analysis and is not a substitute for the ISF cascade.
- When only MS2 peak lists are available and no MS1 feature table (XCMS or custom csv with mz, rt, rtmin, rtmax, intensity columns) can be produced.

## Inputs

- MS1 fullscan mzXML file(s) in a dedicated folder (for XCMS feature extraction)
- DDA mzXML file(s) in a dedicated folder (for MS2 assignment)
- csv feature table with columns in order: mz, rt, rtmin, rtmax, then intensity columns (RT in seconds), e.g. from MS-DIAL or MZmine2
- Standard spectral library in msp format (e.g. MoNA-export-LC-MS-MS_Negative_Mode.msp)

## Outputs

- Dataframe feature table with ISF relationship annotated in additional columns (level 1/2/3 assignments)
- Detailed ISF precursor-fragment relationship dataframe
- ISF relationship tree export

## How to apply

First build the feature table with ISFrag: either XCMS.featuretable() on fullscan mzXML files (e.g. peakwidth = c(5,20)) or custom.featuretable() on a csv from MS-DIAL/MZmine2 whose columns are exactly m/z, retention time, min RT, max RT, then per-sample intensities, with all RTs in seconds. Attach MS2 spectra with ms2.assignment() using DDA mzXML files and annotate with feature.annotation() against a standard msp library (e.g. MoNA-export-LC-MS-MS_Negative_Mode.msp, dot-product score cutoff 0.1). Then run the three-level filtering cascade in strict order: find.level3() to detect candidate level 3 ISF features by correlation/co-elution patterns within m/z and RT tolerances, find.level2() to refine to level 2, and find.level1() on the level 2 output to finalize level 1 assignments; inspect tolerances and thresholds via help('<function name>'). Summarize with get.ISFrag.results() and export either the complete ISF-annotated feature table (export.ISFrag.results()) or the detailed precursor-fragment dataframe (export.ISFrag.detailed()). Judge success by recognition accuracy against metabolite-standard ground truth: 100% for level 1 and >80% for level 2 ISF features.

## Related tools

- **ISFrag** (R package whose find.level3()/find.level2()/find.level1() functions perform the correlation- and m/z/RT-tolerance-based ISF filtering cascade) — https://github.com/HuanLab/ISFrag
- **xcms** (Extracts the MS1 feature table (mz, rt, rtmin, rtmax, maxo/intensity) that the ISF cascade filters)
- **MSnbase** (Provides the spectral object output by XCMS.featuretable()/generate.featuretable() used for MS2 assignment)
- **CAMERA** (Optional adduct/isotope annotation, restricted to XCMS-only ISFrag analyses)

## Examples

```
install_github("HuanLab/ISFrag"); find.level3(...); find.level2(...); find.level1(...)
```

## Evaluation signals

- Recognition accuracy on metabolite-standard test data equals 100% for level 1 ISF features and exceeds 80% for level 2 ISF features.
- Feature table csv has exactly the required column order (m/z, retention time, min retention time, max retention time, intensities) with RT values in seconds.
- The cascade runs in strict order level 3 → level 2 → level 1, with find.level1() executed on the find.level2() output.
- get.ISFrag.results() returns a non-empty summary and export.ISFrag.results()/export.ISFrag.detailed() produce the annotated feature table and per-precursor relationship tables.
- CAMERA adduct/isotope columns are present only when the feature table came from XCMS, never from custom MS-DIAL/MZmine2 tables.

## Limitations

- Level 2 ISF recognition is only above 80%, not perfect; some level 2 fragments are misassigned or missed.
- Custom feature tables are strictly format-checked: fixed column order and retention times in seconds are required.
- Increasing collision energies raises feature counts, but the percentage of ISF features might not necessarily increase, so energy choice affects the filtered output proportion.

## Evidence

- [abstract] abstract: "Tested using metabolite standards, we achieved a 100% correct recognition of level 1 ISF features and over 80% correct recognition for level 2 ISF features."
- [methods] methods: "In-source fragments are identified from Level 3 to Level 1 fragments through functions `find.leve3()`"
- [readme] readme-format: "In order for `ISFrag` to succesfully read the provided csv file, it must contain only columns in the following order: m/z, retention time"
- [readme] readme-camera: "Note: CAMERA adduct and isotope annotation can only be used for `XCMS` ONLY `ISFrag` analysis."
