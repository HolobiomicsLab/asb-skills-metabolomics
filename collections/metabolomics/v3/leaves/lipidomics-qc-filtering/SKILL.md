---
name: lipidomics-qc-filtering
description: Use when downstream of LipiDex 2 Peak Finder and Spectrum Searcher results
  you need to remove spurious/redundant lipid identifications and unidentified features
  using automated lipid-specific QC thresholds before statistical analysis.
when_to_use_negative:
- Input is a raw peak table that has not yet passed Peak Finder and Spectrum Searcher
  — Degreaser takes results from those steps, not raw LC-MS data
- Input comes from LipiDex 1 or non-Compound Discoverer peak tables without the QC
  metadata (QC replicates, dilution series, PQF) the Degreaser criteria require
- The goal is spectral library generation or fragmentation annotation — use the Library
  Generator or Lipid Spectral Annotator modules instead
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: LipiDex 2 Degreaser QC module
  role: Performs the automated QC filtering of lipid identifications and features
    using 6 pieces of evidence across 7 filtering criteria
  repo: https://github.com/coongroup/LipiDex-2
- name: LipiDex 2
  role: Pipeline (Peak Finder, Spectrum Searcher) producing the peak tables and spectral
    matches that Degreaser filters
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: Metaboanalyst
  role: Downstream statistical analysis target compatible with Degreaser outputs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:e31182c17a9b250c8c3f14e3ae06ea750a3a872754a702676ef28e496391e58a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coongroup/LipiDex-2
  tools:
  - LipiDex 2
  - LipiDex 2 Degreaser QC module
  - Metaboanalyst
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# lipidomics-qc-filtering

## Summary

Apply LipiDex 2's Degreaser QC module to automatically validate and filter lipid identifications and features from LC-MS lipidomics peak tables, using seven criteria (ECN-vs-RT modeling, linear dynamic range, %RSD in QC samples, spectral match validation, PQF) to boost identification confidence.

## When to use

Use when downstream of LipiDex 2 Peak Finder and Spectrum Searcher results you need to remove spurious/redundant lipid identifications and unidentified features using automated lipid-specific QC thresholds before statistical analysis.

## When NOT to use

- Input is a raw peak table that has not yet passed Peak Finder and Spectrum Searcher — Degreaser takes results from those steps, not raw LC-MS data
- Input comes from LipiDex 1 or non-Compound Discoverer peak tables without the QC metadata (QC replicates, dilution series, PQF) the Degreaser criteria require
- The goal is spectral library generation or fragmentation annotation — use the Library Generator or Lipid Spectral Annotator modules instead

## Inputs

- LipiDex 2 Peak Finder filtered peak table (results after removal of adducts, dimers, and other spurious features)
- Spectrum Searcher spectral matches for MS2 and MS3+ spectra
- Replicate quality control sample annotations (with run order)
- Dilution series data for linear dynamic range modeling

## Outputs

- Final filtered peak table of lipid identifications and unidentified features aggregated across 7 filtering criteria
- Per-criterion filter attribution summary (which identifications were removed by each filter)
- QC visualization/dashboard of lipid-specific quality metrics
- Results formatted for direct compatibility with Metaboanalyst

## How to apply

Load the Peak Finder filtered results (adducts, dimers, and other spurious features already removed) and Spectral Searcher spectral matches as input to the Degreaser module. Apply the effective carbon number versus retention time (ECN vs. RT) model to filter likely incorrect identifications whose measured RT deviates substantially from predicted RT, and the dilution series linear dynamic range model to remove biosample quantitations falling outside the established linear range. Validate spectral matches for all matches in each lipid feature, flag features with >30% RSD in replicate quality control samples (with run order), and apply Compound Discoverer's Peak Quality Factor (PQF) criteria. Aggregate the seven filtering criteria into a final filtered table, optionally using Degreaser's manual validation visualizations, and output results compatible with Metaboanalyst. In the Nile rat plasma example (MSV000091033, 40 experimental samples), this reduced results to 2443 unidentified features and 461 identifications, raising the identification rate from 5% to 19%.

## Related tools

- **LipiDex 2 Degreaser QC module** (Performs the automated QC filtering of lipid identifications and features using 6 pieces of evidence across 7 filtering criteria) — https://github.com/coongroup/LipiDex-2
- **LipiDex 2** (Pipeline (Peak Finder, Spectrum Searcher) producing the peak tables and spectral matches that Degreaser filters) — https://github.com/coongroup/LipiDex-2/releases/latest
- **Metaboanalyst** (Downstream statistical analysis target compatible with Degreaser outputs)

## Examples

```
LipiDex2.exe
```

## Evaluation signals

- Filtered output counts match expected values: on MSV000091033 Nile rat plasma (40 experimental samples), 461 identifications, 254 identified lipids, and 2443 unidentified features survive filtering
- Identification rate improves from 5% pre-filter to 19% post-filter
- Single-filter attribution table is consistent (e.g., 173 of 254 identified lipids removed by a single filter criterion)
- Features with >30% RSD in QC samples are all flagged for removal
- Filtered quantitation table loads into Metaboanalyst without additional reformatting

## Limitations

- Degreaser requires Peak Finder input from Compound Discoverer 3.3 .cdResult files, so non-Compound Discoverer workflows need intermediate conversion
- Automated threshold filtering is a hybrid workflow — some identifications may still require optional manual validation by the mass spectrometrist
- The wiki/instructions were a work in progress at time of the README ('Instructions are found in the LipiDex 2 Wiki (WiP)')
- .NET 7.0 is required to run the stand-alone executable

## Evidence
