---
name: nmr-fid-visualization
description: Use when raw Bruker 'fid' files have just been extracted into a pandas
  DataFrame (one row per sample) and you need a sanity check that FID traces loaded
  correctly before group delay removal or Fourier transform.
when_to_use_negative:
- The data are already frequency-domain spectra (spectra_df after fourier_transform)
  — inspect those with spectral plots, not FID plots
- The goal is batch-level QC of the processed spectrum matrix (e.g. after baseline_correction
  or binning); FID visualization only validates raw time-domain extraction per sample
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: Protomix
  role: Provides extract_fids to build the FID DataFrame and the interactive visualization
    utilities (px.plot) used to plot the raw FID real part
  repo: https://github.com/mzniber/protomix
- name: plotly
  role: Underlying interactive plotting library used by Protomix's visualization functions
- name: matplotlib
  role: Plotting backend imported in Protomix methods for signal visualization
- name: pandas
  role: Holds the sample-indexed FID and acqus tables that are sliced and plotted
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:b06a6d9602841d04937103d4351b1825eb71d1b83af50a6447b36131858b6dd4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - Python
  - matplotlib
  - numpy
  - pandas
  - plotly
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# nmr-fid-visualization

## Summary

Visualize raw Free Induction Decay (FID) traces extracted from Bruker 1H-NMR data with Protomix's interactive plotting (plotly-based px.plot) to verify that extraction produced intact, sample-indexed FID series before downstream preprocessing.

## When to use

Use when raw Bruker 'fid' files have just been extracted into a pandas DataFrame (one row per sample) and you need a sanity check that FID traces loaded correctly before group delay removal or Fourier transform.

## When NOT to use

- The data are already frequency-domain spectra (spectra_df after fourier_transform) — inspect those with spectral plots, not FID plots
- The goal is batch-level QC of the processed spectrum matrix (e.g. after baseline_correction or binning); FID visualization only validates raw time-domain extraction per sample

## Inputs

- fids_df: pandas DataFrame of raw FID traces (one row per sample) produced by px.extract_fids
- acqus_df: pandas DataFrame of Bruker 'acqus' acquisition parameters used by extract_fids
- spectrIndex: integer row index of the sample FID to inspect

## Outputs

- Interactive plot of the real part of one raw FID trace (time-domain signal) for a selected sample
- Qualitative verification that the FID DataFrame is populated and sample-indexed

## How to apply

After building the FID DataFrame with extract_fids(root_directory=main_directory, acqus_df=acqus_df), select one sample row (e.g. fids_df.iloc[spectrIndex]) and plot the real part of its raw FID signal using Protomix's plotting utility (px.plot), which builds on plotly/matplotlib interactive visualization. Inspect that the trace shows the expected exponentially decaying oscillation of a time-domain 1H-NMR signal rather than zeros, NaNs, or a truncated series. This check is a quality gate between data extraction and preprocessing: if the FID looks wrong, revisit the root_directory layout (original Bruker directory format with per-sample 'acqus' and 'fid' files) or the acqus_df passed to extract_fids, since Protomix uses the parameter table to interpret each sample's FID. Repeat the spot-check on a few samples to confirm all rows populated.

## Related tools

- **Protomix** (Provides extract_fids to build the FID DataFrame and the interactive visualization utilities (px.plot) used to plot the raw FID real part) — https://github.com/mzniber/protomix
- **plotly** (Underlying interactive plotting library used by Protomix's visualization functions)
- **matplotlib** (Plotting backend imported in Protomix methods for signal visualization)
- **pandas** (Holds the sample-indexed FID and acqus tables that are sliced and plotted)

## Examples

```
fids_df = px.extract_fids(root_directory=main_directory, acqus_df=acqus_df); px.plot(fids_df.iloc[spectrIndex].real)
```

## Evaluation signals

- The plotted FID shows a decaying oscillatory time-domain signal, not a flat line, zeros, or NaN values
- fids_df has one row per sample directory in the root_directory (matching acqus_df indexing)
- The selected row fids_df.iloc[spectrIndex] is a numeric vector of complex FID points whose real part plots without errors
- Re-running extract_fids with a correct acqus_df yields a consistent trace for the same sample (reproducibility check)
- Downstream step group_delay_removal(fid_df=fids_df, acqus_df=acqus_df) runs without shape or index errors, indicating the FID table was well formed

## Limitations

- Visualization is a spot-check on individual samples (e.g. fids_df.iloc[spectrIndex]); it does not batch-validate every FID in the dataset
- Protomix expects the original Bruker directory layout with per-sample 'fid' and 'acqus' files; an incorrect root_directory silently yields empty or malformed FID rows
- extract_fids depends on the acqus parameter table to interpret each FID, so a wrong or missing acqus_df invalidates the trace before plotting

## Evidence

- [methods] Extract Free Induction Decays (FID): "fids_df = px.extract_fids(root_directory=main_directory, acqus_df=acqus_df)"
- [abstract] Interactive visualization functionality: "It encompasses a suite of functionalities for data extraction, preprocessing, and interactive visualization."
- [readme] Protomix is a comprehensive Python package designed for preprocessing Nuclear Magnetic Resonance (NMR) data: "Protomix is a comprehensive Python package designed for preprocessing Nuclear Magnetic Resonance (NMR) data. Whether you're an academic researcher, a student, or an industry professional, Protomix"
- [methods] Plotting libraries in Protomix methods: [evidence span withheld — anchor status: not_found]
