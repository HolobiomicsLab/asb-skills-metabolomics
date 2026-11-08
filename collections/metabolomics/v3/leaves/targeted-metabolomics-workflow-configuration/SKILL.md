---
name: targeted-metabolomics-workflow-configuration
description: Use when you need to quantitate targeted LC-MS/MS MRM data (e.g., vendor
  raw files for bile acids or phospholipids converted to mzML/mzXML) with automatic
  coelution handling, standard-curve absolute quantitation, and QC-based batch-effect
  correction.
when_to_use_negative:
- When the input is already a processed feature table or normalized abundance matrix
  (no peak detection or deconvolution is needed)
- When the tool produces error messages for not finding the specified transitions
  (as occurred with MRMPROBS) rather than switching to a different pipeline
- When unquantifiable coeluting or noisy peaks dominate the dataset and another tool
  (e.g., TargetLynx) reports the smallest average STD but at the cost of the highest
  number of unquantifiable compounds
tools:
- name: MRMQuant
  role: Primary quantitation engine performing peak detection, deconvolution, absolute
    quantitation, and batch-effect correction on MRM data
  repo: https://github.com/kslynn128171/MRMQuant
- name: ProteoWizard
  role: Used with msconvert to convert vendor raw data to mzML/mzXML for MRMQuant
    ingestion
- name: msconvert
  role: Concrete converter invoked to transform vendor raw files into the mzML format
    MRMQuant reads
- name: MRMkit
  role: Reference example dataset and companion pipeline used for comparative evaluation
    and reproduction of average-STD metrics
  repo: https://github.com/MRMkit/MRMkit
- name: MATLAB
  role: Runtime environment hosting MRMQuant; a binary MATLAB MAT-file is generated
    and loaded into MRMQuant
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:507a30adafa34fb9a648274ed20b4af8a3d19790824c9fa034153e449095d9cf
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
  - Microsoft Excel
  - ProteoWizard
  - msconvert
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# targeted-metabolomics-workflow-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure and execute the MRMQuant end-to-end targeted metabolomics quantitation pipeline, from vendor raw-data conversion through peak detection, deconvolution, absolute quantitation, and batch-effect correction to inspection-ready Excel output.

## When to use

Use when you need to quantitate targeted LC-MS/MS MRM data (e.g., vendor raw files for bile acids or phospholipids converted to mzML/mzXML) with automatic coelution handling, standard-curve absolute quantitation, and QC-based batch-effect correction.

## When NOT to use

- When the input is already a processed feature table or normalized abundance matrix (no peak detection or deconvolution is needed)
- When the tool produces error messages for not finding the specified transitions (as occurred with MRMPROBS) rather than switching to a different pipeline
- When unquantifiable coeluting or noisy peaks dominate the dataset and another tool (e.g., TargetLynx) reports the smallest average STD but at the cost of the highest number of unquantifiable compounds

## Inputs

- Vendor raw LC-MS/MS MRM data files converted to mzML or mzXML
- Method file describing targeted compounds (precursor/product m/z, RTs, transitions, internal standards, standard-curve points)
- Sample files including QC samples (e.g., five technical-replicate or batch-pooled QCs) when batch correction is needed
- Reference-sample signal definitions for correction runs

## Outputs

- Microsoft Excel (.xlsx) quantitation results file with normalized abundances per compound per sample
- Quantitated peak table for visual inspection and manual correction
- Batch-effect-corrected abundances when QC samples are included

## How to apply

First convert vendor raw files to mzML (or mzXML) using msconvert in ProteoWizard and prepare a method file with precursor/product m/z, retention times, and internal-standard/standard-curve information for the targeted compounds, as MRMQuant loads these together via a MATLAB MAT-file. Load the sample files and method file, then run peak detection per ion chromatogram (using the MATLAB findpeaks function to locate peak tips), quantify transitions to targeted compounds, and enable automatic deconvolution of coeluted peaks and dynamic signal smoothing to avoid overestimating compounds such as coeluting PC 34:3 isomers. If QC samples (technical replicates or batch-pooled QCs) are included, invoke the batch-effect correction step, which fits LOESS curves to QC abundances and normalizes; drop compounds with unreliable signals before computing replicate variability. Output is a Microsoft Excel (.xlsx) quantitation table of normalized abundances plus quantitated peaks for manual visual inspection and correction; on the eight-replicate phospholipid QC dataset (163 retained phospholipids) the pipeline should yield average STDs around 0.08 (0.0832 no deconvolution, 0.0828 automatic, 0.0813 with reference correction).

## Related tools

- **MRMQuant** (Primary quantitation engine performing peak detection, deconvolution, absolute quantitation, and batch-effect correction on MRM data) — https://github.com/kslynn128171/MRMQuant
- **ProteoWizard** (Used with msconvert to convert vendor raw data to mzML/mzXML for MRMQuant ingestion)
- **msconvert** (Concrete converter invoked to transform vendor raw files into the mzML format MRMQuant reads)
- **MRMkit** (Reference example dataset and companion pipeline used for comparative evaluation and reproduction of average-STD metrics) — https://github.com/MRMkit/MRMkit
- **MATLAB** (Runtime environment hosting MRMQuant; a binary MATLAB MAT-file is generated and loaded into MRMQuant)

## Evaluation signals

- On the eight technical-replicate phospholipid QC dataset (163 phospholipids retained), the average STD of normalized abundances should be around 0.08 — 0.0832 without deconvolution, 0.0828 with automatic deconvolution, 0.0813 with reference-sample correction
- Batch-effect correction on the 33-batch, 389-BQC MRMkit dataset should reduce the BQC regression slope from 0.0004 (R² = 0.3) to ~0.0 (R² = 1.44 × 10⁻⁴)
- Output must be a valid .xlsx file containing quantitation results for all targeted compounds and reproducible peak tables for manual inspection
- Peak detection and deconvolution should correctly separate coeluting isomers (e.g., three PC 34:3 isomers at RTs 4.72, 4.84, and 4.92 min) instead of overestimating a single targeted compound
- No compounds should be reported as unquantifiable for cases where MRMQuant succeeded where tools like TargetLynx failed

## Limitations

- Batch-effect correction processing time was around 5 h on the MRMkit dataset (3793 samples in 33 batches), though actual computation time is comparable to MRMkit's 4 h
- TargetLynx achieved a smaller average STD but with the highest number of unquantifiable compounds; comparing raw STD metrics across tools without counting unquantifiable compounds is misleading
- MRMPROBS could not produce quantitation results because error messages about not finding the specified transition prevented the program from running
- MRMkit and Skyline exhibited low (<0.1) average STD and RT differences but had problematic quantitation of coeluted peaks, noisy peaks, and peaks with unevenly sampled signals
- OpenMS was unable to consistently quantify the same peak when multiple peaks existed near the specified RT, leading to large variations in computed abundances

## Evidence

- [results] Average STD figures across MRMQuant modes: [evidence span withheld — anchor status: not_found]
