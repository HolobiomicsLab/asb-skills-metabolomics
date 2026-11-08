---
name: msv000091033-reprocessed-data-analysis
description: Use when analyzing or reproducing results on the MSV000091033 dataset
  (40 Nile rat plasma samples) reprocessed with LipiDex 2, or when validating Degreaser
  automated filter decisions against manual expert review.
when_to_use_negative:
- Input is already a filtered/curated feature table that has not been produced by
  the LipiDex 2 peak-identification and spectral-matching steps Degreaser expects
  as input
- Analysis goal is de novo lipid discovery on datasets other than MSV000091033 without
  the QC evidence (QC sample %RSD, dilution series, ECN vs. RT model) needed to reproduce
  the reported counts
- Peak tables come from tools other than Compound Discoverer 3.3 (.cdResult) or mzMine
  2 / .csv conversions, since Peak Finder requires those specific inputs
tools:
- name: LipiDex 2
  role: Primary pipeline (Library Generator, Spectrum Searcher, Peak Finder) used
    to reprocess MSV000091033 and produce lipid identifications
  repo: https://github.com/coongroup/LipiDex-2
- name: LipiDex 2 Degreaser QC module
  role: Automated threshold filtering of identifications using 6 pieces of evidence
    plus manual validation dashboard
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: LipiDex 2 Lipid Spectral Annotator
  role: Annotates fragment peaks from Spectrum Searcher MS2 and MS3+ spectral matches
    during QC review
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: Metaboanalyst
  role: Downstream statistical analysis of the filtered, QC-validated lipid quantitation
    output
- name: LipiDex 1
  role: Legacy predecessor whose data-processing algorithms LipiDex 2 improves upon;
    still available if needed
  repo: https://github.com/coongroup/LipiDex
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:d3721430f2cbd56f766547ad0ef5764d12753f9042e4226b842035433160cb2b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coongroup/LipiDex-2
  tools:
  - LipiDex 1
  - LipiDex 2
  - LipiDex 2 Degreaser QC module
  - LipiDex 2 Lipid Spectral Annotator
  - Metaboanalyst
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# msv000091033-reprocessed-data-analysis

## Summary

Reproduces the LipiDex 2 reprocessing and Degreaser QC analysis of the MSV000091033 Nile rat plasma lipidomics dataset (40 samples), including automated threshold filtering followed by manual validation of lipid identification filter decisions.

## When to use

Use when analyzing or reproducing results on the MSV000091033 dataset (40 Nile rat plasma samples) reprocessed with LipiDex 2, or when validating Degreaser automated filter decisions against manual expert review.

## When NOT to use

- Input is already a filtered/curated feature table that has not been produced by the LipiDex 2 peak-identification and spectral-matching steps Degreaser expects as input
- Analysis goal is de novo lipid discovery on datasets other than MSV000091033 without the QC evidence (QC sample %RSD, dilution series, ECN vs. RT model) needed to reproduce the reported counts
- Peak tables come from tools other than Compound Discoverer 3.3 (.cdResult) or mzMine 2 / .csv conversions, since Peak Finder requires those specific inputs

## Inputs

- MSV000091033 dataset: 40 experimental Nile rat plasma samples (publicly available, reprocessed with LipiDex 2)
- LipiDex 2 peak identification and spectral matching results (.csv files passed between modules)
- Compound Discoverer 3.3 .cdResult files (Peak Finder input for the peak table)
- Thermo .raw files supported as Spectrum Searcher input

## Outputs

- Degreaser-filtered lipid identification table with automated filter statuses
- manual_validation_summary.csv with per-identification filter status decisions and final counts (46 changed statuses; 480 total identifications, 19 above the automated count)
- Results compatible with Metaboanalyst for downstream statistical analysis

## How to apply

Run the LipiDex 2 pipeline (Library Generator, Spectrum Searcher, Peak Finder) on the MSV000091033 dataset, then feed peak-identification and spectral-matching results into the Degreaser QC module. Apply the automated threshold filters — features with >30% RSD in QC samples are flagged for removal, ECN-vs-RT modeling filters identifications with large deviation from predicted RT, and the results are filtered on 7 criteria total (adducts, dimers, in-source fragments, dilution-series linear dynamic range, biosample quantitations within linear range, spectral match validation, and Compound Discoverer Peak Quality Factors). Then perform optional manual validation in the Degreaser dashboard, re-evaluating each flagged identification as a mass spectrometrist would, since some automatic cutoffs are too stringent or miss nuances only visible in the QC plots. For the reference reproduction, tally identifications whose filter status changes (46 changed statuses), update the accepted-identification count (+19 over the automated count, reaching 480 total), and export per-identification decisions to manual_validation_summary.csv.

## Related tools

- **LipiDex 2** (Primary pipeline (Library Generator, Spectrum Searcher, Peak Finder) used to reprocess MSV000091033 and produce lipid identifications) — https://github.com/coongroup/LipiDex-2
- **LipiDex 2 Degreaser QC module** (Automated threshold filtering of identifications using 6 pieces of evidence plus manual validation dashboard) — https://github.com/coongroup/LipiDex-2/releases/latest
- **LipiDex 2 Lipid Spectral Annotator** (Annotates fragment peaks from Spectrum Searcher MS2 and MS3+ spectral matches during QC review) — https://github.com/coongroup/LipiDex-2/releases/latest
- **Metaboanalyst** (Downstream statistical analysis of the filtered, QC-validated lipid quantitation output)
- **LipiDex 1** (Legacy predecessor whose data-processing algorithms LipiDex 2 improves upon; still available if needed) — https://github.com/coongroup/LipiDex

## Examples

```
LipiDex2.exe  # after downloading and extracting the .zip from https://github.com/coongroup/LipiDex-2/releases/latest (requires .NET 7.0); load MSV000091033 results and run Degreaser QC with manual validation
```

## Evaluation signals

- Manual-validation reproduction must tally exactly 46 lipid identifications whose filter status changed relative to the automated-filter outcome
- Accepted-identification count must reach 480 total, exactly 19 above the automated-filter count; values outside these indicate a failed reproduction
- Automated filtering must flag features with >30% RSD in QC samples for removal
- Results table must carry all 7 filter criteria and remain directly compatible with Metaboanalyst input format
- Per-identification decisions in manual_validation_summary.csv must be traceable to Degreaser evidence types (ECN vs. RT, %RSD with run order, spectral match validation, dilution series, linear range, PQF)

## Limitations

- Automated cutoffs can be too stringent or miss nuances only visible in the QC plots, requiring optional manual validation by a mass spectrometrist
- Peak Finder requires Compound Discoverer version 3.3 .cdResult files; LipiDex 2 requires .NET 7.0 and is a stand-alone Windows executable (LipiDex2.exe)
- The LipiDex 2 Wiki instructions are marked work-in-progress (WiP), and legacy LipiDex 1 is no longer supported
- Manual validation is a human-in-the-loop step and cannot be fully automated; the reported 46/480 outcome depends on expert judgment in the Degreaser dashboard

## Evidence

- [other] Manual evaluation changed 46 filter statuses; final count 480 (+19): [evidence span withheld — anchor status: not_found]
