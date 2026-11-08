---
name: lipid-annotation-threshold-filtering
description: Use when raw LC-MS/MS lipidomics feature annotations must be filtered
  by RBP score thresholds and compared between two fragmentation libraries to reproduce
  annotation counts and agreement statistics.
when_to_use_negative:
- Input is already a filtered annotation table with no raw scores available to re-apply
  RBP thresholds
- The task concerns a single library search rather than a dual-library (CalicoLipids
  vs MS-DIAL) comparison, since library-specific thresholds and overlap statistics
  are not applicable
- Data are untargeted metabolomics (non-lipid) features outside the lipid classes
  covered by the in-silico libraries
edam_operation: http://edamontology.org/operation_3695
tools:
- name: MAVEN2
  role: LC-MS/MS analysis platform used to run MS/MS spectral matching of dataset
    features against the CalicoLipids and MS-DIAL in-silico lipid fragmentation libraries
    and apply RBP score thresholds
  repo: https://github.com/eugenemel/maven
- name: CalicoLipidLibrary
  role: Python package that generates the in-silico CalicoLipids MSP fragmentation
    files (per ionization mode and lipid class) used as the search library
  repo: https://github.com/calico/CalicoLipidLibrary
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:f451a8156583936e22a923ba4cfe713bc6b573ca9798cdbe62b8687604ead14b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - CalicoLipidLibrary
  - MAVEN2
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# lipid-annotation-threshold-filtering

## Summary

Filters lipidomics compound annotations from dual in-silico library searches (CalicoLipids vs MS-DIAL) using library-specific Retention Balance Point (RBP) score thresholds in MAVEN2, then quantifies annotation overlap and library agreement at lipid-class and full-structure level.

## When to use

Use when raw LC-MS/MS lipidomics feature annotations must be filtered by RBP score thresholds and compared between two fragmentation libraries to reproduce annotation counts and agreement statistics.

## When NOT to use

- Input is already a filtered annotation table with no raw scores available to re-apply RBP thresholds
- The task concerns a single library search rather than a dual-library (CalicoLipids vs MS-DIAL) comparison, since library-specific thresholds and overlap statistics are not applicable
- Data are untargeted metabolomics (non-lipid) features outside the lipid classes covered by the in-silico libraries

## Inputs

- Dual-library annotation results from MAVEN2 searches against CalicoLipids and MS-DIAL in-silico lipid libraries (MSP-format fragmentation libraries)
- Feature table containing the 12,054 unique LC-MS/MS features detected across the lipidomics datasets
- Library-specific RBP (Retention Balance Point) score thresholds for CalicoLipids and MS-DIAL

## Outputs

- annotation_agreement_table.csv with counts and percentages of unique features annotated by each library
- Partition counts: CalicoLipids-only (1,009), MS-DIAL-only (461), shared (1,142)
- Agreement metrics: 94% lipid-class-level agreement (1,078/1,142) and 87% class+composition+adduct agreement (990/1,142)
- Per-dataset, per-instrument, and per-ionization-mode stratification of identifications (Table 2 breakdown)

## How to apply

Load the dual-library (CalicoLipids vs MS-DIAL) annotation results together with the feature table of the 12,054 unique features detected across the lipidomics datasets. Apply the library-specific Retention Balance Point (RBP) score thresholds to filter annotations, retaining 2,612 annotated features (~21% of features) as the comparison set. Partition the retained annotations into CalicoLipids-only (1,009), MS-DIAL-only (461), and shared (1,142) sets and count each category. For the 1,142 shared annotations, compute agreement at lipid-class level (1,078/1,142 = 94%) and on class, summed composition, and adduct form (990/1,142 = 87%). Finally, stratify all identifications by dataset, instrument, and ionization mode (positive/negative) to build the per-instrument breakdown; the rationale is that RBP thresholds separate authentic from spurious metabolite identifications, as evaluated in MAVEN2 with standards spiked into water and yeast backgrounds.

## Related tools

- **MAVEN2** (LC-MS/MS analysis platform used to run MS/MS spectral matching of dataset features against the CalicoLipids and MS-DIAL in-silico lipid fragmentation libraries and apply RBP score thresholds) — https://github.com/eugenemel/maven
- **CalicoLipidLibrary** (Python package that generates the in-silico CalicoLipids MSP fragmentation files (per ionization mode and lipid class) used as the search library) — https://github.com/calico/CalicoLipidLibrary

## Examples

```
python2.7 generateDB.py -m "neg"
```

## Evaluation signals

- Total annotated features must equal 2,612 out of 12,054 unique features (~21%) after applying the RBP thresholds
- Partition must satisfy 1,009 (CalicoLipids-only) + 461 (MS-DIAL-only) + 1,142 (shared) = 2,612
- Shared-annotation class-level agreement must equal 1,078/1,142 = 94%
- Shared-annotation full agreement (class, summed composition, adduct form) must equal 990/1,142 = 87%
- Stratified counts must show CalicoLipids returning more identifications than MS-DIAL independent of instrument or ionization mode; any deviation indicates incorrect threshold application

## Limitations

- Only ~21% of the 12,054 unique features received any annotation, even after thresholding; most features remain unassigned
- Libraries disagree on ~6% of shared annotations at class level and ~13% on full structure (class, summed composition, adduct), so shared annotations are not always consistent
- Enumerating all theoretical spectra over all lipid classes with CalicoLipidLibrary can be time-consuming; subset generation by class is recommended
- CalicoLipidLibrary supports only python 2 for interactive spectrum printing, and not all adduct names are valid for all lipid classes

## Evidence

- [abstract] MAVEN2 supports LC-MS/MS lipidomics searches: "This manuscript describes a major update to the program, MAVEN2, which supports LC-MS/MS analysis of metabolomics and lipidomics samples."
- [abstract] Algorithms separate authentic from spurious identifications: "We explore the ability of our approach to separate authentic from spurious metabolite identifications using a set of standards spiked into water and yeast backgrounds."
- [abstract] Novel in-silico lipidomics library introduced and compared: "we introduce a novel in-silico lipidomics library covering major lipid classes and compare searches using our novel library to searches with existing in-silico lipidomics libraries."
- [readme] CalicoLipidLibrary generates MSP files per ionization mode: [evidence span withheld — anchor status: external_doc]
