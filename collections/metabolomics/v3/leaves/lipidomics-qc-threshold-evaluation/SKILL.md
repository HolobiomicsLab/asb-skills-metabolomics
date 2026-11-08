---
name: lipidomics-qc-threshold-evaluation
description: Use when reprocessing LC-MS/MS lipidomics peak tables (e.g., Compound
  Discoverer .cdResult-derived identifications) and you must validate automated filter
  decisions on lipid identifications using RSD, RT, and spectral-match evidence, with
  or without manual curation.
when_to_use_negative:
- Input is raw MS/MS spectra requiring spectral library generation or spectral matching
  first — use LipiDex 2 Library Generator and Spectrum Searcher before Degreaser
- Input is already a fully manually curated, statistics-ready feature table with no
  automated filter statuses to re-evaluate
- Non-lipidomics small-molecule peak tables where lipid-specific QC metrics like ECN
  vs. RT modeling do not apply
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: LipiDex 2 Degreaser QC module
  role: Performs automated threshold filtering on six evidence types and provides
    the dashboard for manual validation of filter statuses
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: LipiDex 2
  role: Upstream pipeline (Library Generator, Spectrum Searcher, Peak Finder) whose
    peak identification and spectral matching results feed Degreaser; stand-alone
    LipiDex2.exe on .NET 7.0
  repo: https://github.com/coongroup/LipiDex-2
- name: Metaboanalyst
  role: Downstream statistical analysis target; Degreaser outputs results ready for
    direct Metaboanalyst compatibility
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:758633ed51d30356692fc4c9ecc71c11954cb1dcb65e763ae7405be5c331c774
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LipiDex 2
  - LipiDex 2 Degreaser QC module
  - Metaboanalyst
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# lipidomics-qc-threshold-evaluation

## Summary

A hybrid lipid-identification QC workflow from LipiDex 2's Degreaser module that combines automated threshold filtering (six evidence types) with optional manual dashboard validation, so mass spectrometrists can overturn overly stringent or missed automated filter decisions before statistical analysis.

## When to use

Use when reprocessing LC-MS/MS lipidomics peak tables (e.g., Compound Discoverer .cdResult-derived identifications) and you must validate automated filter decisions on lipid identifications using RSD, RT, and spectral-match evidence, with or without manual curation.

## When NOT to use

- Input is raw MS/MS spectra requiring spectral library generation or spectral matching first — use LipiDex 2 Library Generator and Spectrum Searcher before Degreaser
- Input is already a fully manually curated, statistics-ready feature table with no automated filter statuses to re-evaluate
- Non-lipidomics small-molecule peak tables where lipid-specific QC metrics like ECN vs. RT modeling do not apply

## Inputs

- LipiDex 2 peak identification and spectral matching results (.csv results files passed through the pipeline)
- Compound Discoverer 3.3 .cdResult peak tables (direct input to Peak Finder)
- Reprocessed LC-MS lipidomics dataset (e.g., MSV000091033, 40 Nile rat plasma experimental samples)

## Outputs

- Degreaser-filtered lipid identification table with automated filter statuses per identification
- Manual-validation summary of per-identification filter status decisions and final accepted-identification counts
- Metaboanalyst-compatible results for downstream statistical analysis

## How to apply

Run Degreaser on results from the peak identification and spectral matching steps of the LipiDex 2 pipeline; it evaluates six pieces of evidence per identification: effective carbon number versus retention time (ECN vs. RT) modeling, dilution-series linear dynamic range modeling, biosample quantitations within linear range, spectral match validation for all matches in the lipid feature, replicate QC-sample %RSD with run order (features with >30% RSD in QC samples are flagged for removal), and Compound Discoverer Peak Quality Factors. Results are filtered on 7 criteria to boost confidence in identifications and quantitation. Then optionally perform manual validation in the Degreaser dashboard: for each automated flag, re-evaluate the filter status as a mass spectrometrist would using the interactive QC plots (some automatic cutoffs are too stringent or miss nuances only visible in the plots), tally status changes, and update the accepted-identification count. Finally, export results that are directly compatible with Metaboanalyst without coding.

## Related tools

- **LipiDex 2 Degreaser QC module** (Performs automated threshold filtering on six evidence types and provides the dashboard for manual validation of filter statuses) — https://github.com/coongroup/LipiDex-2/releases/latest
- **LipiDex 2** (Upstream pipeline (Library Generator, Spectrum Searcher, Peak Finder) whose peak identification and spectral matching results feed Degreaser; stand-alone LipiDex2.exe on .NET 7.0) — https://github.com/coongroup/LipiDex-2
- **Metaboanalyst** (Downstream statistical analysis target; Degreaser outputs results ready for direct Metaboanalyst compatibility)

## Examples

```
Download the LipiDex 2 .zip, extract, and run LipiDex2.exe; open Degreaser and load the peak identification .csv results from the MSV000091033 Nile rat plasma run, then review flagged identifications (>30% QC %RSD, ECN-vs-RT deviation) in the dashboard before exporting to Metaboanalyst.
```

## Evaluation signals

- In the MSV000091033 reproduction, manual validation changes the filter status of 46 lipid identifications relative to the automated outcome
- Final accepted count reaches 480 identifications — 19 above the automated-filter count; counts outside these values indicate a failed reproduction
- Every feature flagged for removal satisfies the automated evidence rules, e.g., >30% RSD in QC samples
- Outputs pass through the pipeline as .csv results files and import directly into Metaboanalyst without coding
- Identifications downgraded per required-fragment logic (e.g., to sum composition or plasmanyl) match the expected downgrade rules

## Limitations

- Automatic cutoffs can be too stringent or miss nuances only visible in the QC plots, necessitating manual dashboard validation
- Peak Finder requires Compound Discoverer version 3.3 .cdResult files as input for the peak table; other formats need intermediate conversion to .csv
- LipiDex 2 requires .NET 7.0; LipiDex 1 is legacy and no longer supported
- The LipiDex 2 Wiki instructions are marked work-in-progress (WiP)

## Evidence
