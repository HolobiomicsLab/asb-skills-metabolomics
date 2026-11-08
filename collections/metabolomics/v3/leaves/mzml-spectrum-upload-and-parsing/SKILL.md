---
name: mzml-spectrum-upload-and-parsing
description: Use when raw tandem mass spectra are stored in mzML, mzXML, or MGF files
  and must be ingested into the npanalysis.org platform as parsed 'Mass Spectrum'
  data before running VInSMoC search tasks.
when_to_use_negative:
- The spectrum has already been uploaded and successfully parsed on the server (skip
  re-upload and associate it to the project instead).
- The input is a chemical database rather than a mass spectrum file — databases like
  NPAtlas are instead made available through the project data table.
- The file extension does not match the actual data format (e.g. an mzXML payload
  declared as .mzML); the parse will fail, and the format declaration must be corrected
  first.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: VInSMoC
  role: Downstream consumer of the parsed Mass Spectrum data; VInSMoC is the tool
    this ingestion step prepares inputs for
  repo: https://github.com/mohimanilab/VInSMoC
- name: npanalysis.org server (run.npanalysis.org)
  role: Hosts the upload form, performs the parsing of MGF/mzXML/mzML inputs into
    a server-internal representation, and reports parse status
  repo: https://run.npanalysis.org
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:a25fd3b2c4019fce09833e97458a5671c22f1347643946ca14bab329f22259ea
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  - npanalysis.org server (run.npanalysis.org)
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# mzml-spectrum-upload-and-parsing

## Summary

Upload an mzML (or MGF/mzXML) mass spectrum file to the run.npanalysis.org server, select 'Mass Spectrum' as the data kind, and wait for the server to parse it into a representation usable by downstream tools such as VInSMoC. This is the prerequisite data-ingestion step before any VInSMoC database search task can be launched.

## When to use

Use when raw tandem mass spectra are stored in mzML, mzXML, or MGF files and must be ingested into the npanalysis.org platform as parsed 'Mass Spectrum' data before running VInSMoC search tasks.

## When NOT to use

- The spectrum has already been uploaded and successfully parsed on the server (skip re-upload and associate it to the project instead).
- The input is a chemical database rather than a mass spectrum file — databases like NPAtlas are instead made available through the project data table.
- The file extension does not match the actual data format (e.g. an mzXML payload declared as .mzML); the parse will fail, and the format declaration must be corrected first.

## Inputs

- Mass spectral input file in MGF, mzXML, or mzML format (e.g. extract-3_Metal-mix_1mM.mzML from MSV000086287)
- npanalysis.org account with access to the run.npanalysis.org dashboard

## Outputs

- A parsed 'Mass Spectrum' data point on run.npanalysis.org with successful parse status, ready to be associated with a project and used as VInSMoC input

## How to apply

Create an account at run.npanalysis.org/#/signup if needed, then from the Data tab click the plus button to open the upload form. Choose 'Mass Spectrum' as the data kind and select an mzML, mzXML, or MGF file from your computer (e.g. extract-3_Metal-mix_1mM.mzML from MSV000086287), then press submit. The data point is not immediately ready: the server parses each input into a representation convenient for downstream tools, so wait until the status shows successfully parsed — a few minutes in the article's example. If the status instead shows an error, double-check that the input format matches the declared file extension. Once parsed, associate the data point to a project via the plus button in the project's data table so it becomes selectable in a VInSMoC task's 'Input Data' section.

## Related tools

- **VInSMoC** (Downstream consumer of the parsed Mass Spectrum data; VInSMoC is the tool this ingestion step prepares inputs for) — https://github.com/mohimanilab/VInSMoC
- **npanalysis.org server (run.npanalysis.org)** (Hosts the upload form, performs the parsing of MGF/mzXML/mzML inputs into a server-internal representation, and reports parse status) — https://run.npanalysis.org

## Examples

```
# Via the run.npanalysis.org dashboard: Data tab → plus button → data kind "Mass Spectrum" → select extract-3_Metal-mix_1mM.mzML → submit → wait for parse status: success
```

## Evaluation signals

- The data point's status on the npanalysis.org dashboard transitions from pending to successfully parsed (not error).
- The file's declared extension matches its actual format (MGF, mzXML, or mzML) — the README states a mismatch causes an error status.
- The parsed data point appears in the project's data table after association via the plus button.
- The data point is selectable in the 'Input Data' section of a new VInSMoC task form.
- The underlying file is one of the supported types: MGF, mzXML, or mzML.

## Limitations

- Parsing is asynchronous: the data point is not immediately ready for use, so downstream tasks cannot be launched until parsing completes.
- Malformed inputs end in an error status; the only remediation described is to double-check that the input format matches the declared extension.
- Only MGF, mzXML, and mzML mass spectral formats are supported for upload.
- Uploaded data must be explicitly associated to each project by the uploading user; public data provided by the developers (like the NPAtlas database) is automatically available, but user uploads are not.

## Evidence
