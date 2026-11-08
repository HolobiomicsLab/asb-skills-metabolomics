---
name: processing-report-interpreting-per-filter-spectrum-changes
description: 'Use when auditing or reproducing a matchms SpectrumProcessor/Pipeline
  cleaning run (e.g. on GNPS, MassBank, or MoNA libraries): you need to verify which
  filters ran in the fixed global order and how many spectra each filter removed or
  modified.'
when_to_use_negative:
- 'You need to judge chemical correctness of annotations: the report only counts changes
  — wrong annotations consistent with the measured mass go unnoticed (no fragment-matching
  filters are implemented).'
- Your processing was not run through the matchms SpectrumProcessor/Pipeline machinery,
  so no auto-generated report or YAML file exists to interpret.
- You need standardized instrument-type or collision-energy metadata quality metrics
  — current matchms filters do not yet clean or report on these fields.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Generates the processing report and YAML workflow file automatically at the
    end of every SpectrumProcessor/Pipeline cleaning run (version 0.26.4 used in the
    paper)
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: Loads and compares SMILES, InChI and InChIKey so annotation-validation filter
    counts in the report (Require Valid Annotation) can be interpreted
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:cdcd063a8c6c91fddf748ec39b660a6b359b9138be1ee121b87eee4006e9fd4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - RDKit
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# processing-report-interpreting-per-filter-spectrum-changes

## Summary

Interprets the automatically generated matchms processing report, which tabulates per-filter counts of spectra removed, metadata fields changed, and peaks changed during a library-cleaning run, alongside the auto-created YAML workflow file that fixes all settings for reproducibility.

## When to use

Use when auditing or reproducing a matchms SpectrumProcessor/Pipeline cleaning run (e.g. on GNPS, MassBank, or MoNA libraries): you need to verify which filters ran in the fixed global order and how many spectra each filter removed or modified.

## When NOT to use

- You need to judge chemical correctness of annotations: the report only counts changes — wrong annotations consistent with the measured mass go unnoticed (no fragment-matching filters are implemented).
- Your processing was not run through the matchms SpectrumProcessor/Pipeline machinery, so no auto-generated report or YAML file exists to interpret.
- You need standardized instrument-type or collision-energy metadata quality metrics — current matchms filters do not yet clean or report on these fields.

## Inputs

- matchms processing report (per-filter counts of removed spectra, changed metadata, changed peaks)
- auto-generated YAML workflow file with all filter settings and order
- cleaned spectral library file (e.g. GNPS, MassBank, MoNA library output)
- supplementary report tables (Tables S1–S5) as reference counts for comparison

## Outputs

- Verified audit of filter execution order and per-filter spectrum-change counts
- Reproducibility assessment of the cleaning run against the YAML-captured configuration
- Estimated annotation error rates (e.g. SMILES derived from compound names)

## How to apply

After running the matchms 0.26.4 library-cleaning pipeline (basic, default, or library-cleaning filter tiers) on a spectral library, read the automatically generated processing report rather than the raw outputs: it lists every filter in the fixed global order with counts of spectra removed, metadata fields changed, and peaks changed. Cross-check these per-filter counts against the supplementary report tables (Tables S1–S5) for the GNPS reference run of 500,569 input spectra, or against the MassBank and MoNA run reports when transferring the YAML-captured configuration to other libraries. Confirm that the companion YAML file captures the identical filter order and settings so the cleaning run can be replicated; the report plus YAML together are the reproducibility record. Use the counts to judge filter impact — e.g. how many spectra lacked complete SMILES/InChI/InChIKey annotations checked via RDKit, or how many had adduct/parent-mass repairs (molar-mass vs monoisotopic-mass errors) — and to estimate error rates, as demonstrated by removing SMILES annotations and re-deriving them from compound names via PubChem. Validation passes only if each run produced both a cleaned library file and a processing report whose filter list matches the configured global order.

## Related tools

- **matchms** (Generates the processing report and YAML workflow file automatically at the end of every SpectrumProcessor/Pipeline cleaning run (version 0.26.4 used in the paper)) — https://github.com/matchms/matchms
- **RDKit** (Loads and compares SMILES, InChI and InChIKey so annotation-validation filter counts in the report (Require Valid Annotation) can be interpreted)

## Evaluation signals

- Each cleaning run produced both a cleaned library file and a processing report, never one without the other.
- The report enumerates the filters applied in the fixed global order, matching the filter set in the YAML workflow file.
- Per-filter spectrum-change counts are comparable to the supplementary report tables (Tables S3–S5 for MassBank/MoNA; Tables S1–S2 for GNPS examples).
- Counts partition correctly: spectra removed by filters plus spectra surviving equal the input library size (e.g. 500,569 GNPS spectra before cleaning).
- The YAML file reproduces the identical run when re-executed, confirming the report reflects the same settings.

## Limitations

- The report quantifies changes, not correctness: plausibility checks considering both metadata and measured fragments are lacking, so wrong chemical annotations consistent with the measured mass go unnoticed.
- Fragment-matching filters (checking if fragments match the given annotation) are only a possible future expansion, not implemented.
- Instrument type and collision-energy fields are not yet cleaned or standardized, so the report cannot assess metadata quality in those fields.
- The processing-report examples are only referenced as supplementary tables (Table S1 and Table S2), and Supplementary file 1 appears without content in the document.

## Evidence

- [methods] a processing report is generated. This report gives an overview of the number of spectra that were changed in the library by each filter.: "a processing report is generated. This report gives an overview of the number of spectra that were changed in the library by each filter."
- [methods] it automatically creates a YAML file containing all settings and filters used.: "it automatically creates a YAML file containing all settings and filters used."
- [results] Running the machms pipeline with the filters given in Supplementary Table S1 on the GNPS library of 500,569 spectra took 6 h and 45 min.: "Running the machms pipeline with the filters given in Supplementary Table S1 on the GNPS library of 500,569 spectra took 6 h and 45 min."
- [discussion] Examples of such processing reports can be found in Table S1 and Table S2.: "Examples of such processing reports can be found in Table S1 and Table S2."
- [readme] This also allows to create, import, export, or modify workflows using yaml files.: [evidence span withheld — anchor status: external_doc]
