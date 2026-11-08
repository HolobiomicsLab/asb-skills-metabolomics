---
name: large-scale-metabolomics-data-ingestion
description: Use when raw high-throughput LC-MS/MS MRM data from a large targeted
  metabolomics study (e.g. thousands of samples in dozens of batches, with BQCs) must
  be loaded and prepared for automated quantitation.
when_to_use_negative:
- The input is already a quantitated feature/abundance table rather than raw or mzML/mzXML
  MRM chromatogram data
- The data are untargeted (DDA) or otherwise not MRM ion chromatograms with a targeted-compounds
  method file
- Only a tiny single-batch run with no QC samples is involved, so batch-effect correction
  and large-scale ingestion machinery add no value
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MRMQuant
  role: Automated MRM quantitation tool that ingests MRM sample files and a method
    file, generates a binary MATLAB MAT-file, and writes xlsx quantitation results
  repo: https://github.com/kslynn128171/MRMQuant
- name: ProteoWizard
  role: Source of msconvert, used to convert raw vendor data into the mzML/mzXML formats
    MRMQuant accepts
- name: msconvert
  role: ProteoWizard conversion utility applied to raw vendor files before ingestion
- name: MRMkit
  role: Provides the large example dataset (3793 samples, 33 batches, 295 compounds,
    389 BQCs) and an alternative processing tool for ingestion benchmarking
  repo: https://github.com/MRMkit/MRMkit
- name: MATLAB
  role: Binary MAT-file format generated during ingestion and loaded into MRMQuant;
    findpeaks is a MATLAB built-in used downstream
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:7433d3c9702fa3df09f0ea5c4e4db98239ccc1702ed029fec094361d15de0c89
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/MRMQuant
  repo_url: https://github.com/kslynn128171/MRMQuant
  tools:
  - MATLAB
  - MRMQuant
  - MRMkit
  - ProteoWizard
  - msconvert
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# large-scale-metabolomics-data-ingestion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ingests large-scale targeted metabolomics MRM datasets (thousands of samples across many batches) into MRMQuant by converting vendor raw files to mzML/mzXML with msconvert and loading them with a targeted-compounds method file, producing loadable MATLAB MAT data and xlsx quantitation outputs.

## When to use

Use when raw high-throughput LC-MS/MS MRM data from a large targeted metabolomics study (e.g. thousands of samples in dozens of batches, with BQCs) must be loaded and prepared for automated quantitation.

## When NOT to use

- The input is already a quantitated feature/abundance table rather than raw or mzML/mzXML MRM chromatogram data
- The data are untargeted (DDA) or otherwise not MRM ion chromatograms with a targeted-compounds method file
- Only a tiny single-batch run with no QC samples is involved, so batch-effect correction and large-scale ingestion machinery add no value

## Inputs

- Raw vendor MRM LC-MS/MS sample files (converted to mzML or mzXML via ProteoWizard msconvert)
- Method file with information about the targeted compounds
- Large-scale batched dataset such as the MRMkit dataset (3793 samples in 33 batches, 295 compounds, 389 BQCs) from github.com/MRMkit/MRMkit
- Example samples and method files from github.com/kslynn128171/MRMQuant

## Outputs

- Binary MATLAB MAT-file loadable into MRMQuant
- Quantitation results in Microsoft Excel xlsx format
- Batch-effect-corrected abundances when QC samples (e.g. BQCs) are present

## How to apply

Convert vendor raw data to mzML or mzXML using msconvert in ProteoWizard, since MRMQuant accepts both formats. Load the MRM sample files together with a method file containing the information necessary about the targeted compounds; optionally install get_PrecMZ_ProdMZ from MRMQuant's 'program/associated programs' folder to acquire precursor and product m/z values from the MRM sample file. MRMQuant then generates a binary MATLAB MAT-file that is loaded into the tool for peak detection and quantitation, and saves results in Microsoft Excel xlsx format. For very large sets such as the MRMkit dataset (3793 samples in 33 batches, 295 compounds, 389 BQCs), run batch-effect correction when QC samples are included and budget wall-clock time on the order of ~5 h (comparable to MRMKit's ~4 h in actual computation). Always use the latest MRMQuant release (currently v2.7) per the README.

## Related tools

- **MRMQuant** (Automated MRM quantitation tool that ingests MRM sample files and a method file, generates a binary MATLAB MAT-file, and writes xlsx quantitation results) — https://github.com/kslynn128171/MRMQuant
- **ProteoWizard** (Source of msconvert, used to convert raw vendor data into the mzML/mzXML formats MRMQuant accepts)
- **msconvert** (ProteoWizard conversion utility applied to raw vendor files before ingestion)
- **MRMkit** (Provides the large example dataset (3793 samples, 33 batches, 295 compounds, 389 BQCs) and an alternative processing tool for ingestion benchmarking) — https://github.com/MRMkit/MRMkit
- **MATLAB** (Binary MAT-file format generated during ingestion and loaded into MRMQuant; findpeaks is a MATLAB built-in used downstream)

## Evaluation signals

- All vendor raw files convert cleanly to mzML or mzXML with msconvert before loading
- A binary MATLAB MAT-file is generated and can be loaded into MRMQuant without error
- Quantitation results are emitted as a Microsoft Excel file in xlsx format
- Batch structure is preserved for large sets (e.g. 3793 samples in 33 batches with 389 BQCs for the MRMkit dataset)
- Wall-clock processing for a dataset of this scale is on the order of hours (~5 h for MRMQuant vs ~4 h for MRMKit, with comparable actual computation time)

## Limitations

- MRMkit repository is archived; all development moved to a new repository, so its example dataset location may change
- Processing time on very large datasets (~3793 samples) is around 5 h, which may seem longer than MRMKit's 4 h even though actual computation time is comparable
- Ingestion requires a method file with the information necessary about the targeted compounds; without it, peaks corresponding to targeted compounds cannot be matched

## Evidence
