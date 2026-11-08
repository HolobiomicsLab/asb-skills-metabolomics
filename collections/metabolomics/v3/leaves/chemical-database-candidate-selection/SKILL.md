---
name: chemical-database-candidate-selection
description: Use when running VInSMoC on run.npanalysis.org and you must pick which
  chemical database to search against, or associate an uploaded spectrum file with
  a project so it can serve as input to a database-search task.
when_to_use_negative:
- The uploaded spectrum file has not yet been parsed successfully (pending or error
  status) — do not associate or use it as input until parsing completes.
- The task uses a custom pre-processed local database workflow rather than the run.npanalysis.org
  project data table, since association of databases to projects is done through that
  interface.
- The goal is a standalone chemical-library build (e.g. in silico fragmentation outside
  VInSMoC) rather than selecting candidates for VInSMoC spectral search.
edam_operation: http://edamontology.org/operation_3754
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: VInSMoC
  role: Consumes the selected chemical database and spectrum file as 'Input Data'
    to score query mass spectra against candidate molecules in exact or variable search
    mode.
  repo: https://github.com/mohimanilab/VInSMoC
- name: npanalysis.org server (run.npanalysis.org)
  role: Hosts VInSMoC and provides the project/data/task tables through which chemical
    databases (e.g. NPAtlas) are associated to projects and selected for a run.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:397a1d2673234104950de022d127b32707ff9067693e0decbbad4ae0f76ccbd0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  - npanalysis.org server (run.npanalysis.org)
  edam_operation: http://edamontology.org/operation_3754
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# chemical-database-candidate-selection

## Summary

Select and associate an appropriate chemical database (e.g. NPAtlas, PubChem, COCONUT) as the candidate-molecule library for VInSMoC mass-spectral database search, and couple it with a parsed mass spectrum file (MGF, mzXML, or mzML) in a project on run.npanalysis.org.

## When to use

Use when running VInSMoC on run.npanalysis.org and you must pick which chemical database to search against, or associate an uploaded spectrum file with a project so it can serve as input to a database-search task.

## When NOT to use

- The uploaded spectrum file has not yet been parsed successfully (pending or error status) — do not associate or use it as input until parsing completes.
- The task uses a custom pre-processed local database workflow rather than the run.npanalysis.org project data table, since association of databases to projects is done through that interface.
- The goal is a standalone chemical-library build (e.g. in silico fragmentation outside VInSMoC) rather than selecting candidates for VInSMoC spectral search.

## Inputs

- Chemical database available to the project (e.g. NPAtlas, PubChem, COCONUT)
- Parsed mass spectrum file (MGF, mzXML, or mzML) such as extract-3_Metal-mix_1mM.mzML from MSV000086287
- Project page on run.npanalysis.org with data and tasks tables

## Outputs

- Project with both a mass spectrum file and a chemical database associated in its data table, ready as 'Input Data' for a VInSMoC task
- VInSMoC task whose best hits link to entries in the selected chemical database

## How to apply

Create a project on run.npanalysis.org via the plus button, then open its data table: public chemical databases such as NPAtlas are automatically available to every project, while your own spectrum file must be imported with the plus button in the data table. In the VInSMoC task form, under the 'Input Data' section, select both the mass spectrum file (MGF, mzXML, or mzML, e.g. extract-3_Metal-mix_1mM.mzML from MSV000086287) and the chemical database via their dropdowns. The database choice determines the candidate molecules VInSMoC scores query spectra against, so match the database to the expected chemistry (e.g. NPAtlas for natural products); results links back to each molecule's entry in the original chemical database it came from. If the desired database is absent, upload it as a new data point and wait for successful parsing before associating it to the project.

## Related tools

- **VInSMoC** (Consumes the selected chemical database and spectrum file as 'Input Data' to score query mass spectra against candidate molecules in exact or variable search mode.) — https://github.com/mohimanilab/VInSMoC
- **npanalysis.org server (run.npanalysis.org)** (Hosts VInSMoC and provides the project/data/task tables through which chemical databases (e.g. NPAtlas) are associated to projects and selected for a run.)

## Evaluation signals

- The project page's data table lists both the spectrum file (e.g. extract-3_Metal-mix_1mM.mzML) and the chosen database (e.g. NPAtlas) after association.
- The VInSMoC task form's 'Input Data' dropdowns show the intended spectrum file and chemical database selected before submission.
- Every molecule in the finished task's results is a link to its entry in the original chemical database that it came from.
- The chosen database matches the search chemistry — e.g. NPAtlas for natural-product variants such as the 85,000 previously unreported variants VInSMoC revealed.
- Public databases like NPAtlas are already accessible to the project; only user-uploaded spectra require explicit association via the plus button in the data table.

## Limitations

- Only databases that are public (automatically available to every project) or explicitly uploaded and associated via the data table can be used; arbitrary local libraries are not visible to the task.
- Data points must complete parsing before use, and malformed inputs show an error status requiring you to verify the declared extension matches the actual format.
- The search scope is limited to molecules in the selected database — e.g. the large-scale benchmark used 87 million molecules from PubChem and COCONUT, not all known chemistry.

## Evidence

- [intro] VInSMoC is a mass spectral search tool that scores query mass spectra against a database of candidate molecules.: "VInSMoC is a mass spectral search tool that scores query mass spectra against a database of candidate molecules."
- [readme] All public data provided by the developers is automatically available to every project.: "All public data provided by the developers is automatically available to every project.
We can see that we have access to the NPAtlas chemical database already."
- [readme] First, we need to select the mass spectrum file and chemical database we want to use for this run.: "First, we need to select the mass spectrum file and chemical database we want to use for this run.
We can do so using the dropdowns under the "Input Data" section"
- [readme] Each molecule is a link to its entry in the original chemical database that it came from.: "Each molecule is a link to its entry in the original chemical database that it came from."
- [readme] Each data input is parsed into a representation that is convenient for our tools to use downstream, so the data point is not immediately ready for use.: "Each data input is parsed into a representation that is convenient for our tools to use downstream, so the data point is not immediately ready for use."
