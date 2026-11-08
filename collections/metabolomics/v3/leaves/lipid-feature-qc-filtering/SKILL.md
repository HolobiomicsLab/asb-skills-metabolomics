---
name: lipid-feature-qc-filtering
description: Use when a lipidomics peak table (e.g. from Compound Discoverer 3.3 .cdResult)
  contains adduct peaks, dimers, or identifications failing QC evidence such as >30%
  RSD in QC samples, and must be filtered before quantitation.
when_to_use_negative:
- Input is still raw MS/MS data before spectral matching and peak identification —
  filtering operates on post-identification results from the LipiDex 2 pipeline
- Peak table originates from a tool other than Compound Discoverer 3.3 — Peak Finder
  requires .cdResult files as input for the peak table
- Input is already a manually curated or previously filtered feature table where adduct/dimer
  removal would be redundant
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: LipiDex 2 Peak Finder
  role: Reads Compound Discoverer 3.3 .cdResult peak tables and performs filtering
    to remove adducts, dimers, and other spurious features
  repo: https://github.com/coongroup/LipiDex-2
- name: LipiDex 2 Degreaser
  role: QC module that combines automated threshold filtering with optional manual
    validation of lipid identifications and features using 6 pieces of evidence
  repo: https://github.com/coongroup/LipiDex-2
- name: Compound Discoverer
  role: Upstream peak extraction software whose .cdResult peak table and Peak Quality
    Factors (PQF) feed the filtering workflow
  repo: https://www.thermofisher.com/us/en/home/industrial/mass-spectrometry/liquid-chromatography-mass-spectrometry-lc-ms/lc-ms-software/multi-omics-data-analysis/compound-discoverer-software.html
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:85ebfc91134b34541eb6b86bf5ae1a457e51a99532344c2909e07a037a8aae9e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coongroup/LipiDex-2
  tools:
  - Compound Discoverer
  - LipiDex 2
  - LipiDex 2 Degreaser
  - LipiDex 2 Peak Finder
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 24
---

# lipid-feature-qc-filtering

## Summary

Post-identification quality control for LC-MS lipidomics using LipiDex 2's Peak Finder filtering and Degreaser QC module to remove adducts, dimers, in-source fragments, and other spurious features and to downgrade identifications lacking required fragment evidence.

## When to use

Use when a lipidomics peak table (e.g. from Compound Discoverer 3.3 .cdResult) contains adduct peaks, dimers, or identifications failing QC evidence such as >30% RSD in QC samples, and must be filtered before quantitation.

## When NOT to use

- Input is still raw MS/MS data before spectral matching and peak identification — filtering operates on post-identification results from the LipiDex 2 pipeline
- Peak table originates from a tool other than Compound Discoverer 3.3 — Peak Finder requires .cdResult files as input for the peak table
- Input is already a manually curated or previously filtered feature table where adduct/dimer removal would be redundant

## Inputs

- Compound Discoverer 3.3 .cdResult peak table (required input for Peak Finder)
- LipiDex 2 Spectrum Searcher / Peak Finder identification results (.csv)
- Thermo .raw LC-MS/MS files (e.g. the 40-sample Nile rat plasma MSV000091033 dataset)

## Outputs

- Filtered peak feature table with adducts, dimers, and spurious features removed (e.g. LipiDex2_PeakFinder_Results.csv)
- Post-filter feature counts for unidentified and identified features
- Downgraded lipid identifications (sum composition or plasmanyl) where fragment evidence was insufficient
- Degreaser output compatible with Metaboanalyst for downstream statistical analysis

## How to apply

Load the peak table into LipiDex 2 Peak Finder (it reads Compound Discoverer 3.3 .cdResult files directly, avoiding intermediate .csv conversion) and run filtering to remove adducts, dimers, and other spurious features correlated with primary lipid features. Apply the 7 filter criteria that boost confidence in identifications and quantitation, including RT-model filtering of identifications with large deviation from predicted retention time (ECN vs. RT modeling). Use Degreaser's automated threshold filtering, which validates lipid identifications and features against 6 pieces of evidence: ECN vs. RT, dilution series linear dynamic range, biosample quantitations within linear range, spectral match validation for all matches in the lipid feature, replicate QC samples' %RSD with run order, and Compound Discoverer's Peak Quality Factors (PQF). Apply required-fragment logic: identifications are downgraded to sum composition when a required fragment is absent, and to plasmanyl when plasmenyl ether evidence is insufficient. As a benchmark, on the MSV000091033 Nile rat plasma dataset this reduced 14826 unidentified / 779 identified features to 13898 unidentified / 719 identified.

## Related tools

- **LipiDex 2 Peak Finder** (Reads Compound Discoverer 3.3 .cdResult peak tables and performs filtering to remove adducts, dimers, and other spurious features) — https://github.com/coongroup/LipiDex-2
- **LipiDex 2 Degreaser** (QC module that combines automated threshold filtering with optional manual validation of lipid identifications and features using 6 pieces of evidence) — https://github.com/coongroup/LipiDex-2
- **Compound Discoverer** (Upstream peak extraction software whose .cdResult peak table and Peak Quality Factors (PQF) feed the filtering workflow) — https://www.thermofisher.com/us/en/home/industrial/mass-spectrometry/liquid-chromatography-mass-spectrometry-lc-ms/lc-ms-software/multi-omics-data-analysis/compound-discoverer-software.html

## Examples

```
LipiDex2.exe  # run LipiDex 2 -> Peak Finder, load Compound Discoverer 3.3 .cdResult peak table, apply QC filtering, save LipiDex2_PeakFinder_Results.csv
```

## Evaluation signals

- Post-filter counts on the benchmark MSV000091033 dataset match 13898 unidentified and 719 identified features (reduced from 14826 unidentified and 779 identified)
- Features with >30% RSD in QC samples are flagged for removal in the output
- Identifications missing required fragments appear as downgraded sum compositions, and insufficient plasmenyl evidence appears as plasmanyl
- Filtered results are written to a .csv (e.g. LipiDex2_PeakFinder_Results.csv) with remaining features per the 7 filter criteria
- Degreaser-filtered output loads directly into Metaboanalyst without coding

## Limitations

- Peak Finder requires Compound Discoverer version 3.3 .cdResult files as input for the peak table; intermediate .csv conversion is bypassed only in LipiDex 2
- The RT model filters likely incorrect identifications on lipids with large deviation from predicted RT, so identifications of lipids with poorly modeled retention behavior may be spuriously filtered or downgraded
- Degreaser's automated threshold filtering is a hybrid workflow — optional manual validation may still be required for borderline features
- The LipiDex 2 Wiki instructions were a work in progress at the time of the README

## Evidence
