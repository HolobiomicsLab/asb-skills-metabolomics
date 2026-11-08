---
name: replicate-rsd-quality-control
description: Use when QC filtering lipidomics feature tables (e.g. LipiDex 2 Peak
  Finder/Spectrum Searcher results) where replicate quality control samples were injected
  and features with >30% RSD should be flagged for removal.
when_to_use_negative:
- The dataset lacks replicate quality control sample injections, so RSD across replicates
  cannot be computed.
- The task is upstream peak detection or spectral matching rather than post-identification
  QC filtering of a feature table.
edam_operation: http://edamontology.org/operation_3281
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: LipiDex 2 Degreaser QC module
  role: Implements the replicate QC sample %RSD (with run order) filtering criterion
    and aggregates it with the other QC filters
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: LipiDex 2
  role: Parent pipeline whose Peak Finder and Spectrum Searcher outputs are the inputs
    to the %RSD QC filtering
  repo: https://github.com/coongroup/LipiDex-2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:027a930a196c391f2a9756cd51cafa0f5c217fc8e963ba19c42ec5124982de6c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LipiDex 2
  - LipiDex 2 Degreaser QC module
  edam_operation: http://edamontology.org/operation_3281
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# replicate-rsd-quality-control

## Summary

Applies replicate quality control sample percent relative standard deviation (%RSD) analysis, with run-order tracking, to flag lipid features with excessive quantitation variability for removal, as implemented in the LipiDex 2 Degreaser QC module.

## When to use

Use when QC filtering lipidomics feature tables (e.g. LipiDex 2 Peak Finder/Spectrum Searcher results) where replicate quality control samples were injected and features with >30% RSD should be flagged for removal.

## When NOT to use

- The dataset lacks replicate quality control sample injections, so RSD across replicates cannot be computed.
- The task is upstream peak detection or spectral matching rather than post-identification QC filtering of a feature table.

## Inputs

- LipiDex 2 Peak Finder filtered results (.csv peak table)
- Spectrum Searcher spectral matches (.csv)
- Replicate quality control sample injections with run-order metadata

## Outputs

- Filtered feature table with >30% RSD features flagged/removed
- Single-filter attribution summary counting identifications removed per QC criterion
- Final filtered table (e.g. 254 identified lipids, 461 identifications, 2443 unidentified features) with improved identification rate

## How to apply

Load the Peak Finder filtered results and Spectrum Searcher spectral matches for the experimental samples plus replicate quality control samples into the LipiDex 2 Degreaser QC module. Compute the percent relative standard deviation (%RSD) of quantitation across replicate QC samples, tracking run order, and flag any feature exceeding the 30% RSD threshold for removal. Degreaser aggregates this %RSD criterion with six other pieces of evidence (ECN vs. RT modeling, dilution series linear dynamic range, biosample quantitations within linear range, spectral match validation, and Compound Discoverer Peak Quality Factors) so results are filtered on 7 criteria; report single-filter attributions for identifications removed. On the Nile rat plasma dataset (MSV000091033), this QC step contributed to filtering 254 identified lipids, improving identification rate from 5% to 19% with 461 identifications and 2443 unidentified features retained.

## Related tools

- **LipiDex 2 Degreaser QC module** (Implements the replicate QC sample %RSD (with run order) filtering criterion and aggregates it with the other QC filters) — https://github.com/coongroup/LipiDex-2/releases/latest
- **LipiDex 2** (Parent pipeline whose Peak Finder and Spectrum Searcher outputs are the inputs to the %RSD QC filtering) — https://github.com/coongroup/LipiDex-2

## Examples

```
Download LipiDex2.zip from https://github.com/coongroup/LipiDex-2/releases/latest, run LipiDex2.exe, and in the Degreaser QC module load Peak Finder and Spectrum Searcher .csv results with replicate QC samples to apply the >30% RSD filter.
```

## Evaluation signals

- Features flagged for removal should be exactly those with >30% RSD across replicate quality control samples.
- Run-order information should be incorporated into the %RSD QC visualization/metric.
- Filtered output for MSV000091033 Nile rat plasma should contain 461 identifications, 254 identified lipids, 2443 unidentified features, and a 19% identification rate (up from 5%).
- The single-filter attribution table should be consistent, e.g. 173 of 254 removed identifications attributed to a single filter.
- Results should pass all 7 filtering criteria aggregations as described for Degreaser.

## Limitations

- The 30% RSD cutoff is the documented default; datasets with different replicate designs or variability expectations may need threshold adjustment.
- Degreaser requires Compound Discoverer 3.3 .cdResult-derived peak tables (via Peak Finder), so peak tables from other sources need prior conversion to .csv.
- %RSD filtering alone may remove valid low-abundance features with high measurement noise; it is intended to be combined with the other 6 QC evidence types.

## Evidence
