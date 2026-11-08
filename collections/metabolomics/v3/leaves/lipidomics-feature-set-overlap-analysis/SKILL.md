---
name: lipidomics-feature-set-overlap-analysis
description: Use when you have LC-MS/MS lipidomics feature annotations from two spectral
  libraries and need to quantify annotation overlap, library agreement on lipid class/composition/adduct,
  or reproduce per-instrument identification counts.
when_to_use_negative:
- You have only a single library's annotations and no second library to compare against
  — there is no overlap or agreement to quantify
- Your annotations were produced without RBP-score filtering (unthresholded matches),
  since the counts being reproduced depend specifically on the thresholded dual-library
  search
- The task is de novo lipid identification or spectral library construction rather
  than comparison of existing annotation results
tools:
- name: MAVEN2
  role: Open-source LC-MS/MS analysis platform used to run the dual-library lipidomics
    searches, perform MS/MS spectral matching against the fragmentation libraries,
    and produce the per-feature annotations being compared
  repo: https://github.com/eugenemel/maven
- name: CalicoLipidLibrary
  role: In-silico lipid fragmentation library (CalicoLipids side of the comparison),
    generated as MSP files per ionization mode and lipid class subset
  repo: https://github.com/calico/CalicoLipidLibrary
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:d86d418a1a22f948581d912d2d514cf1addb1f68c16bd0bb6080c49f3f9d0067
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - CalicoLipidLibrary
  - MAVEN2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# lipidomics-feature-set-overlap-analysis

## Summary

Reproduces the dual-library (CalicoLipids vs MS-DIAL) lipidomics annotation comparison described in the MAVEN2 paper by applying Retention Balance Point (RBP) score thresholds, partitioning annotated features into library-specific and shared sets, and computing class-level and full-structure agreement statistics.

## When to use

Use when you have LC-MS/MS lipidomics feature annotations from two spectral libraries and need to quantify annotation overlap, library agreement on lipid class/composition/adduct, or reproduce per-instrument identification counts.

## When NOT to use

- You have only a single library's annotations and no second library to compare against — there is no overlap or agreement to quantify
- Your annotations were produced without RBP-score filtering (unthresholded matches), since the counts being reproduced depend specifically on the thresholded dual-library search
- The task is de novo lipid identification or spectral library construction rather than comparison of existing annotation results

## Inputs

- Dual-library lipidomics annotation results (CalicoLipids search vs MS-DIAL search) with per-feature RBP scores
- Feature table of unique LC-MS/MS features across all lipidomics datasets (reference study: 12,054 unique features)
- Dataset metadata mapping features to instrument type and ionization mode (positive/negative)
- Library-specific Retention Balance Point (RBP) score thresholds

## Outputs

- Counts of annotated vs total unique features (reference: 2,612/12,054)
- Partition counts of CalicoLipids-only (1,009), MS-DIAL-only (461), and shared (1,142) annotations
- Agreement percentages for shared annotations: lipid-class level (~94%) and full class+composition+adduct level (~87%)
- Per-dataset, per-instrument, and per-ionization-mode identification breakdown (Table 2 equivalent)
- annotation_agreement_table.csv containing all counts and percentages

## How to apply

Load the dual-library annotation results together with the feature table of all unique features (12,054 in the reference study). Apply the library-specific RBP score thresholds to filter annotations in both searches, retaining only confident identifications (2,612 annotated features, ~21%, in the reference run). Partition the retained features into CalicoLipids-only, MS-DIAL-only, and shared sets and count each category (expected partition: 1,009 + 461 + 1,142). For the shared set, compute agreement at the lipid-class level (~94%, 1078/1142) and on the full structure including class, summed composition, and adduct form (~87%, 990/1142). Finally stratify identifications by dataset, instrument, and ionization mode to reproduce the per-instrument breakdown and confirm that CalicoLipids returns more identifications than MS-DIAL independent of instrument or ionization mode.

## Related tools

- **MAVEN2** (Open-source LC-MS/MS analysis platform used to run the dual-library lipidomics searches, perform MS/MS spectral matching against the fragmentation libraries, and produce the per-feature annotations being compared) — https://github.com/eugenemel/maven
- **CalicoLipidLibrary** (In-silico lipid fragmentation library (CalicoLipids side of the comparison), generated as MSP files per ionization mode and lipid class subset) — https://github.com/calico/CalicoLipidLibrary

## Examples

```
python2.7 generateDB.py -m "neg"
```

## Evaluation signals

- Annotated-feature count must equal 2,612 out of 12,054 unique features (~21%)
- Partition must satisfy 1,009 (CalicoLipids-only) + 461 (MS-DIAL-only) + 1,142 (shared) = 2,612
- Shared-set class-level agreement must equal 1078/1142 ≈ 94%
- Shared-set full-structure (class + summed composition + adduct) agreement must equal 990/1142 ≈ 87%
- Per-instrument/per-ionization stratification must show CalicoLipids returning more identifications than MS-DIAL in every stratum

## Limitations

- Reproduced counts are only valid when the exact library-specific RBP thresholds are applied; any deviation indicates incorrect threshold application
- Agreement is only measured on the shared set — library-specific annotations cannot be cross-validated against the other library
- CalicoLipidLibrary library enumeration over all lipid classes may be time-consuming; the README recommends subsetting classes with the -c option
- CalicoLipidLibrary spectra generation runs under Python 2 only (e.g., python2.7 generateDB.py)

## Evidence

- [other] MAVEN2 dual-library annotation counts: [evidence span withheld — anchor status: not_found]
- [other] Shared annotation agreement: [evidence span withheld — anchor status: not_found]
- [abstract] MAVEN2 MS/MS spectral matching: "We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries."
- [abstract] In-silico library comparison: "we introduce a novel in-silico lipidomics library covering major lipid classes and compare searches using our novel library to searches with existing in-silico lipidomics libraries."
- [readme] CalicoLipidLibrary MSP generation: [evidence span withheld — anchor status: external_doc]
