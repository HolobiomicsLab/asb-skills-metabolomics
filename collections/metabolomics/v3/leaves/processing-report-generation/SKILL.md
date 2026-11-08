---
name: processing-report-generation
description: Use when a matchms library-cleaning pipeline has been run (e.g. on the
  GNPS public library of 500,569 spectra) and you need a per-filter accounting of
  removals/repairs to quantify repair-function impact or audit the cleaning run.
when_to_use_negative:
- You need to detect wrong chemical annotations that are consistent with the measured
  mass — the report only counts filter effects and the pipeline lacks plausibility
  checks that consider both metadata and measured fragments
- You need standardized instrument type or collision-energy metadata — these fields
  are not yet cleaned or reported by current matchms filters
- You need curation during library upload rather than a post-hoc batch cleaning report
  — a fast validator tool for upload-time curation is only proposed as future work
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Runs the library-cleaning pipeline (v0.26.4) and automatically generates the
    processing report and reproducibility YAML file
  repo: github.com/matchms/matchms
- name: RDKit
  role: Loads and compares SMILES, InChI and InChIKey annotations, so require/repair
    filter counts in the report reflect annotation validity checks
provenance:
  source_task_ids:
  - task_004
  - task_001
  - task_007
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:560e2d6beb26cec44ed1b3e495a9e7180d966faedb1c90ddbffb312cc817145d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# processing-report-generation

## Summary

Auto-generation of a processing report that tallies, per matchms library-cleaning filter, how many spectra were removed, had metadata changed, or had peaks changed, alongside a YAML record of all settings and filters used for reproducibility.

## When to use

Use when a matchms library-cleaning pipeline has been run (e.g. on the GNPS public library of 500,569 spectra) and you need a per-filter accounting of removals/repairs to quantify repair-function impact or audit the cleaning run.

## When NOT to use

- You need to detect wrong chemical annotations that are consistent with the measured mass — the report only counts filter effects and the pipeline lacks plausibility checks that consider both metadata and measured fragments
- You need standardized instrument type or collision-energy metadata — these fields are not yet cleaned or reported by current matchms filters
- You need curation during library upload rather than a post-hoc batch cleaning report — a fast validator tool for upload-time curation is only proposed as future work

## Inputs

- Raw GNPS public mass spectral library snapshot (500,569 spectra, downloaded 21-08-2023; readable formats include msp, MGF, mzML, mzXML, JSON)
- matchms pipeline/filter configuration (create_workflow default filters or custom SpectrumProcessor filter list)
- Reference processing report from a prior run (e.g. the with-repair run removing 31,758 spectra) for comparison

## Outputs

- Processing report listing per filter the number of spectra removed, metadata changed, and peaks changed (e.g. cond_gnps_no_repair_report.txt)
- YAML file containing all settings and filters used, enabling exact rerun of the pipeline
- Total removed-spectra count for cross-run comparison (31,758 with repair vs 83,843 without)

## How to apply

Run the library-cleaning pipeline in matchms 0.26.4 (default filters plus repair filters such as Repair Smiles Of Salts, Repair Parent Mass Is Molar Mass, Repair Not Matching Annotation); matchms automatically generates the processing report, which counts per filter how many spectra were removed, had metadata changed, or had peaks changed. It simultaneously writes a YAML file containing all settings and filters used so the exact run can be rerun. To quantify the value of the repair functions, execute the pipeline twice — once with all repair filters and once with a custom SpectrumProcessor excluding them — and compare total spectra removed: on the GNPS library the with-repair run removed 31,758 spectra while the no-repair run removed 83,843, so the no-repair count must exceed 31,758. Save the report (e.g. as cond_gnps_no_repair_report.txt) and check the mass balance 500,569 − 31,758 = 448,485 curated spectra deposited on Zenodo.

## Related tools

- **matchms** (Runs the library-cleaning pipeline (v0.26.4) and automatically generates the processing report and reproducibility YAML file) — github.com/matchms/matchms
- **RDKit** (Loads and compares SMILES, InChI and InChIKey annotations, so require/repair filter counts in the report reflect annotation validity checks)

## Examples

```
from matchms import Pipeline; pipeline = Pipeline(yaml_file="library_cleaning_pipeline.yaml"); pipeline.run()
```

## Evaluation signals

- The spectra-removed count in the no-repair report (cond_gnps_no_repair_report.txt) is strictly greater than 31,758 — the reference with-repair removal count
- The with-repair run on the 500,569-spectrum GNPS library removes exactly 31,758 spectra, leaving 448,485 curated spectra
- The report partitions per-filter effects into removed / metadata-changed / peaks-changed counts with no missing or double-counted spectra relative to the 500,569 inputs
- A YAML file containing all settings and filters used is emitted and can be rerun to reproduce the same report
- Per-filter changed counts are non-negative and consistent with the paper's processing-report tables (Supplementary Tables S1/S2)

## Limitations

- Wrong chemical annotations consistent with the measured mass go unnoticed, since no plausibility checks combine metadata and measured fragments
- Report examples are only referenced as supplementary tables (Table S1 and S2); Supplementary file 1 appears without content in the document
- The full GNPS run takes ~6 h 45 min, so report generation is tied to a long batch processing step rather than an interactive check

## Evidence

- [methods] a processing report is generated. This report gives an overview of the number of spectra that were changed in the library by each filter.: "a processing report is generated. This report gives an overview of the number of spectra that were changed in the library by each filter."
- [methods] it automatically creates a YAML file containing all settings and filters used.: "it automatically creates a YAML file containing all settings and filters used."
- [results] Running the machms pipeline with the filters given in Supplementary Table S1 on the GNPS library of 500,569 spectra took 6 h and 45 min.: "Running the machms pipeline with the filters given in Supplementary Table S1 on the GNPS library of 500,569 spectra took 6 h and 45 min."
- [readme] This also allows to create, import, export, or modify workflows using yaml files.: [evidence span withheld — anchor status: external_doc]
