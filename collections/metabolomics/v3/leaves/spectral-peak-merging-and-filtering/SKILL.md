---
name: spectral-peak-merging-and-filtering
description: Use when preparing MGF/mzXML/mzML query spectra for VInSMoC exact or
  variable search and raw spectra need noise reduction via peak merging and intensity-window
  filtering before scoring.
when_to_use_negative:
- When input spectra have already been pre-processed (merged/filtered) before uploading
  to the npanalysis.org server — the README says advanced settings 'should be need
  to be changed often' and to 'leave unchanged unless you have already done some preprocessing'.
- When only neutral-mass/adduct handling is needed (charge conversion to [M+H]+ or
  [M-H]-) and no peak-level noise reduction is required.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: VInSMoC
  role: Executes the peak merging and peak filtering pre-processing on uploaded mass
    spectral inputs before scoring query spectra against a chemical database in exact
    or variable mode
  repo: https://github.com/mohimanilab/VInSMoC
- name: run.npanalysis.org server
  role: Hosts the VInSMoC task form where advanced settings (Peak Merge Threshold,
    Minimum Peak Count, Peaks Per Window, Peak Filtering Window) are configured
  repo: https://run.npanalysis.org
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:9272d24ad09dc30194acdce4de599a67c972f9a911a8dabb3ac6ced27294859e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  - run.npanalysis.org server
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# spectral-peak-merging-and-filtering

## Summary

Pre-processing of query mass spectra before scoring in VInSMoC: peaks within a configurable threshold are merged while retaining a minimum peak count, and only the most intense peaks per m/z window are kept, reducing noise without losing candidate fragments.

## When to use

Use when preparing MGF/mzXML/mzML query spectra for VInSMoC exact or variable search and raw spectra need noise reduction via peak merging and intensity-window filtering before scoring.

## When NOT to use

- When input spectra have already been pre-processed (merged/filtered) before uploading to the npanalysis.org server — the README says advanced settings 'should be need to be changed often' and to 'leave unchanged unless you have already done some preprocessing'.
- When only neutral-mass/adduct handling is needed (charge conversion to [M+H]+ or [M-H]-) and no peak-level noise reduction is required.

## Inputs

- Query mass spectra in MGF, mzXML, or mzML format (e.g. extract-3_Metal-mix_1mM.mzML from MSV000086287)
- Advanced-settings parameters: Peak Merge Threshold, Minimum Peak Count, Peaks Per Window, Peak Filtering Window

## Outputs

- Pre-processed query spectrum (merged, filtered peak list) fed into VInSMoC scoring
- Peak count score contributions reported in VInSMoC hit outputs (vinSMoC_exact_mode_hits.csv)

## How to apply

Apply the two advanced settings in the VInSMoC task form before submitting a run. First, peak merging combines peaks that lie within the 'Peak Merge Threshold' m/z distance of one another, ensuring at least 'Minimum Peak Count' peaks remain so the spectrum is not reduced to nothing. Second, peak filtering retains only the 'Peaks Per Window' most intense peaks in m/z windows of size 'Peak Filtering Window', concentrating the spectrum on high-intensity signals. The resulting pre-processed spectrum is scored against candidate molecules (e.g. NPAtlas) with theoretical fragments matched to experimental peaks within the product ion tolerance. The README advises leaving these settings at their defaults unless spectra were already pre-processed before upload, so downstream interpretation must account for whether raw or pre-cleaned spectra are supplied.

## Related tools

- **VInSMoC** (Executes the peak merging and peak filtering pre-processing on uploaded mass spectral inputs before scoring query spectra against a chemical database in exact or variable mode) — https://github.com/mohimanilab/VInSMoC
- **run.npanalysis.org server** (Hosts the VInSMoC task form where advanced settings (Peak Merge Threshold, Minimum Peak Count, Peaks Per Window, Peak Filtering Window) are configured) — https://run.npanalysis.org

## Examples

```
On run.npanalysis.org: create VInSMoC task → select 'Mass Spectrum' input (e.g. extract-3_Metal-mix_1mM.mzML) → set Advanced Settings → submit.
```

## Evaluation signals

- Post-merging spectrum retains at least 'Minimum Peak Count' peaks (no over-collapse).
- After filtering, each m/z window of size 'Peak Filtering Window' contains at most 'Peaks Per Window' peaks.
- Remaining peaks fall within the product ion tolerance when matched against theoretical fragments of candidate molecules.
- Task reaches 'Finished' state with best-hit results exceeding the minimum score reporting threshold on the task page.
- Hits link correctly back to the originating chemical database entries (e.g. NPAtlas).

## Limitations

- Defaults are intended for raw spectra; supplying spectra pre-processed elsewhere may double-filter and bias scoring.
- Peak filtering is intensity-driven within fixed m/z windows, so low-intensity diagnostic fragments may be dropped.
- Only applicable to the VInSMoC/npanalysis.org workflow; not a general-purpose standalone peak-picking module.

## Evidence

- [methods] Match theoretical fragments to experimental peaks within the product ion tolerance and compute the exact-mode peak count score for each candidate molecule.: [evidence span withheld — anchor status: not_found]
