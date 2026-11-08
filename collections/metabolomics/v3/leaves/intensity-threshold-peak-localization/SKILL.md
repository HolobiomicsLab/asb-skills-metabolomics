---
name: intensity-threshold-peak-localization
description: Use when you need to locate peak clusters in simulated (or preprocessed)
  1D 1H, 2D J-Res, or 2D COSY 1H NMR spectra before further modeling, using only a
  user-supplied intensity cutoff.
when_to_use_negative:
- Real experimental spectra with heavy noise, baseline drift, or broad protein background
  — a simple intensity threshold is designed for sparse, clean simulated pure-compound
  spectra
- Spectra where you need deconvolution of heavily overlapped multiplets rather than
  simple above-threshold cluster marking
- Workflows requiring automated threshold selection — the cutoff must be supplied
  by the user
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: MetAssimulo 2.0
  role: Provides the simulated 1D/J-Res/COSY spectra and the peak-detection module
    being reconstructed; also the reference implementation
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: Python
  role: Implementation language for reimplementing the threshold-based peak localization
    and cluster filtering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:d15bca22e6f88f6b8cc05462b4305eadbc66ba794ff7db6cbc239edd02afaadd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yanyan5420/MetAssimulo_2.git
  tools:
  - MetAssimulo 2.0
  - Python
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# intensity-threshold-peak-localization

## Summary

Peak-detection method from MetAssimulo 2.0 that locates peaks in simulated 1D 1H, 2D J-Res, and 2D COSY NMR spectra by marking local maxima above a user-defined intensity threshold and grouping adjacent above-threshold points into clusters, with J-Res and COSY first reduced to 1D profiles via skyline (F2) or diagonal projection.

## When to use

Use when you need to locate peak clusters in simulated (or preprocessed) 1D 1H, 2D J-Res, or 2D COSY 1H NMR spectra before further modeling, using only a user-supplied intensity cutoff.

## When NOT to use

- Real experimental spectra with heavy noise, baseline drift, or broad protein background — a simple intensity threshold is designed for sparse, clean simulated pure-compound spectra
- Spectra where you need deconvolution of heavily overlapped multiplets rather than simple above-threshold cluster marking
- Workflows requiring automated threshold selection — the cutoff must be supplied by the user

## Inputs

- Simulated 1D 1H spectrum intensity array (from MetAssimulo 2.0 Output)
- Simulated 2D J-Res spectrum matrix
- Simulated 2D COSY spectrum matrix
- User-defined intensity threshold (from Input/parameters.txt)

## Outputs

- Set of detected peak locations / local maxima above threshold
- Clustered peak regions per spectrum (1D, J-Res, COSY) with discarded clusters removed
- 1D projected profile for J-Res (skyline F2) and COSY (diagonal) spectra feeding the shared localization routine

## How to apply

Feed the simulated spectrum arrays (1D 1H from the Input data, or a 2D J-Res / COSY matrix) into a shared cluster-localization entry point. For 1D 1H spectra, scan the intensity profile and mark local maxima whose intensity exceeds the user-defined threshold specified in Input/parameters.txt, then group adjacent above-threshold points into clusters and discard clusters that fail filtering criteria. For 2D J-Res spectra, first collapse the matrix along F2 via a skyline projection; for COSY, extract the diagonal projection — each produces a 1D profile that is passed through the same threshold-and-cluster algorithm. The rationale is that pure-compound simulated spectra have sparse, well-separated peaks, so a single intensity cutoff suffices to localize them before mixture-spectrum simulation, avoiding heavier statistical denoising.

## Related tools

- **MetAssimulo 2.0** (Provides the simulated 1D/J-Res/COSY spectra and the peak-detection module being reconstructed; also the reference implementation) — https://github.com/yanyan5420/MetAssimulo_2.git
- **Python** (Implementation language for reimplementing the threshold-based peak localization and cluster filtering)

## Examples

```
python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- Run the module end-to-end on a simulated spectrum from the Input data and confirm it completes without error
- Manually inspect the simulated spectrum at the user-defined intensity threshold and verify detected clusters match visually identifiable peaks
- Confirm J-Res skyline (F2) and COSY diagonal projections each produce a valid 1D profile that feeds the same cluster-localization routine
- Verify clusters span contiguous above-threshold points and that clusters failing filtering criteria are discarded

## Limitations

- Threshold is user-defined and must be manually tuned per spectrum — no automatic cutoff selection is described
- The method is intended for pure-compound simulated spectra; the article does not validate it on real noisy spectra
- For 2D spectra only skyline (F2) and diagonal projections are used, discarding off-diagonal 2D information for J-Res and non-diagonal COSY cross-peaks

## Evidence

- [methods] peaks are located based on a user-defined intensity threshold: [evidence span withheld — anchor status: not_found]
- [methods] skyline projection collapses the 2D J-Res matrix to a 1D profile fed into the same cluster-localization algorithm: [evidence span withheld — anchor status: not_found]
- [readme] README confirms the pipeline runs on simulated spectra with a user-editable parameters file: "python3 apps/index.py -p Input/parameters.txt"
