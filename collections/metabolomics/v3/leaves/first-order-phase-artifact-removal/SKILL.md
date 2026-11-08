---
name: first-order-phase-artifact-removal
description: Use when working with raw Bruker 1H-NMR FIDs whose points carry the digital-filter
  group-delay artifact, after extraction of fids_df and acqus_df and before solvent
  removal or apodization.
when_to_use_negative:
- Data are not raw Bruker FIDs (e.g. already Fourier-transformed spectra) — the leading-point
  deletion applies only to time-domain FIDs with a recorded group delay in acqus
- acqus_df lacks the group-delay/DSP parameter, so the number of points to delete
  cannot be determined per sample
- The FID has already been group-delay corrected (grpd_df exists); deleting points
  again would remove genuine signal
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: Protomix
  role: Provides the group_delay_removal function that strips the acqus-specified
    number of leading points from each FID
  repo: https://github.com/mzniber/protomix
- name: pandas
  role: Holds the fids_df and acqus_df DataFrames that are the function's inputs and
    output
- name: numpy
  role: Underlying numeric representation of the complex FID time-series data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:9fc7af3fee0506bb9903384cbe65e177c17ade2c315101a4f3653b604475d021
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# first-order-phase-artifact-removal

## Summary

Removes the Bruker digital-filter group delay ('death time') artifact from each FID by deleting exactly the acqus-specified number of leading data points, correcting the first-order phase error before further preprocessing.

## When to use

Use when working with raw Bruker 1H-NMR FIDs whose leading points carry the digital-filter group-delay artifact, after extraction of fids_df and acqus_df and before solvent removal or apodization.

## When NOT to use

- Data are not raw Bruker FIDs (e.g. already Fourier-transformed spectra) — the leading-point deletion applies only to time-domain FIDs with a recorded group delay in acqus
- acqus_df lacks the group-delay/DSP parameter, so the number of points to delete cannot be determined per sample
- The FID has already been group-delay corrected (grpd_df exists); deleting points again would remove genuine signal

## Inputs

- fids_df: pandas DataFrame of extracted Free Induction Decays (samples as rows, time points as columns), from px.extract_fids
- acqus_df: pandas DataFrame of Bruker acquisition parameters, from px.extract_params, containing the group-delay (DSP) parameter per sample

## Outputs

- grpd_df: pandas DataFrame of group-delay-corrected FIDs, each trace shortened by the acqus-specified number of leading points

## How to apply

Load the extracted FID table (fids_df) and acquisition-parameters table (acqus_df) produced by Protomix's px.extract_fids and px.extract_params. Read the group-delay parameter, derived from the Bruker acqus DSP settings, for each sample from acqus_df. Call Protomix's group_delay_removal, which deletes exactly that many leading data points from each FID — no complex transformation is needed because the artifact is a fixed point offset. Save the corrected table as grpd_df with samples as rows and time points as columns, then pass it into subsequent steps such as solvent_residuals_removal and apodization.

## Related tools

- **Protomix** (Provides the group_delay_removal function that strips the acqus-specified number of leading points from each FID) — https://github.com/mzniber/protomix
- **pandas** (Holds the fids_df and acqus_df DataFrames that are the function's inputs and output)
- **numpy** (Underlying numeric representation of the complex FID time-series data)

## Examples

```
grpd_df = px.group_delay_removal(fid_df=fids_df, acqus_df=acqus_df); grpd_df.to_csv('grpd_df.csv')
```

## Evaluation signals

- grpd_df.csv exists as a CSV table with samples as rows and time points as columns
- Per-sample time-domain traces in grpd_df are shortened by exactly the number of leading points specified in the acqus-derived group-delay parameter
- No complex-valued transformation artifacts: remaining FID values match the original fids_df values offset by the deleted leading points
- Downstream steps (px.solvent_residuals_removal, px.apodization) accept grpd_df without shape errors
- script_runs and file_exists checks pass on the grpd_df.csv artifact

## Limitations

- Only corrects the first-order phase error from the Bruker digital-filter group delay; residual zero/first-order phase is handled later by px.phase_correction on the spectra
- Requires the group-delay parameter to be present and correct in acqus_df from Bruker acqus DSP settings; a wrong value deletes the wrong number of points
- The deletion is irreversible point truncation — genuine early-time FID signal within the deleted region is lost

## Evidence

- [methods] group_delay_removal deletes leading FID points per acqus group delay: "grpd_df = px.group_delay_removal(fid_df=fids_df, acqus_df=acqus_df)"
- [methods] Extraction step precedes group-delay correction: "fids_df = px.extract_fids(root_directory=main_directory, acqus_df=acqus_df)"
- [readme] Protomix lists Group Delay Removal as a supported preprocessing feature: "- **Group Delay Removal**"
- [methods] Acquisition parameters are extracted from Bruker data: "acqus_df = px.extract_params(root_directory=main_directory)"
