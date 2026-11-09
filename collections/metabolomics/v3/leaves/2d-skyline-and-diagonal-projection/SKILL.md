---
name: 2d-skyline-and-diagonal-projection
description: Use when peak detection must be performed on simulated or real 2D J-Res
  or COSY 1H NMR spectra and a unified 1D cluster-localization algorithm is desired.
when_to_use_negative:
- Input is already a 1D 1H spectrum — apply the threshold-based cluster localization
  directly instead of projecting.
- A genuinely 2D peak-picking output (cross-peak positions in both F1 and F2) is required,
  since this skill collapses one dimension before localization.
edam_operation: http://edamontology.org/operation_3218
edam_topics: []
tools:
- name: MetAssimulo 2.0
  role: Simulation and peak-detection pipeline in which the skyline (F2) and diagonal
    projections feed the shared cluster-localization algorithm
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: Python
  role: Implementation language for the projection and cluster-localization module
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:880797366401169218dab04ce079d7702f150e5282d0ddc7613ba72a2c148cdf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yanyan5420/MetAssimulo_2.git
  tools:
  - MetAssimulo 2.0
  - Python
  edam_operation: http://edamontology.org/operation_3218
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# 2D-skyline-and-diagonal-projection

## Summary

Reduces 2D J-Res and COSY metabolomic 1H NMR spectra to 1D profiles via skyline (F2) or diagonal projection so that a single threshold-based cluster-localization peak-detection pathway can be shared with 1D spectra in MetAssimulo 2.0.

## When to use

Use when peak detection must be performed on simulated or real 2D J-Res or COSY 1H NMR spectra and a unified 1D cluster-localization algorithm is desired.

## When NOT to use

- Input is already a 1D 1H spectrum — apply the threshold-based cluster localization directly instead of projecting.
- A genuinely 2D peak-picking output (cross-peak positions in both F1 and F2) is required, since this skill collapses one dimension before localization.

## Inputs

- Simulated 2D J-Res 1H NMR spectrum (2D intensity matrix)
- Simulated 2D COSY 1H NMR spectrum (2D intensity matrix)
- User-defined intensity threshold parameter

## Outputs

- Projected 1D intensity profile (skyline or diagonal)
- Detected peak clusters with positions above the intensity threshold

## How to apply

Load the simulated 2D spectrum (J-Res or COSY) into Python as part of the MetAssimulo 2.0 pipeline. For J-Res spectra, compute a skyline projection along F2, collapsing the 2D matrix to a projected 1D profile. For COSY spectra, extract the diagonal projection, which captures the correlated peaks along the diagonal. Feed the resulting 1D profile into the same user-defined intensity-threshold cluster-localization routine used for 1D 1H spectra: scan the profile, mark local maxima above the user-specified threshold, group adjacent above-threshold points into clusters, and discard clusters failing the filtering criteria. This design keeps one common peak-detection entry point for 1D, J-Res and COSY data, so downstream localization logic does not need dimension-specific variants.

## Related tools

- **MetAssimulo 2.0** (Simulation and peak-detection pipeline in which the skyline (F2) and diagonal projections feed the shared cluster-localization algorithm) — https://github.com/yanyan5420/MetAssimulo_2.git
- **Python** (Implementation language for the projection and cluster-localization module)

## Examples

```
python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- The projected 1D profile (skyline for J-Res, diagonal for COSY) is produced without altering peak positions relative to the original 2D matrix.
- Detected clusters match manual inspection at the same user-defined intensity threshold.
- Identical threshold applied to a 1D 1H spectrum and to the projected 2D profile yields clusters via the same code path.
- Clusters failing the filtering criteria are discarded consistently across 1D, J-Res and COSY inputs.

## Limitations

- The projection collapses a dimension, so cross-peak geometry of COSY/J-Res spectra is not preserved after this step.
- Peak detection quality depends on the user-specified intensity threshold; no automatic threshold selection is described in the article.

## Evidence

- [methods] For 1D 1H spectra, peaks are located based on a user-defined intensity threshold; for 2D J-Res and COSY spectra, a skyline projection (F2) or diagonal projection first produces a 1D profile that then feeds the same cluster-localization algorithm.: [evidence span withheld — anchor status: not_found]
- [abstract] MetAssimulo 2.0 expands functionality to urine, blood, and cerebral spinal fluid, enhancing the realism of blood spectra by incorporating a broad protein background.: "MetAssimulo 2.0 expands functionality to urine, blood, and cerebral spinal fluid, enhancing the realism of blood spectra by incorporating a broad protein background."
