---
name: lipidomics-annotation-table-summarization
description: Use when comparing which lipid classes and adduct forms are enumerated
  or annotated across two or more in-silico lipid libraries, and you need to identify
  classes unique to a single library (e.g., OxPC only in MS-DIAL, CE only in CalicoLipids).
when_to_use_negative:
- The input is already a summarized count matrix or class-level composition table
  rather than raw annotation/library records
- The analysis goal is per-feature metabolite identification scoring rather than library/class
  coverage comparison
- Comparing experimental MS/MS spectral libraries against samples rather than comparing
  in-silico library compositions to each other
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3172
tools:
- name: MAVEN2
  role: Runs the LC-MS/MS library-comparison searches that produce the annotation
    tables and library composition statistics being summarized
  repo: https://github.com/eugenemel/maven
- name: Maven GUI
  role: Interactive interface (Metabolomics Analysis and Visualization Engine) for
    exploring annotation results from the lipidomics library searches
  repo: https://github.com/eugenemel/maven
- name: CalicoLipidLibrary
  role: Generates the in-silico lipid fragmentation spectra (MSP libraries) for enumerated
    lipid classes; its -c option subsets classes when regenerating library files for
    verification
  repo: https://github.com/calico/CalicoLipidLibrary
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:ac35d3bb1ffb367fa231c7b139cf1909daf507ea5d13de95795c5caf56edc65c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - CalicoLipidLibrary
  - MAVEN2
  - Maven GUI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# Lipidomics annotation-table summarization

## Summary

Summarizes thresholded lipidomics annotation tables and in-silico library compositions (e.g., CalicoLipids vs MS-DIAL searches run in MAVEN2) into a per-lipid-class, per-adduct count matrix, flagging classes represented in only one library.

## When to use

Use when comparing which lipid classes and adduct forms are enumerated or annotated across two or more in-silico lipid libraries, and you need to identify classes unique to a single library (e.g., OxPC only in MS-DIAL, CE only in CalicoLipids).

## When NOT to use

- The input is already a summarized count matrix or class-level composition table rather than raw annotation/library records
- The analysis goal is per-feature metabolite identification scoring rather than library/class coverage comparison
- Comparing experimental MS/MS spectral libraries against samples rather than comparing in-silico library compositions to each other

## Inputs

- Thresholded lipidomics annotation table from MAVEN2 library-comparison searches
- Library composition statistics for in-silico libraries (CalicoLipids, MS-DIAL, MAVEN2 novel library)
- MSP-format in-silico lipid fragmentation spectra libraries (optionally regenerated via CalicoLipidLibrary generateDB.py)

## Outputs

- Per-class, per-library count matrix (CSV, e.g., dataset_s5_lipid_class_counts.csv)
- List of lipid classes represented in only one library

## How to apply

Load the thresholded annotation table and library composition statistics exported from the MAVEN2 library-comparison searches (searches run against CalicoLipids, MS-DIAL, and MAVEN2's novel in-silico lipidomics library). Group records by lipid class and adduct form, then for each class/adduct combination count both library entries (e.g., SM: 14,076 vs 117 entries across libraries) and annotated features (e.g., Ceramide: 36,288 vs 39,882 features). Cross-compare counts across libraries to build a per-class, per-library count matrix. Flag any class present in only one library — the article reports OxPC present only in MS-DIAL and cholesterol esters (CE) present only in CalicoLipids. CalicoLipids supports subsetting classes during enumeration via generateDB.py's -c option when regenerating library subsets for verification.

## Related tools

- **MAVEN2** (Runs the LC-MS/MS library-comparison searches that produce the annotation tables and library composition statistics being summarized) — https://github.com/eugenemel/maven
- **Maven GUI** (Interactive interface (Metabolomics Analysis and Visualization Engine) for exploring annotation results from the lipidomics library searches) — https://github.com/eugenemel/maven
- **CalicoLipidLibrary** (Generates the in-silico lipid fragmentation spectra (MSP libraries) for enumerated lipid classes; its -c option subsets classes when regenerating library files for verification) — https://github.com/calico/CalicoLipidLibrary

## Examples

```
python2.7 generateDB.py -m "neg" -c "Ceramide,SM" -n "neg_sphingolipids.msp"
```

## Evaluation signals

- Per-class, per-adduct counts match the values reported in Figure 4 / Dataset S5 of the article (e.g., SM 14,076 vs 117 entries; Ceramide 36,288 vs 39,882 annotated features)
- Single-library class list reproduces the article's assignments: OxPC present only in MS-DIAL, CE present only in CalicoLipids
- Output CSV (e.g., dataset_s5_lipid_class_counts.csv) has one row per lipid-class/adduct combination with non-negative integer counts for each library
- Cross-tabulation totals equal the raw record counts in the input annotation table (no dropped or duplicated records during grouping)

## Limitations

- Lipid class coverage is bounded by the generating libraries: CalicoLipids enumerates a fixed class list (CE and ErgE among them, but no OxPC), so single-library flags reflect library design rather than measurement capability
- Enumerating all theoretical spectra over all lipid classes is time-consuming, motivating class subsetting via the -c option
- Counts are only meaningful for the specific thresholded search settings used in the MAVEN2 comparison; different thresholds change annotated-feature counts

## Evidence

- [other] Some lipid classes were represented in only one library: the oxidized phosphatidylcholines (OxPCs) were present only in MS-DIAL, while the cholesterol esters (CEs) were present only in CalicoLipids.: "we introduce a novel in-silico lipidomics library covering major lipid classes and compare searches using our novel library to searches with existing in-silico lipidomics libraries."
- [readme] Enumerating all theoretical spectra over all lipid classes may be time-consuming: [evidence span withheld — anchor status: external_doc]
- [readme] CalicoLipidLibrary generates in-silico lipid fragmentation spectra from standards and literature: [evidence span withheld — anchor status: external_doc]
