---
name: bruker-acquisition-parameter-parsing
description: Use when raw Bruker 1D 1H-NMR acquisition directories are the input and
  FID-level preprocessing steps (group delay removal, zero filling, Fourier transform)
  need per-sample instrument parameters.
when_to_use_negative:
- Input data is not in Bruker format (e.g. vendor-neutral spectra tables) — extract_params
  expects Bruker acquisition directories
- FIDs have already been extracted and corrected; then work with fids_df/grpd_df directly
  instead of re-parsing acqus files
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: Protomix
  role: Provides extract_params to parse Bruker acqus acquisition parameters into
    acqus_df, and the dependent functions extract_fids, group_delay_removal, zero_filling,
    fourier_transform that read those parameters
  repo: https://github.com/mzniber/protomix
- name: pandas
  role: Holds and manipulates the acqus_df parameter table and the FID/parameter DataFrames
    passed between Protomix steps
- name: numpy
  role: Numeric backend used by Protomix when loading and processing FID data referenced
    against the parsed parameters
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:e6b69f901159466261295fe4edd0bfd8cbbbfb4a7e1eb9ea5ab0a931f01416ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# bruker-acquisition-parameter-parsing

## Summary

Extract Bruker acquisition parameters (acqus settings such as the DSP group delay and zero-filling targets) into a pandas DataFrame, which downstream Protomix preprocessing steps consume to correctly shape and correct each sample's FID.

## When to use

Use when raw Bruker 1D 1H-NMR acquisition directories are the input and FID-level preprocessing steps (group delay removal, zero filling, Fourier transform) need per-sample instrument parameters.

## When NOT to use

- Input data is not in Bruker format (e.g. vendor-neutral spectra tables) — extract_params expects Bruker acquisition directories
- FIDs have already been extracted and corrected; then work with fids_df/grpd_df directly instead of re-parsing acqus files

## Inputs

- Bruker NMR raw data root directory containing per-sample acquisition folders with acqus parameter files
- pandas/numpy Python environment with Protomix installed via pip

## Outputs

- acqus_df: a pandas DataFrame of extracted acquisition parameters (one row per sample) including the group-delay parameter
- A parameter table that downstream steps (extract_fids, group_delay_removal, zero_filling, fourier_transform) consume

## How to apply

Run Protomix's extract_params over the root directory containing the Bruker datasets to build acqus_df, a pandas DataFrame keyed by sample with acquisition parameters derived from the acqus files (e.g. the DSP group delay, 'death time'). This table must be produced before extract_fids, since extract_fids requires acqus_df to read each FID correctly. Downstream, group_delay_removal reads the group-delay value per sample from acqus_df and deletes exactly that many leading points from each FID, and zero_filling uses acqus_df (with an explicit target_points, e.g. 10000) to size the padded FIDs. Persist acqus_df alongside fids_df so that each parameter-driven step can be re-run and validated against the original instrument settings.

## Related tools

- **Protomix** (Provides extract_params to parse Bruker acqus acquisition parameters into acqus_df, and the dependent functions extract_fids, group_delay_removal, zero_filling, fourier_transform that read those parameters) — https://github.com/mzniber/protomix
- **pandas** (Holds and manipulates the acqus_df parameter table and the FID/parameter DataFrames passed between Protomix steps)
- **numpy** (Numeric backend used by Protomix when loading and processing FID data referenced against the parsed parameters)

## Examples

```
import protomix as px; acqus_df = px.extract_params(root_directory=main_directory)
```

## Evaluation signals

- acqus_df exists as a pandas DataFrame with one entry per sample directory and carries the group-delay parameter per sample
- Downstream artifact grpd_df.csv exists and each sample's time-domain trace is shortened by exactly the acqus-specified number of leading points
- extract_fids succeeds only when acqus_df is passed (px.extract_fids(root_directory=..., acqus_df=acqus_df)), confirming the parameter table is wired correctly
- zero_filling output zf_df has the target number of points (e.g. 10000) consistent with acqus_df-driven sizing

## Limitations

- Parameter parsing is specific to Bruker acqus files; the DSP-derived group delay ('death time') artifact is what motivates the extraction, and other vendors' parameter layouts are not covered
- The article does not document a changelog, so version-specific parameter-handling behavior may change without notice

## Evidence

- [methods] extract_params producing acqus_df from Bruker root directory: "acqus_df = px.extract_params(root_directory=main_directory)"
- [methods] extract_fids depends on the acqus parameter table: "fids_df = px.extract_fids(root_directory=main_directory, acqus_df=acqus_df)"
- [methods] group delay read from acqus drives leading-point deletion: "grpd_df = px.group_delay_removal(fid_df=fids_df, acqus_df=acqus_df)"
- [methods] zero filling uses acqus parameters with explicit target: "zf_df = px.zero_filling(fid_df=apod_df, acqus_df=acqus_df, target_points=10000)"
- [readme] Protomix is a comprehensive NMR preprocessing package: "Protomix is a comprehensive Python package designed for preprocessing Nuclear Magnetic Resonance (NMR) data"
