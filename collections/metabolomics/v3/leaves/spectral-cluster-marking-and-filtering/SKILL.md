---
name: spectral-cluster-marking-and-filtering
description: Use when reconstructing or validating peak-detection code for simulated
  1D 1H, 2D J-Res or COSY metabolomic NMR spectra where peaks must be localized as
  clusters above a user-defined intensity threshold before downstream spectral simulation.
when_to_use_negative:
- Input is raw unprocessed spectra where no user-defined intensity threshold has been
  specified — the method requires a threshold value to locate maxima.
- The goal is de novo 2D peak picking on the full 2D matrix (J-Res or COSY) rather
  than 1D-projected cluster localization via skyline or diagonal projection.
- Spectra are real experimental data where protein background or other artefacts dominate;
  MetAssimulo 2.0's cluster localization is designed for its simulated spectral structures.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: MetAssimulo 2.0
  role: The source implementation whose cluster-marking and filtering logic this skill
    reconstructs; provides the 1D, J-Res and COSY simulated spectral data structures
    and the common cluster-localization entry point.
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: Python
  role: Language used to implement the threshold-based cluster-localization module
    and to run MetAssimulo 2.0 as a web application.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:01a2e802cd6757366e25e607c81b7f6ee2e6fa6af8668d05690016d6e2ddd170
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Spectral cluster marking and filtering

## Summary

Locates peaks in simulated NMR spectra by grouping adjacent points above a user-defined intensity threshold into clusters and filtering clusters that fail criteria, applying the same cluster-localization pathway to 1D 1H, J-Res (skyline projection) and COSY (diagonal projection) spectra in MetAssimulo 2.0.

## When to use

Use when reconstructing or validating peak-detection code for simulated 1D 1H, 2D J-Res or COSY metabolomic NMR spectra where peaks must be localized as clusters above a user-defined intensity threshold before downstream spectral simulation.

## When NOT to use

- Input is raw unprocessed spectra where no user-defined intensity threshold has been specified — the method requires a threshold value to locate maxima.
- The goal is de novo 2D peak picking on the full 2D matrix (J-Res or COSY) rather than 1D-projected cluster localization via skyline or diagonal projection.
- Spectra are real experimental data where protein background or other artefacts dominate; MetAssimulo 2.0's cluster localization is designed for its simulated spectral structures.

## Inputs

- Simulated 1D 1H NMR spectral data structures
- Simulated 2D J-Res NMR spectral data structures
- Simulated 2D COSY NMR spectral data structures
- User-defined intensity threshold parameter
- MetAssimulo 2.0 Input/parameters.txt configuration

## Outputs

- Marked peak clusters (groups of adjacent above-threshold points)
- Filtered cluster list surviving filtering criteria
- Peak-localization results usable as input to mixture spectrum simulation

## How to apply

Load the simulated spectral data structure for 1D 1H, J-Res or COSY spectra and scan for local maxima whose intensity exceeds the user-specified threshold. Mark and group adjacent points above threshold into clusters around each detected peak, then discard clusters that fail the filtering criteria. For 2D J-Res spectra, first collapse the 2D matrix along F2 via a skyline projection to produce a 1D profile that feeds the same cluster-localization algorithm; for COSY spectra, extract the diagonal signal via diagonal projection and feed it into the same routine. Wrap these steps into a reusable module with a common cluster-localization entry point so all three spectral types share one peak-detection pathway. Rationale: MetAssimulo 2.0 needs a consistent peak-localization mechanism across spectral dimensionalities before it can simulate mixture spectra, and the threshold-based clustering is deliberately shared between projected and native 1D profiles.

## Related tools

- **MetAssimulo 2.0** (The source implementation whose cluster-marking and filtering logic this skill reconstructs; provides the 1D, J-Res and COSY simulated spectral data structures and the common cluster-localization entry point.) — https://github.com/yanyan5420/MetAssimulo_2.git
- **Python** (Language used to implement the threshold-based cluster-localization module and to run MetAssimulo 2.0 as a web application.)

## Examples

```
conda activate py37_metassimulo_2_env && python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- Run the module end-to-end on a simulated spectrum from the Input data and confirm detected peak clusters match manual inspection at the user-defined intensity threshold.
- Verify that J-Res spectra first produce a skyline (F2) projection and COSY spectra a diagonal projection before entering the shared cluster-localization routine.
- Check that clusters discarded are exactly those failing the filtering criteria, and no above-threshold local maxima are missed.
- Confirm a single common cluster-localization entry point is exercised by 1D, J-Res and COSY pathways (code-level invariant).
- Peak-cluster positions should align with known metabolite peaks in the MetaboLights MTBLS12081 reference spectra.

## Limitations

- Peak detection depends on a user-defined intensity threshold; inappropriate thresholds cause missed or spurious clusters.
- For 2D J-Res and COSY spectra, peaks are only localized from 1D projections (skyline or diagonal), not from the full 2D matrix, so off-diagonal 2D-only signals are not captured as separate clusters.
- The method is validated within MetAssimulo 2.0's simulated spectral structures; applying it to other spectral formats requires adaptation.

## Evidence

- [methods] 1D threshold peak location: [evidence span withheld — anchor status: not_found]
- [methods] shared cluster-localization algorithm: [evidence span withheld — anchor status: not_found]
- [abstract] MetAssimulo 2.0 upgrade scope: "Here, we introduce MetAssimulo 2.0, a comprehensive upgrade of the MetAssimulo 1.b metabolomic 1H NMR simulation tool, reimplemented as a Python-based web application."
- [readme] repository and instruction location: "This repository contains all the necessary code for the MetAssimulo 2 project. Please follow the instructions below to set up and run the project on your local machine."
