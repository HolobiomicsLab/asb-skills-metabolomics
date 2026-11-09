---
name: peak-area-quantification
description: Use when you have a referenced 2D 13C INADEQUATE spectrum and need to
  detect peaks and quantify their areas with a multi-threshold sweep, before network
  filtering and metabolite matching.
when_to_use_negative:
- Input is already a picked peak list — this skill covers threshold-sweep picking;
  use PyINETA's cluster/find/match steps instead for network filtering and database
  matching.
- Query spectra are not 2D INADEQUATE with a direct and double-quantum dimension —
  the PPCS/PPDQ center-of-mass merging assumes that dimensional structure.
- Raw input files exceed 100 Mb (as in the README's example2), which the repository
  does not ship — you must contact the Edison lab for those raw files rather than
  expecting local example data.
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: PyINETA
  role: Python package performing the slice-wise peak picking, center-of-mass peak
    merging, and picked_peaks.csv output
  repo: https://github.com/edisonomics/PyINETA
- name: Python
  role: Runtime for run_pyineta.py with the pick / pick+ step of the INETA pipeline
- name: NMRbox
  role: Hosted environment where PyINETA is installed and publicly available for running
    the picking pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:01fd3d2744a8d44e6b1980e6eba0963d0fe68e8b346fedaf86af13fc917302db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edisonomics/PyINETA
  tools:
  - NMRbox
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# peak-area-quantification

## Summary

Slice-wise peak picking for 2D INADEQUATE NMR spectra in PyINETA: sweeps an intensity threshold from PPmax down to PPmin to define peaks and their areas, then merges nearby picks via center-of-mass tolerances (PPCS, PPDQ) into a picked_peaks.csv for downstream network filtering.

## When to use

Use when you have a referenced 2D 13C INADEQUATE spectrum and need to detect peaks and quantify their areas with a multi-threshold sweep, before network filtering and metabolite matching.

## When NOT to use

- Input is already a picked peak list — this skill covers threshold-sweep picking; use PyINETA's cluster/find/match steps instead for network filtering and database matching.
- Query spectra are not 2D INADEQUATE with a direct and double-quantum dimension — the PPCS/PPDQ center-of-mass merging assumes that dimensional structure.
- Raw input files exceed 100 Mb (as in the README's example2), which the repository does not ship — you must contact the Edison lab for those raw files rather than expecting local example data.

## Inputs

- INADEQUATE NMR spectrum data file (referenced via basic shifting)
- config.ini with picking parameters: PPmax, PPmin, steps, PPCS, PPDQ

## Outputs

- picked_peaks.csv (chemical shift coordinates and peak areas per merged peak)

## How to apply

Load the INADEQUATE spectrum and reference its chemical shifts with PyINETA's basic shifting, then set picking parameters in config.ini: maximum picking level (PPmax), minimum picking level (PPmin), and the number of sweep slices ('steps'). Sweep the picking level from PPmax down to PPmin in 'steps' slices; at each slice, identify local maxima to define candidate peaks and their peak areas — this multi-slice approach resolves peaks a single intensity threshold cannot separate. Merge picks whose centers of mass fall within PPCS along the direct dimension and PPDQ along the double-quantum dimension into a single peak. Write the merged peaks (chemical shift coordinates and peak areas) to picked_peaks.csv, which feeds PyINETA's network filtering and simulated INADEQUATE database matching steps.

## Related tools

- **PyINETA** (Python package performing the slice-wise peak picking, center-of-mass peak merging, and picked_peaks.csv output) — https://github.com/edisonomics/PyINETA
- **Python** (Runtime for run_pyineta.py with the pick / pick+ step of the INETA pipeline)
- **NMRbox** (Hosted environment where PyINETA is installed and publicly available for running the picking pipeline)

## Examples

```
python <path_to_pyineta_repo>/run_pyineta.py -c config.ini -s pick+ -o example1_output
```

## Evaluation signals

- picked_peaks.csv is produced with peak entries whose positions fall within the spectrum's chemical-shift range
- the slice sweep covered [PPmin, PPmax] in exactly 'steps' increments
- peak positions and areas in example1 output match the provided example1_default_output folder (README: 'You can cross-check the results with the provided output in folder example1_default_output')
- merged peak count is reduced relative to unmerged slice-level picks, consistent with PPCS/PPDQ center-of-mass merging
- downstream network filtering runs without missing picks, indicating the pick+ chain completed before cluster/find/match

## Limitations

- Picking quality depends on tuning PPmax, PPmin, 'steps', PPCS, and PPDQ; a single intensity threshold cannot separate nearby peaks, so insufficient sweep depth merges or misses them.
- Large raw INADEQUATE files (README example2, inputs >100 Mb) are not distributed with the repository and must be requested from the Edison lab.
- The article's discussion states no explicit limitations or reproducibility gaps, so failure modes beyond parameter sensitivity are not documented.

## Evidence

- [intro] pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking.: "pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking."
- [intro] It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites: "It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites to identify"
- [readme] Optional: Specify which steps you want to run. Can be one of {all,load,pick,cluster,find,match,plot,summary,singleplot,load+,pick+,cluster+,find+,match+}.: "Optional: Specify which steps you want to run. Can be
                        one of {all,load,pick,cluster,find,match,plot,summary,"
- [readme] You can cross-check the results with the provided output in folder example1_default_output.: [evidence span withheld — anchor status: external_doc]
