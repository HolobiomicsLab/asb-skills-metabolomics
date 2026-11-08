---
name: metadata-versus-peak-change-differentiation
description: Use when running a matchms SpectrumProcessor library-cleaning pipeline
  (e.g. on the GNPS public library) and you need a per-filter breakdown of how many
  spectra were deleted, had metadata changed, or had peak information changed.
when_to_use_negative:
- When you only need a cleaned library without auditing which filter changed what
  — the three-class breakdown is a reporting artifact, not a cleaning step itself.
- 'When spectra carry no annotations at all (e.g. unannotated MGF peaks lists): the
  metadata-change column relies on annotation fields (SMILES, InChI, InChIKey, adduct,
  precursor m/z) that may be absent.'
- When you need plausibility checks that consider both metadata and measured fragments
  — the article notes wrong chemical annotations consistent with the measured mass
  go unnoticed, and fragment-matching filters are only a possible future expansion.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Emits the processing report during the library-cleaning run, differentiating
    completely removed spectra, metadata changes, and peak-information changes per
    filter
  repo: https://github.com/matchms/matchms
- name: matchms 0.26.4
  role: Specific version used to run the GNPS library cleaning pipelines that produce
    the report
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: Loads and compares SMILES, InChI and InChIKey so Require Valid Annotation
    can flag spectra for the 'removed' column of the report
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:12052c791162e9ae1e36c70d7eceadc072a3b477aea21ec90e27ed4b66d99165
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - RDKit
  - matchms
  - matchms 0.26.4
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Metadata versus peak-change differentiation

## Summary

Interpret matchms library-cleaning processing reports by classifying each filter's effect on spectra into three distinct change classes: complete spectrum removal, metadata-only changes, and peak-information changes. This differentiation is the reporting backbone of the reproducible MS/MS library cleaning pipeline.

## When to use

Use when running a matchms SpectrumProcessor library-cleaning pipeline (e.g. on the GNPS public library) and you need a per-filter breakdown of how many spectra were deleted, had metadata changed, or had peak information changed.

## When NOT to use

- When you only need a cleaned library without auditing which filter changed what — the three-class breakdown is a reporting artifact, not a cleaning step itself.
- When spectra carry no annotations at all (e.g. unannotated MGF peaks lists): the metadata-change column relies on annotation fields (SMILES, InChI, InChIKey, adduct, precursor m/z) that may be absent.
- When you need plausibility checks that consider both metadata and measured fragments — the article notes wrong chemical annotations consistent with the measured mass go unnoticed, and fragment-matching filters are only a possible future expansion.

## Inputs

- Mass spectral library file (e.g. GNPS public library .mgp/.msp downloaded 21-08-2023, or MoNA / MassBank libraries)
- Configured matchms SpectrumProcessor pipeline (library-cleaning filter set with matchms version 0.26.4)
- Automatically generated YAML workflow file (workflow.yaml) recording all settings and filters used

## Outputs

- Processing report (e.g. processing_report.csv) giving, per filter, the number of spectra completely removed, the number with changed metadata, and the number with changed peak information
- Automatically created YAML file containing all settings and filters used, directly reusable to rerun the cleaning pipeline

## How to apply

Build the SpectrumProcessor pipeline with the library cleaning filter set (basic/default/library-cleaning tiers: metadata harmonization, derivation of missing metadata, annotation repair such as Repair Smiles Of Salts, and Require Valid Annotation). Run it over the input library, e.g. the GNPS public mass spectral library downloaded 21-08-2023 (500,569 spectra; run with matchms 0.26.4). During the run, matchms automatically generates a processing report that gives an overview of the number of spectra changed by each filter, differentiating between completely removing spectra, changing metadata, and changing peak information. Read the report column-wise per filter step: a filter counts as 'removing' if spectra were discarded entirely (e.g. Require Valid Annotation removing spectra whose SMILES/InChI/InChIKey do not match), as 'metadata change' if only fields like adduct, precursor m/z or annotations were corrected, and as 'peak change' if peak filtering or intensity normalization altered the peaks. Cross-check the report against the YAML file matchms also creates, which records all settings and filters used and can rerun the pipeline unchanged.

## Related tools

- **matchms** (Emits the processing report during the library-cleaning run, differentiating completely removed spectra, metadata changes, and peak-information changes per filter) — https://github.com/matchms/matchms
- **matchms 0.26.4** (Specific version used to run the GNPS library cleaning pipelines that produce the report) — https://github.com/matchms/matchms
- **RDKit** (Loads and compares SMILES, InChI and InChIKey so Require Valid Annotation can flag spectra for the 'removed' column of the report)

## Examples

```
from matchms import Pipeline; pipeline = Pipeline.from_yaml('workflow.yaml'); report = pipeline.run('gnps_library_20230821.msp'); print(report)
```

## Evaluation signals

- The report must contain three change classes per filter (spectrum removal, metadata change, peak change) — absence of any class column means the report was not produced by a library-cleaning-tier run.
- Spectrum counts must reconcile: for each filter, removed + metadata-changed + peak-changed + untouched spectra should equal the input library size (e.g. 500,569 for the GNPS library before cleaning; 448,485 curated spectra available on Zenodo after cleaning).
- Running the pipeline again from the automatically created YAML file should reproduce an identical processing report.
- Require Valid Annotation should dominate the 'removed' class; Repair Adduct And Parent Mass Based On Smiles and Derive Annotation From Compound Name should populate the 'metadata change' class rather than 'removed'.
- Filter counts must be consistent with the installed matchms version (0.26.4 for the published GNPS run).

## Limitations

- Current filters lack plausibility checks that consider both metadata and measured fragments, so wrong chemical annotations consistent with the measured mass go unnoticed in the report.
- Fragment-matching filters (checking if fragments match the given annotation) are only a possible future expansion, not implemented, so the report cannot flag peak/annotation inconsistencies.
- Metadata fields such as instrument type and collision energies are not yet cleaned/standardized by current matchms filters and thus cannot be tracked in the metadata-change class.
- Examples of processing reports are only referenced as supplementary tables (Table S1 and Table S2), and the published report examples may not be fully embedded in the article text.

## Evidence

- [methods] Processing report classifies three change types: "This report gives an overview of the number of spectra that were changed in the library by each filter."
- [methods] Three-class differentiation is explicit: [evidence span withheld — anchor status: not_found]
- [methods] YAML co-generated for reproducibility: "it automatically creates a YAML file containing all settings and filters used."
- [methods] Concrete library and version for the reported numbers: "Matchms version 0.26.4 was used to run these pipelines."
- [readme] Verification by rerunning from YAML: [evidence span withheld — anchor status: external_doc]
- [results] Scale of the cleaned library confirming the report totals: "The final cleaned GNPS public mass spectral library contains 448.485 curated mass spectra and is available on Zenodo"
