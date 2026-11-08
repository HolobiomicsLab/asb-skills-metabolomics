---
name: inadequate-spectral-peak-picking
description: Use when you have an INADEQUATE spectrum (with chemical-shift referencing)
  and need to extract peak positions and areas where a single intensity threshold
  cannot separate overlapping 13C-13C correlations.
when_to_use_negative:
- Input is already a picked-peak table or a filtered peak network — peak picking should
  be skipped (use -s steps after 'pick', e.g. cluster+ or find+)
- The spectrum is a 13C-JRES or other non-INADEQUATE experiment; this skill targets
  INADEQUATE spectra specifically
- Raw data files exceed available resources and the Edison lab must be contacted first
  (as noted for example2)
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: PyINETA
  role: Python package that reads and references INADEQUATE spectra (basic shifting)
    and performs slice-wise peak picking with PPmax/PPmin/steps and PPCS/PPDQ merge
    tolerances
  repo: https://github.com/edisonomics/PyINETA
- name: NMRbox
  role: Hosted environment where PyINETA is installed and publicly available for running
    the picking pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:f82e659f40e23e344287361153098a6f004f47495c52a88f912d27c26f0531df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edisonomics/PyINETA
  tools:
  - NMRbox
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# inadequate-spectral-peak-picking

## Summary

Slice-wise peak picking of 2D INADEQUATE NMR spectra using PyINETA, which sweeps picking levels from PPmax to PPmin across multiple threshold slices and merges nearby peaks by center-of-mass, producing picked_peaks.csv for downstream network filtering.

## When to use

Use when you have an INADEQUATE spectrum (with chemical-shift referencing) and need to extract peak positions and areas where a single intensity threshold cannot separate overlapping 13C-13C correlations.

## When NOT to use

- Input is already a picked-peak table or a filtered peak network — peak picking should be skipped (use -s steps after 'pick', e.g. cluster+ or find+)
- The spectrum is a 13C-JRES or other non-INADEQUATE experiment; this skill targets INADEQUATE spectra specifically
- Raw data files exceed available resources and the Edison lab must be contacted first (as noted for example2)

## Inputs

- INADEQUATE NMR spectrum data file (2D 13C-13C correlation spectrum)
- Config file (config.ini) with picking parameters: PPmax, PPmin, steps, PPCS, PPDQ

## Outputs

- picked_peaks.csv with chemical shift coordinates and peak areas per merged peak
- Pick output folder from run_pyineta.py (e.g. -o outdir)

## How to apply

Load the INADEQUATE spectrum and reference its chemical shifts using PyINETA's basic shifting. Configure the picking module with a maximum picking level (PPmax), minimum picking level (PPmin), and the number of sweep slices ('steps'). Sweep the picking level from PPmax down to PPmin in 'steps' increments; within each slice identify local maxima and define an area for each candidate peak. Merge nearby picked peaks into a single peak when their centers of mass fall within PPCS tolerance along the direct dimension and PPDQ tolerance along the double-quantum dimension. Run the pipeline with the picking step (e.g. -s pick or pick+) and inspect picked_peaks.csv for chemical-shift coordinates and peak areas feeding the network filtering step.

## Related tools

- **PyINETA** (Python package that reads and references INADEQUATE spectra (basic shifting) and performs slice-wise peak picking with PPmax/PPmin/steps and PPCS/PPDQ merge tolerances) — https://github.com/edisonomics/PyINETA
- **NMRbox** (Hosted environment where PyINETA is installed and publicly available for running the picking pipeline)

## Examples

```
cd example/example1/ && python ../../run_pyineta.py -c config.ini -s pick -o example1_output
```

## Evaluation signals

- picked_peaks.csv is produced and each picked peak's chemical shifts fall within the spectrum's chemical-shift range
- The picking-level sweep covered [PPmin, PPmax] in exactly 'steps' slices
- Overlapping candidate peaks whose centers of mass are within PPCS (direct dimension) and PPDQ (DQ dimension) were merged into single peaks
- Re-running the repository example (example1) reproduces the reference outputs in example1_default_output

## Limitations

- Single-intensity-threshold picking cannot separate nearby INADEQUATE peaks; the multi-slice sweep and PPCS/PPDQ merge tolerances must be tuned to avoid over- or under-merging
- PyINETA outputs must be validated as part of the pipeline; the article notes no explicit limitations or reproducibility gaps in the discussion
- Repository example2 inputs/temporary files exceed 100 MB and require contacting the Edison lab

## Evidence

- [methods] PyINETA's peak-picking begins at the highest intensity threshold (PPmax), collects data points at successive slices until reaching the minimum threshold (PPmin): [evidence span withheld — anchor status: not_found]
- [methods] merges peaks whose centers of mass fall within PPCS along the direct dimension and PPDQ along the double-quantum dimension into a single peak: [evidence span withheld — anchor status: not_found]
- [intro] pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking: "pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking."
- [readme] It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound: "It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database"
