---
name: nmr-fid-group-delay-correction
description: Use when raw Bruker FIDs (time-domain, complex) carry a DSP digital-filter
  group delay recorded in the acqus parameters and you need phase-corrected FIDs prior
  to apodization, zero filling, and Fourier transform.
when_to_use_negative:
- Input data is already frequency-domain spectra (post fourier_transform) — group
  delay is a time-domain point-offset artifact and must be corrected on the FID before
  Fourier transform.
- The Bruker dataset lacks the acqus DSP group-delay parameter (acqus_df has no group-delay
  entry), in which case there is no defined number of leading points to delete.
- FIDs were acquired on a non-Bruker instrument without a digital-filter group delay,
  so no first-order phase error from the 'death time' artifact exists.
tools:
- name: Protomix
  role: provides the group_delay_removal function that strips the acqus-specified
    number of leading points from each FID
  repo: https://github.com/mzniber/protomix
- name: pandas
  role: used to load and manipulate the fids_df and acqus_df DataFrames and save grpd_df
    as CSV
- name: numpy
  role: numerical handling of the complex FID arrays within the DataFrames
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:4bc8bafafde7d7e27fc152370c9a523d2b291948d86cf69419d8859e062f7e08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - numpy
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# nmr-fid-group-delay-correction

## Summary

Removes the Bruker digital-filter group-delay ('death time') artifact from each 1H-NMR FID by deleting exactly the acqus-specified number of leading data points, correcting the first-order phase error before further Protomix preprocessing.

## When to use

Use when raw Bruker FIDs (time-domain, complex) carry a DSP digital-filter group delay recorded in the acqus parameters and you need phase-corrected FIDs prior to apodization, zero filling, and Fourier transform.

## When NOT to use

- Input data is already frequency-domain spectra (post fourier_transform) — group delay is a time-domain point-offset artifact and must be corrected on the FID before Fourier transform.
- The Bruker dataset lacks the acqus DSP group-delay parameter (acqus_df has no group-delay entry), in which case there is no defined number of leading points to delete.
- FIDs were acquired on a non-Bruker instrument without a digital-filter group delay, so no first-order phase error from the 'death time' artifact exists.

## Inputs

- fids_df: pandas DataFrame of extracted Free Induction Decays (complex time-domain data, one row per sample), as produced by px.extract_fids(root_directory=main_directory, acqus_df=acqus_df)
- acqus_df: pandas DataFrame of Bruker acquisition parameters (including the DSP group-delay parameter), as produced by px.extract_params(root_directory=main_directory)

## Outputs

- grpd_df: pandas DataFrame of group-delay-corrected FIDs (samples as rows, time points as columns), each trace shortened by the acqus-specified number of leading points
- grpd_df.csv: saved CSV artifact of the corrected FID table

## How to apply

Load the extracted FID table (fids_df) and acquisition-parameter table (acqus_df) produced by Protomix's extract_params and extract_fids steps, using pandas. Read the group-delay parameter (derived from the Bruker acqus DSP settings) for each sample from acqus_df. Apply Protomix's group_delay_removal, which simply deletes exactly that many leading data points from each FID — no complex transformations are needed, since the Bruker group delay is a pure point-offset artifact. Store the result (grpd_df) as a DataFrame with samples as rows and time points as columns, so downstream steps (solvent_residuals_removal, apodization, zero_filling, fourier_transform) operate on delay-corrected FIDs.

## Related tools

- **Protomix** (provides the group_delay_removal function that strips the acqus-specified number of leading points from each FID) — https://github.com/mzniber/protomix
- **pandas** (used to load and manipulate the fids_df and acqus_df DataFrames and save grpd_df as CSV)
- **numpy** (numerical handling of the complex FID arrays within the DataFrames)

## Examples

```
grpd_df = px.group_delay_removal(fid_df=fids_df, acqus_df=acqus_df)
```

## Evaluation signals

- grpd_df.csv exists as a CSV artifact (file_exists check) after the step runs (script_runs check).
- Each per-sample FID trace in grpd_df has been shortened relative to fids_df by exactly the group-delay value read from acqus_df for that sample.
- grpd_df preserves the samples-as-rows, time-points-as-columns layout expected by downstream Protomix steps (solvent_residuals_removal, apodization, zero_filling).
- Values in each corrected FID are the original data points shifted — i.e. grpd_df.iloc[i] equals fids_df.iloc[i][group_delay:] — confirming no complex transformation was applied, only point deletion.
- Downstream fourier_transform output shows no first-order phase distortion attributable to the digital-filter delay.

## Limitations

- The correction deletes exactly the acqus-specified number of leading points; if the group-delay parameter in acqus_df is missing or wrong, the correction is correspondingly wrong.
- It is a Bruker-specific artifact remedy ('death time'); the step assumes the DSP group-delay convention and needs no complex transformation, so it should not be repurposed as a general phase-correction routine.
- Each FID is shortened by the number of deleted points, slightly reducing time-domain resolution before zero_filling (which can restore target_points, e.g. 10000).

## Evidence

- [methods] group_delay_removal call in pipeline: "grpd_df = px.group_delay_removal(fid_df=fids_df, acqus_df=acqus_df)"
- [methods] Protomix scope statement: "Protomix is a comprehensive Python package designed for preprocessing Nuclear Magnetic Resonance (NMR) data"
- [readme] Feature list including Group Delay Removal: [evidence span withheld — anchor status: external_doc]
- [abstract] Automated preprocessing rationale: "Protomix offers a set of automated, efficient, and user-friendly signal-preprocessing steps"
- [methods] pandas/numpy as the data-handling substrate: "import numpy as np
import pandas as pd"
