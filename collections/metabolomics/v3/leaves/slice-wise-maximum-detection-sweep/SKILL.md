---
name: slice-wise-maximum-detection-sweep
description: Use when peak-picking a 13C INADEQUATE spectrum where overlapping or
  weak peaks are missed by a single intensity cutoff, and you need slice-wise threshold
  sweeps with merge tolerances.
when_to_use_negative:
- The input is not an INADEQUATE spectrum (e.g. 1D 13C or 13C-JRES data) — this sweep
  is designed for 2D INADEQUATE peak picking.
- A simple single-threshold picking already separates all peaks cleanly and no merge
  step is needed.
- You only need network filtering or database matching — those downstream steps consume
  picked_peaks.csv but do not require re-picking.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: PyINETA
  role: Python package whose picking module performs the slice-wise threshold sweep,
    local-maximum detection, and center-of-mass peak merging
  repo: https://github.com/edisonomics/PyINETA
- name: Python
  role: Runtime for run_pyineta.py, invoked with the config file and the 'pick' or
    'all' steps
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:d806e8b407aebebbd6ce982d0d96919da13651e070b530cb097cf5aa208f9776
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edisonomics/PyINETA
  tools:
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# slice-wise-maximum-detection-sweep

## Summary

PyINETA's peak-picking algorithm for 2D INADEQUATE spectra: sweep a picking threshold from PPmax down to PPmin in 'steps' slices, define peaks from local maxima per slice, then merge nearby picks via center-of-mass tolerances (PPCS, PPDQ). It resolves peaks a single fixed threshold cannot separate.

## When to use

Use when peak-picking a 13C INADEQUATE spectrum where overlapping or weak peaks are missed by a single intensity cutoff, and you need slice-wise threshold sweeps with merge tolerances.

## When NOT to use

- The input is not an INADEQUATE spectrum (e.g. 1D 13C or 13C-JRES data) — this sweep is designed for 2D INADEQUATE peak picking.
- A simple single-threshold picking already separates all peaks cleanly and no merge step is needed.
- You only need network filtering or database matching — those downstream steps consume picked_peaks.csv but do not require re-picking.

## Inputs

- INADEQUATE spectrum data file (referenced via basic shifting)
- PyINETA config file (config.ini) with picking parameters: PPmax, PPmin, steps, PPCS, PPDQ

## Outputs

- picked_peaks.csv with merged peak chemical shift coordinates and peak areas

## How to apply

Load and reference the INADEQUATE spectrum using PyINETA's picking module (referencing via basic shifting), then set the maximum picking level (PPmax), minimum picking level (PPmin), and the number of sweep slices ('steps') in the config file. Sweep the picking level from PPmax down to PPmin in 'steps' increments; at each slice, find local maxima and define an area for each candidate peak, so peaks with different intensities are captured across multiple thresholds. Merge overlapping/nearby candidate peaks into single peaks using center-of-mass calculations with the PPCS tolerance along the direct dimension and the PPDQ tolerance along the double-quantum dimension. Write the merged picks (chemical shift coordinates, peak areas) to picked_peaks.csv for downstream network filtering and metabolite matching.

## Related tools

- **PyINETA** (Python package whose picking module performs the slice-wise threshold sweep, local-maximum detection, and center-of-mass peak merging) — https://github.com/edisonomics/PyINETA
- **Python** (Runtime for run_pyineta.py, invoked with the config file and the 'pick' or 'all' steps)

## Examples

```
python run_pyineta.py -c config.ini -s pick -o example1_output
```

## Evaluation signals

- picked_peaks.csv is produced and non-empty
- All picked peak chemical shift positions fall within the spectrum's chemical-shift range
- The threshold sweep covered [PPmin, PPmax] in exactly 'steps' increments
- Peaks whose centers of mass fall within PPCS (direct dimension) and PPDQ (double-quantum dimension) were merged into single entries, not duplicated
- Downstream network filtering steps accept picked_peaks.csv without coordinate errors

## Limitations

- Sweep parameters (PPmax, PPmin, steps, PPCS, PPDQ) must be tuned per spectrum; too-large tolerances over-merge distinct nearby peaks
- Example2 raw input/temporary files exceed 100 MB and are not distributed in the repo — users must contact the Edison lab for raw files
- Peak picking is only the first stage; picks must still be filtered into networks and matched to a simulated INADEQUATE metabolite database

## Evidence

- [methods] Slice sweep and local maxima: [evidence span withheld — anchor status: not_found]
- [methods] Merging by center of mass: [evidence span withheld — anchor status: not_found]
- [intro] Peak picking in the package scope: "pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking."
- [readme] Downstream use of picks: "It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites"
- [readme] CLI invocation of the pipeline: [evidence span withheld — anchor status: external_doc]
