---
name: nmr-spectral-peak-detection
description: Use when the input is a simulated (e.g. MetAssimulo 2.0) or real 1D 1H,
  2D J-Res or COSY NMR spectrum and you need to identify peak positions/clusters before
  downstream analysis such as mixture spectrum simulation or metabolite quantification.
when_to_use_negative:
- The input is already a processed peak/feature table rather than a raw or simulated
  spectrum — peak detection would be redundant.
- The spectrum is not a 1H NMR 1D, J-Res or COSY spectrum (e.g. 13C, HSQC-TOCSY, or
  non-NMR data) since the projection and cluster-localization routines are specific
  to these spectral types.
- You need peak detection on an un-preprocessed spectrum where the user-defined intensity
  threshold is not meaningful (e.g. very noisy, un-normalized data without prior baseline
  correction).
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: MetAssimulo 2.0
  role: Provides the simulated 1D, J-Res and COSY spectra on which the threshold-based
    peak-detection module is exercised and validated.
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: Python
  role: Implementation language for the cluster-localization module, the skyline (F2)
    projection for J-Res and the diagonal projection for COSY.
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:052ab728aef96e0b9b6800d9325e4853416b51523dcd9de9306b80a767729277
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yanyan5420/MetAssimulo_2.git
  tools:
  - MetAssimulo 2.0
  - Python
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# nmr-spectral-peak-detection

## Summary

Locate and cluster peaks in simulated or real 1D 1H, 2D J-Res and 2D COSY metabolomic 1H NMR spectra by applying a user-defined intensity threshold and a shared cluster-localization algorithm, optionally preceded by a skyline (F2) or diagonal projection for 2D data.

## When to use

Use when the input is a simulated (e.g. MetAssimulo 2.0) or real 1D 1H, 2D J-Res or COSY NMR spectrum and you need to identify peak positions/clusters before downstream analysis such as mixture spectrum simulation or metabolite quantification.

## When NOT to use

- The input is already a processed peak/feature table rather than a raw or simulated spectrum — peak detection would be redundant.
- The spectrum is not a 1H NMR 1D, J-Res or COSY spectrum (e.g. 13C, HSQC-TOCSY, or non-NMR data) since the projection and cluster-localization routines are specific to these spectral types.
- You need peak detection on an un-preprocessed spectrum where the user-defined intensity threshold is not meaningful (e.g. very noisy, un-normalized data without prior baseline correction).

## Inputs

- 1D 1H NMR spectrum (simulated via MetAssimulo 2.0 or real experimental data)
- 2D J-Res NMR spectrum
- 2D COSY NMR spectrum
- User-defined intensity threshold

## Outputs

- Detected peak clusters (positions/intensities) for each spectrum
- Filtered cluster list after discarding clusters failing the filtering criteria

## How to apply

For a 1D 1H spectrum, scan the spectral intensity profile and mark local maxima whose intensity exceeds a user-defined threshold, then group adjacent above-threshold points into clusters and discard clusters that fail the filtering criteria. For 2D J-Res spectra, first collapse the 2D matrix along the F2 axis with a skyline projection to obtain a 1D profile, then feed it into the same cluster-localization routine; for COSY spectra, extract the diagonal signal as the projected profile instead. All three spectral types share a common cluster-localization entry point so that 1D, J-Res and COSY peak detection follow one pathway. Wrap steps 2-5 as a reusable module so the user-defined intensity threshold is the single tuning parameter controlling sensitivity across all spectral types.

## Related tools

- **MetAssimulo 2.0** (Provides the simulated 1D, J-Res and COSY spectra on which the threshold-based peak-detection module is exercised and validated.) — https://github.com/yanyan5420/MetAssimulo_2.git
- **Python** (Implementation language for the cluster-localization module, the skyline (F2) projection for J-Res and the diagonal projection for COSY.) — https://github.com/yanyan5420/MetAssimulo_2.git

## Examples

```
export PYTHONPATH=$HOME/MetAssimulo_2/:$PYTHONPATH; python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- The module runs end-to-end on a simulated spectrum produced by MetAssimulo 2.0 from the Input data without errors.
- Detected peak clusters, when manually inspected against the spectrum at the user-defined intensity threshold, match the visually apparent maxima/regions above threshold.
- The same cluster-localization entry point produces coherent clusters for all three spectral types (1D 1H, J-Res after skyline projection, COSY after diagonal projection).
- Changing the intensity threshold changes the number/extent of detected clusters in a monotonic, interpretable way.

## Limitations

- Peak detection is threshold-based, so sensitivity and specificity depend entirely on the user-defined intensity threshold; no adaptive or automatic threshold is described.
- For 2D spectra, information is reduced to a 1D projection (skyline F2 for J-Res, diagonal for COSY), so off-diagonal cross-peak structure is not localized by this module.
- The workflow is documented as part of MetAssimulo 2.0's simulation pipeline; applying it to arbitrary experimental spectra may require additional preprocessing not described in the article.

## Evidence

- [results] peak detection uses user-defined intensity threshold: [evidence span withheld — anchor status: not_found]
- [results] cluster-localization shared across spectral types: [evidence span withheld — anchor status: not_found]
- [abstract] MetAssimulo 2.0 code availability: "The code and the detailed instruction/tutorial for MetAssimulo 2.0 is available at https://github.com/yanyan5420/MetAssimulo_2.git"
- [readme] README run command: "export PYTHONPATH=$HOME/MetAssimulo_2/:$PYTHONPATH
python3 apps/index.py -p Input/parameters.txt"
- [abstract] simulation purpose: "In this context, simulation of spectral data becomes a tractable solution for algorithm development."
