---
name: lipid-class-library-composition-analysis
description: Use when comparing lipid-class/adduct coverage across in-silico lipidomics
  libraries (MAVEN2, MS-DIAL, CalicoLipids) to find classes present in only one library.
when_to_use_negative:
- Input is a raw LC-MS/MS file needing peak picking or spectral matching rather than
  post-search library comparison
- Only one library is available — the skill requires cross-library comparison to identify
  single-library classes
- The question concerns metabolite (non-lipid) library coverage rather than lipid
  classes and adducts
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: MAVEN2
  role: Runs the LC-MS/MS library-comparison searches against the in-silico lipidomics
    libraries and exports the annotation table and library composition statistics
  repo: https://github.com/eugenemel/maven
- name: CalicoLipidLibrary
  role: Generates the in-silico lipid fragmentation spectra (MSP) for the CalicoLipids
    library, whose class coverage is compared against MS-DIAL
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:5c8b8673005bc76f1eac111a10136b627074bf9269b172cb1e08c5d5a90e6830
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - CalicoLipidLibrary
  - MAVEN2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# lipid-class-library-composition-analysis

## Summary

Enumerates and compares library entries and annotated features per lipid class and adduct across in-silico lipidomics libraries (MAVEN2's novel library, MS-DIAL, CalicoLipids) to flag lipid classes represented in only one library.

## When to use

Use when comparing lipid-class/adduct coverage across in-silico lipidomics libraries (MAVEN2, MS-DIAL, CalicoLipids) to find classes present in only one library.

## When NOT to use

- Input is a raw LC-MS/MS file needing peak picking or spectral matching rather than post-search library comparison
- Only one library is available — the skill requires cross-library comparison to identify single-library classes
- The question concerns metabolite (non-lipid) library coverage rather than lipid classes and adducts

## Inputs

- Thresholded lipid annotation table from MAVEN2 library-comparison searches
- Library composition statistics for the MAVEN2 novel library, MS-DIAL, and CalicoLipids
- In-silico lipid library MSP files (e.g., CalicoLipidLibrary generateDB.py output)

## Outputs

- Per-class, per-library count matrix of library entries and annotated features
- List of lipid classes represented in only one library (e.g., OxPC only in MS-DIAL, CE only in CalicoLipids)
- CSV file (dataset_s5_lipid_class_counts.csv)

## How to apply

Load the thresholded annotation table and library composition statistics exported from the MAVEN2 library-comparison searches. Group records by lipid class and adduct form, then count library entries and annotated features per class/adduct combination (e.g., SM spectra 14,076 vs 117 across libraries; Ceramide features 36,288 vs 39,882). Build a per-class, per-library count matrix by cross-comparing the MAVEN2 novel library, MS-DIAL, and CalicoLipids searches. Flag any class represented in only one library — in this analysis OxPCs were present only in MS-DIAL and cholesterol esters (CEs) only in CalicoLipids — and write the matrix plus the single-library class list to a CSV (dataset_s5_lipid_class_counts.csv). Validate the tabulated counts against Figure 4 / Dataset S5 of the article.

## Related tools

- **MAVEN2** (Runs the LC-MS/MS library-comparison searches against the in-silico lipidomics libraries and exports the annotation table and library composition statistics) — https://github.com/eugenemel/maven
- **CalicoLipidLibrary** (Generates the in-silico lipid fragmentation spectra (MSP) for the CalicoLipids library, whose class coverage is compared against MS-DIAL)

## Examples

```
python2.7 generateDB.py -m "pos" -c "PC,PE,PS" -o ./calicolipidlibrary_output -n "glycophospholipids_pos.msp"
```

## Evaluation signals

- Per-class/per-adduct counts in the output matrix match the values reported in Figure 4 / Dataset S5 of the article (e.g., SM 14,076 vs 117 spectra; Ceramide 36,288 vs 39,882 features)
- Single-library class list correctly assigns OxPC to MS-DIAL only and CE to CalicoLipids only
- Count matrix covers all lipid-class/adduct combinations present in the three libraries with no missing rows
- Output CSV (dataset_s5_lipid_class_counts.csv) contains both the count matrix and the single-library class list

## Limitations

- Enumerating all theoretical spectra over all lipid classes in CalicoLipidLibrary may be time-consuming; the -c option must be used to restrict to classes of interest
- Not all adduct forms are valid for all lipid classes, so per-class/adduct counting must handle absent combinations gracefully
- Coverage comparisons reflect the in-silico libraries' fixed class lists (CalicoLipidLibrary exposes a predefined set of ~63 classes), so classes outside those lists cannot be analyzed

## Evidence

- [abstract] abstract: "we introduce a novel in-silico lipidomics library covering major lipid classes and compare searches using our novel library to searches with existing in-silico lipidomics libraries."
- [readme] readme-library-generation: [evidence span withheld — anchor status: external_doc]
- [readme] readme-subsetting: "Enumerating all theoretical spectra over all lipid classes may be time-consuming, so the `-c` option may be used
to generate a subset of the library for a set of classes of interest."
- [readme] readme-adduct-validity: [evidence span withheld — anchor status: external_doc]
