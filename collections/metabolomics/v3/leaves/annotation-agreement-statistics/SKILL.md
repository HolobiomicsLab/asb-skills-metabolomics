---
name: annotation-agreement-statistics
description: Use when comparing compound annotations produced by two spectral libraries
  (e.g. CalicoLipids vs MS-DIAL) over the same feature table and you must reproduce
  per-library, shared, and concordance counts stratified by instrument or ionization
  mode.
when_to_use_negative:
- Input is a single-library search result with no second library to compare against
  — the library-specific/shared partition cannot be computed
- Features have not yet been detected or aligned across datasets (no 12,054-feature
  table exists) — perform feature detection first in MAVEN2
- The analysis goal is spectral matching quality assessment with spiked standards
  rather than counting and concordance of annotations
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: MAVEN2
  role: Performs the dual-library LC-MS/MS spectral search with RBP-threshold filtering
    and produces the annotation results being counted and compared
  repo: https://github.com/eugenemel/maven
- name: CalicoLipidLibrary
  role: Generates the in-silico CalicoLipids MSP fragmentation libraries (e.g. via
    generateDB.py) that supply one side of the dual-library comparison
  repo: https://github.com/calico/CalicoLipidLibrary
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:04f9ba03ce45cdc1513f5b381809e48043146139a2ba1985e14fd3debcdf8f2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - CalicoLipidLibrary
  - MAVEN2
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# annotation-agreement-statistics

## Summary

Reproduces lipidomics annotation counting and library-agreement statistics from the MAVEN2 dual-library (CalicoLipids vs MS-DIAL) search: partitioning RBP-thresholded annotations into library-specific and shared sets and quantifying lipid class and composition concordance.

## When to use

Use when comparing compound annotations produced by two spectral libraries (e.g. CalicoLipids vs MS-DIAL) over the same feature table and you must reproduce per-library, shared, and concordance counts stratified by instrument or ionization mode.

## When NOT to use

- Input is a single-library search result with no second library to compare against — the library-specific/shared partition cannot be computed
- Features have not yet been detected or aligned across datasets (no 12,054-feature table exists) — perform feature detection first in MAVEN2
- The analysis goal is spectral matching quality assessment with spiked standards rather than counting and concordance of annotations

## Inputs

- Dual-library annotation results from CalicoLipids and MS-DIAL searches (MAVEN2 output)
- Feature table containing the 12,054 unique features detected across the lipidomics datasets
- Library-specific Retention Balance Point (RBP) score thresholds

## Outputs

- annotation_agreement_table.csv with all counts and percentages
- Counts of CalicoLipids-only (1,009), MS-DIAL-only (461), and shared (1,142) annotations
- Class-level agreement (94%) and full-structure agreement (87%) statistics for shared annotations
- Table 2-style stratification of identifications by dataset, instrument, and ionization mode

## How to apply

Load the dual-library annotation results together with the feature table of 12,054 unique features and apply the library-specific Retention Balance Point (RBP) score thresholds to filter annotations, yielding 2,612 annotated features (~21% of features). Partition the retained annotations into CalicoLipids-only (1,009), MS-DIAL-only (461), and shared (1,142) sets and count each category. For the 1,142 shared annotations, compute class-level agreement (1,078/1,142 = ~94%) and full-structure agreement on class, summed composition, and adduct form (990/1,142 = ~87%). Finally, stratify all identifications by dataset, instrument, and ionization mode to reproduce the Table 2 breakdown; validation requires all reproduced counts to exactly equal the reported values.

## Related tools

- **MAVEN2** (Performs the dual-library LC-MS/MS spectral search with RBP-threshold filtering and produces the annotation results being counted and compared) — https://github.com/eugenemel/maven
- **CalicoLipidLibrary** (Generates the in-silico CalicoLipids MSP fragmentation libraries (e.g. via generateDB.py) that supply one side of the dual-library comparison) — https://github.com/calico/CalicoLipidLibrary

## Examples

```
python2.7 generateDB.py -m "pos" -c "PC,PE,PS" -n "pos_glycerophospholipids.msp"
```

## Evaluation signals

- Reproduced annotated-feature count equals 2,612 out of 12,054 unique features (~21%)
- Partition sums exactly: 1,009 (CalicoLipids-only) + 461 (MS-DIAL-only) + 1,142 (shared) = 2,612
- Class-level agreement of shared annotations equals 1,078/1,142 (~94%)
- Full agreement on class, summed composition, and adduct form equals 990/1,142 (~87%)
- CalicoLipids returns more identifications than MS-DIAL consistently across every instrument and ionization-mode stratum in the Table 2 breakdown

## Limitations

- Any deviation of reproduced counts from the reported values (2,612/12,054 annotated, 1,009+461+1,142 partition, 94% class-level, 87% full agreement) indicates an incorrect threshold application
- MAVEN2 installers are limited to macOS and Windows as of 2024-11-05; Linux builds have been retired
- Enumerating all theoretical spectra over all lipid classes with CalicoLipidLibrary can be time-consuming; subset generation with the -c option may be needed

## Evidence

- [results] dual-library annotation counts: [evidence span withheld — anchor status: not_found]
- [results] agreement metrics: [evidence span withheld — anchor status: not_found]
- [readme] library generation: [evidence span withheld — anchor status: external_doc]
- [abstract] tool role: "MAVEN2, which supports LC-MS/MS analysis of metabolomics and lipidomics samples"
- [supplementary] validation criterion: [evidence span withheld — anchor status: not_found]
