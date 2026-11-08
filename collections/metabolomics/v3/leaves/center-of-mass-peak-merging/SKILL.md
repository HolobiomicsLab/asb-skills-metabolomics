---
name: center-of-mass-peak-merging
description: Use when peak-picking a 2D 13C INADEQUATE NMR spectrum in PyINETA where
  a single intensity threshold cannot separate overlapping cross-peaks, and nearby
  picked peaks must be merged before network filtering.
when_to_use_negative:
- The data are not INADEQUATE spectra (e.g. 1D 1H or 13C-JRES data) — PPCS/PPDQ tolerances
  are defined for direct and double-quantum dimensions specific to INADEQUATE.
- A single intensity threshold already cleanly separates all peaks and no merging
  across slices is needed.
- The goal is the downstream network-clustering or metabolite database-matching step
  rather than picking/merging itself.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: PyINETA
  role: Python package that performs slice-wise peak picking of INADEQUATE spectra
    and merges nearby picked peaks by center-of-mass (PPCS/PPDQ) tolerances before
    writing picked_peaks.csv
  repo: https://github.com/rhilt13/PyINETA.git
- name: Python
  role: runtime for the run_pyineta.py pipeline driver invoked with a config file
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:e350333f1e03b6bbe4cb7f4eae3288077a04c7d58f73883442c99fa2d06e9db0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhilt13/PyINETA.git
  tools:
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# center-of-mass-peak-merging

## Summary

PyINETA's slice-wise INADEQUATE peak-picking/merging strategy: sweep the picking level from PPmax down to PPmin in 'steps' slices, define candidate peaks per slice, then merge nearby peaks whose centers of mass fall within PPCS (direct dimension) and PPDQ (double-quantum dimension) tolerances.

## When to use

Use when peak-picking a 2D 13C INADEQUATE NMR spectrum in PyINETA where a single intensity threshold cannot separate overlapping cross-peaks, and nearby picked peaks must be merged before network filtering.

## When NOT to use

- The data are not INADEQUATE spectra (e.g. 1D 1H or 13C-JRES data) — PPCS/PPDQ tolerances are defined for direct and double-quantum dimensions specific to INADEQUATE.
- A single intensity threshold already cleanly separates all peaks and no merging across slices is needed.
- The goal is the downstream network-clustering or metabolite database-matching step rather than picking/merging itself.

## Inputs

- INADEQUATE NMR spectrum data file (read and referenced via basic shifting)
- PyINETA config.ini with picking parameters: PPmax, PPmin, steps, PPCS, PPDQ

## Outputs

- picked_peaks.csv containing merged peak chemical-shift coordinates and peak areas
- picked-peak list feeding PyINETA network filtering and simulated-INADEQUATE database matching

## How to apply

Load and reference the INADEQUATE spectrum (basic shifting), then set picking parameters: maximum picking level (PPmax), minimum picking level (PPmin), and the number of sweep slices ('steps'). Sweep the picking level from PPmax down to PPmin in 'steps' slices, collecting data points at each slice and identifying local maxima to define candidate peaks and their peak areas. Merge peaks whose centers of mass fall within PPCS along the direct dimension and PPDQ along the double-quantum dimension, collapsing overlapping/nearby candidates into a single peak per feature. Write the merged peaks (chemical shift coordinates, peak areas) to picked_peaks.csv for the downstream network-filtering and database-matching steps. Rationale: successive threshold slices recover peaks of widely varying intensity that a single threshold would miss, while center-of-mass distance checks prevent the same physical peak from being counted multiple times across slices.

## Related tools

- **PyINETA** (Python package that performs slice-wise peak picking of INADEQUATE spectra and merges nearby picked peaks by center-of-mass (PPCS/PPDQ) tolerances before writing picked_peaks.csv) — https://github.com/rhilt13/PyINETA.git
- **Python** (runtime for the run_pyineta.py pipeline driver invoked with a config file)

## Examples

```
python run_pyineta.py -c config.ini -s pick -o example1_output
```

## Evaluation signals

- picked_peaks.csv is produced with peak entries whose positions fall within the spectrum's chemical-shift range.
- The threshold sweep covered [PPmin, PPmax] in exactly 'steps' increments.
- No two retained peaks have centers of mass closer than the PPCS (direct dimension) / PPDQ (double-quantum dimension) tolerances.
- Results can be cross-checked against the provided example1_default_output folder in the repository.
- Downstream filtering step still yields networks of peaks attributable to the same compound (i.e., merging did not collapse distinct cross-peaks).

## Limitations

- Merging depends on the PPCS and PPDQ tolerance settings; overly permissive tolerances can merge distinct neighboring peaks, while overly tight tolerances leave slice duplicates unmerged.
- Example2 input and temporary files exceed 100 Mb, so the raw files must be requested from the Edison lab; large spectra may be impractical to run locally without them.
- The article's discussion states no explicit limitations or reproducibility gaps, so failure modes must be probed empirically (e.g., by varying steps/PPmin).

## Evidence

- [methods] PyINETA's peak-picking begins at the highest intensity threshold (PPmax), collects data points at successive slices until reaching the minimum threshold (PPmin), finds the local maximum and defines an area for each peak per slice, then merges peaks whose centers of mass fall within PPCS along the direct dimension and PPDQ along the double-quantum dimension into a single peak.: [evidence span withheld — anchor status: not_found]
- [intro] pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking. It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound: "pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking."
- [readme] Optional: Specify which steps you want to run. Can be one of {all,load,pick,cluster,find,match,plot,summary,singleplot,load+,pick+,cluster+,find+,match+}.: [evidence span withheld — anchor status: not_found]
