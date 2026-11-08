---
name: lcms-lipidomics-data-reprocessing
description: Use when reprocessing publicly deposited LC-MS lipidomics raw files (Thermo
  .raw, e.g. MSV000091033 Nile rat plasma) to reproduce or re-annotate reported lipid
  feature counts and identifications.
when_to_use_negative:
- Input is already a filtered feature/quantification table ready for statistical analysis
  (e.g. Metaboanalyst input) rather than raw LC-MS/MS files requiring peak extraction
  and spectral matching
- MS/MS data are not from Thermo instruments or Compound Discoverer 3.3 is unavailable,
  since Peak Finder requires .cdResult files as input for the peak table
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: LipiDex 2
  role: Stand-alone pipeline (Spectrum Searcher, Peak Finder, Degreaser, Lipid Spectral
    Annotator) for spectral matching, peak annotation, and QC filtering of lipidomics
    data
  repo: https://github.com/coongroup/LipiDex-2
- name: Compound Discoverer
  role: Peak extraction software (version 3.3) producing .cdResult peak tables consumed
    directly by LipiDex 2 Peak Finder
  repo: https://www.thermofisher.com/us/en/home/industrial/mass-spectrometry/liquid-chromatography-mass-spectrometry-lc-ms/lc-ms-software/multi-omics-data-analysis/compound-discoverer-software.html
- name: LipiDex 2 Library Generator
  role: Generates the in silico lipid spectral library (with MSn tree-based fragmentation)
    used by Spectrum Searcher for identification
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:950a3cb4fad7f9554afde4d77a01d7e824ded4f6e2014d50d6327a38b57d971a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coongroup/LipiDex-2
  tools:
  - Compound Discoverer
  - LipiDex 2
  - LipiDex 2 Library Generator
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# lcms-lipidomics-data-reprocessing

## Summary

Reprocess public LC-MS/MS lipidomics datasets (e.g. Thermo .raw files from MassIVE study MSV000091033) through the LipiDex 2 pipeline — Compound Discoverer 3.3 peak extraction, Spectrum Searcher spectral matching, and Peak Finder filtering — to reproduce reported feature and identification counts.

## When to use

Use when reprocessing publicly deposited LC-MS lipidomics raw files (Thermo .raw, e.g. MSV000091033 Nile rat plasma) to reproduce or re-annotate reported lipid feature counts and identifications.

## When NOT to use

- Input is already a filtered feature/quantification table ready for statistical analysis (e.g. Metaboanalyst input) rather than raw LC-MS/MS files requiring peak extraction and spectral matching
- MS/MS data are not from Thermo instruments or Compound Discoverer 3.3 is unavailable, since Peak Finder requires .cdResult files as input for the peak table

## Inputs

- Thermo .raw LC-MS/MS files from a public MassIVE dataset (e.g. MSV000091033, 40 Nile rat plasma samples)
- In silico lipid spectral library generated with LipiDex 2 Library Generator
- Compound Discoverer 3.3 .cdResult peak table (input to Peak Finder)

## Outputs

- LipiDex2_PeakFinder_Results.csv with unidentified and identified feature counts (e.g. 14826/779 pre-filter, 13898/719 post-filter)
- Annotated chromatographic peak table with lipid identifications filtered on 7 criteria
- .csv results files passed between stand-alone LipiDex 2 modules

## How to apply

Download the raw dataset (e.g. 40 Nile rat plasma .raw files from MassIVE study MSV000091033) and run Compound Discoverer 3.3 peak extraction with m/z range 100-5000, retention time 0.4-21 min, and 5e5 intensity threshold, producing a .cdResult peak table. Run LipiDex 2 Spectrum Searcher against the in silico lipid library with 75% spectral purity and dot product 500/700 scoring on MS2/MS3+ spectra. Run LipiDex 2 Peak Finder, which reads Compound Discoverer .cdResult files directly, to annotate the peak table and remove adducts, dimers, and other spurious features; apply QC filters such as flagging features with >30% RSD in QC samples, RT-deviation filtering from the ECN vs. RT model, and required-fragment logic that downgrades identifications to sum composition. Finally apply the 7 filtering criteria and count unidentified versus identified features, saving results as LipiDex2_PeakFinder_Results.csv. Correctness is judged by matching reported counts (e.g. 14826 unidentified / 779 identified before filtering; 13898 / 719 after Peak Finder filtering).

## Related tools

- **LipiDex 2** (Stand-alone pipeline (Spectrum Searcher, Peak Finder, Degreaser, Lipid Spectral Annotator) for spectral matching, peak annotation, and QC filtering of lipidomics data) — https://github.com/coongroup/LipiDex-2
- **Compound Discoverer** (Peak extraction software (version 3.3) producing .cdResult peak tables consumed directly by LipiDex 2 Peak Finder) — https://www.thermofisher.com/us/en/home/industrial/mass-spectrometry/liquid-chromatography-mass-spectrometry-lc-ms/lc-ms-software/multi-omics-data-analysis/compound-discoverer-software.html
- **LipiDex 2 Library Generator** (Generates the in silico lipid spectral library (with MSn tree-based fragmentation) used by Spectrum Searcher for identification) — https://github.com/coongroup/LipiDex-2/releases/latest

## Evaluation signals

- Output CSV contains exactly 14826 unidentified features and 779 identified lipid features before Peak Finder filtering
- After Peak Finder filtering, counts match the reported 13898 unidentified and 719 identified features
- Features with >30% RSD in QC samples are flagged for removal
- Identifications lacking required fragments are downgraded to sum composition (and insufficient plasmenyl evidence downgraded to plasmanyl)
- Peak Finder accepts Compound Discoverer .cdResult files directly without intermediate .csv conversion

## Limitations

- Peak Finder requires Compound Discoverer version 3.3 .cdResult files as input; older or other vendors' peak tables are not supported
- .NET 7.0 runtime is required for the stand-alone LipiDex2.exe
- LipiDex 1 is no longer supported; legacy workflows in Java (.mgf/.mzXML input) differ from the LipiDex 2 pipeline
- The LipiDex 2 Wiki documentation is a work in progress (WiP)

## Evidence
