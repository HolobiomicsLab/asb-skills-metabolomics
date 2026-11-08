---
name: ionization-mode-stratified-reporting
description: Use when a dual-library or library-vs-library annotation search must
  be broken down per instrument and per positive/negative ionization mode to reproduce
  per-instrument identification counts and mode-specific agreement statistics.
when_to_use_negative:
- Input is a single-library search with no second library to compare against, so no
  per-library stratified agreement can be computed
- Feature metadata lacks instrument and ionization-mode labels, making the per-stratum
  breakdown impossible to reproduce
- The goal is MS/MS spectral quality assessment (e.g., spiked-standard separation)
  rather than annotation-count reporting across datasets
tools:
- name: MAVEN2
  role: Performs the dual-library LC-MS/MS annotation searches and supplies the per-feature
    identifications that are then stratified by instrument and ionization mode
  repo: https://github.com/eugenemel/maven
- name: CalicoLipidLibrary
  role: Generates the mode-specific (positive/negative) in silico lipid fragmentation
    MSP libraries that define the ionization-mode strata
  repo: https://github.com/calico/CalicoLipidLibrary
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:5c93d345a65a48fe52994c77b175ac68d79223195b0a5ba1438926dce29b9cd7
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
n_claims: 8
---

# ionization-mode-stratified-reporting

## Summary

Stratify lipidomics compound annotations by dataset, instrument, and ionization mode (positive/negative) so that library comparison counts (e.g., CalicoLipids vs MS-DIAL) are reported per stratum as in Table 2 of the MAVEN2 dual-library lipidomics search.

## When to use

Use when a dual-library or library-vs-library annotation search must be broken down per instrument and per positive/negative ionization mode to reproduce per-instrument identification counts and mode-specific agreement statistics.

## When NOT to use

- Input is a single-library search with no second library to compare against, so no per-library stratified agreement can be computed
- Feature metadata lacks instrument and ionization-mode labels, making the per-stratum breakdown impossible to reproduce
- The goal is MS/MS spectral quality assessment (e.g., spiked-standard separation) rather than annotation-count reporting across datasets

## Inputs

- Dual-library annotation results (CalicoLipids and MS-DIAL) with RBP scores per feature
- Feature table of 12,054 unique features detected across the lipidomics datasets, with dataset, instrument, and ionization-mode metadata
- In silico lipid fragmentation libraries in MSP format, generated per ionization mode (pos/neg)

## Outputs

- annotation_agreement_table.csv with per-dataset, per-instrument, and per-ionization-mode identification counts (Table 2 reproduction)
- Per-stratum library agreement percentages (94% class-level, 87% class + summed composition + adduct)
- Partitioned annotation sets: CalicoLipids-only (1,009), MS-DIAL-only (461), shared (1,142)

## How to apply

After applying the library-specific Retention Balance Point (RBP) score thresholds to the dual CalicoLipids vs MS-DIAL search results, partition the 2,612 annotated features (of 12,054 unique features) into CalicoLipids-only (1,009), MS-DIAL-only (461), and shared (1,142) sets. For each stratum, tag every annotation with its source dataset, instrument, and ionization mode, then compute per-mode identification counts and the class-level (1,078/1,142 = 94%) and full-structure (990/1,142 = 87%) agreement between libraries. Libraries must be searched in both ionization modes separately — the CalicoLipidLibrary generates mode-specific MSP files via the -m pos|neg flag — because adducts and lipid-class coverage differ by mode. Write the per-stratum counts to annotation_agreement_table.csv and verify the strata sum to the overall partition before interpreting mode-dependent trends such as CalicoLipids returning more identifications than MS-DIAL.

## Related tools

- **MAVEN2** (Performs the dual-library LC-MS/MS annotation searches and supplies the per-feature identifications that are then stratified by instrument and ionization mode) — https://github.com/eugenemel/maven
- **CalicoLipidLibrary** (Generates the mode-specific (positive/negative) in silico lipid fragmentation MSP libraries that define the ionization-mode strata) — https://github.com/calico/CalicoLipidLibrary

## Examples

```
python2.7 generateDB.py -m "pos" -c "DG,PA,PC,PE,PG,PI,PS,BMP,HexDG,MMPE,DMPE,FAFHFA"
```

## Evaluation signals

- Annotated-feature count equals 2,612 out of 12,054 unique features
- Partition sums exactly: 1,009 (CalicoLipids-only) + 461 (MS-DIAL-only) + 1,142 (shared) = 2,612
- Shared-annotation agreement equals 1,078/1,142 (94%) at lipid-class level and 990/1,142 (87%) for class + summed composition + adduct
- Per-instrument and per-ionization-mode counts reproduce the Table 2 breakdown with CalicoLipids exceeding MS-DIAL identifications in every stratum
- Each stratum's counts sum to the overall totals across all modes and instruments

## Limitations

- Agreement statistics depend on the chosen RBP thresholds; misapplied thresholds shift the partition counts away from the validated values
- Ionization-mode stratification requires the library to have been enumerated separately per mode; unsupported class/adduct combinations in one mode reduce that stratum's coverage
- Counts are specific to the datasets and instruments used in the study; new data will not reproduce the exact 1,009/461/1,142 partition

## Evidence

- [methods] Dual-library RBP-thresholded counts: [evidence span withheld — anchor status: not_found]
- [results] Agreement percentages: [evidence span withheld — anchor status: not_found]
- [readme] CalicoLipidLibrary mode-specific generation: [evidence span withheld — anchor status: external_doc]
- [readme] generateDB ionization flag: "-m <pos|neg>:
	enumerate positive or negative mode spectra.
	DEFAULT: neg (negative ionization mode)"
- [abstract] MAVEN2 library search scope: "This manuscript describes a major update to the program, MAVEN2, which supports LC-MS/MS analysis of metabolomics and lipidomics samples."
