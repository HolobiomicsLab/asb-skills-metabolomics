---
name: acetyl-coa-fractional-labelling-estimation
description: Use when you have natural-isotope-corrected, normalized fatty acid isotopologue
  distributions from 13C6-glucose tracing and need per-cell (or per-pixel) fractional
  labelling p of the lipogenic acetyl-CoA pool, e.g. for ACLY knockdown or control
  conditions.
when_to_use_negative:
- When overall substrate usage is very low, e.g. the hypoxia/normoxia experiment —
  the article states it becomes impossible to reliably fit a binomial distribution
  and estimate the uptake fraction, so the model was only used for ACLYkd data
- When you need absolute rather than relative contribution to the lipogenic acetyl-CoA
  pool — the methodology cannot quantify pool size
- When the input is raw imzML or uncorrected isotopologue intensities — natural isotope
  abundance correction and per-cell normalization must be done first
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_0154
tools:
- name: 13C_binomial
  role: Provides fit_binomial (fit_binomial.py) that fits the binomial model to normalized,
    natural-isotope-corrected fatty acid isotopologue distributions; includes Isocorrector
    example notebook and environment.yml
  repo: https://github.com/Buglakova/13C_binomial
- name: Scanpy
  role: Loads the pickled anndata objects storing per-cell normalized isotopologue
    distributions used as fitting input
- name: 13C-SpaceM
  role: Upstream pipeline producing the quality-controlled single-cell anndata objects
    (sc_analysis.py, sciso fitting routine) whose palmitate isotopologue distributions
    are fitted
  repo: https://github.com/Buglakova/13C-SpaceM
- name: IsoCorrectoR
  role: External tool whose natural-isotope-corrected isotopologue output CSV can
    be analyzed via the provided example_isocorrector_analysis.ipynb notebook
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:fb2dac376b405a37301e3e85fa95b1da333c352ba31acd19a2dcc1042b695d9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Buglakova/13C_binomial
  tools:
  - 13C-SpaceM
  - 13C_binomial
  - IsoCorrectoR
  - Scanpy
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# acetyl-coa-fractional-labelling-estimation

## Summary

Estimates the fractional labelling (p) of the lipogenic acetyl-CoA pool by fitting a 13C binomial model to normalized, natural-isotope-corrected palmitate (C16) or stearate/oleate (C18) isotopologue distributions, per cell or per pixel, from 13C-SpaceM single-cell data.

## When to use

Use when you have natural-isotope-corrected, normalized fatty acid isotopologue distributions from 13C6-glucose tracing and need per-cell (or per-pixel) fractional labelling p of the lipogenic acetyl-CoA pool, e.g. for ACLY knockdown or control conditions.

## When NOT to use

- When overall substrate usage is very low, e.g. the hypoxia/normoxia experiment — the article states it becomes impossible to reliably fit a binomial distribution and estimate the uptake fraction, so the model was only used for ACLYkd data
- When you need absolute rather than relative contribution to the lipogenic acetyl-CoA pool — the methodology cannot quantify pool size
- When the input is raw imzML or uncorrected isotopologue intensities — natural isotope abundance correction and per-cell normalization must be done first

## Inputs

- Natural-isotope-abundance-corrected, normalized isotopologue intensity vectors (numpy arrays or CSV, e.g. from IsoCorrectoR) per fatty acid per cell
- Quality-controlled single-cell anndata objects (.h5ad) from 13C-SpaceM sc_analysis.py for ACLYkd or hypoxia/normoxia experiments
- Fatty acid model type specification: 'C16' or 'C18'

## Outputs

- Per-sample/per-cell fitted parameters table (fit_binomial_result.csv): model, fit_success, uptake, uptake_palmitate, p, mean, p_mean
- Model predictions matching the input format (fit_binomial_prediction.csv)
- Plots of original isotopologue bars with fitted model prediction line, and per-condition histograms of fitted p

## How to apply

Load the quality-controlled single-cell anndata objects for the relevant experiment (e.g. control, ACLYkd oligo 1, ACLYkd oligo 2) with Scanpy. Extract each cell's normalized palmitate isotopologue distribution (already natural-isotope-abundance corrected and median-aggregated per cell by sc_analysis.py). Fit a binomial model with fit_binomial from the 13C_binomial repository, choosing model type 'C16' for palmitate or 'C18' for 18-carbon fatty acids — the C18 model additionally accounts for the M+2 peak arising from unlabeled palmitate plus two 13C atoms. The fit returns parameters p (binomial labelling fraction), uptake, mean, and p_mean (p estimated from the distribution mean, mean = pn). Group fitted p values by condition and plot per-cell histograms to reveal heterogeneity hidden in bulk analyses.

## Related tools

- **13C_binomial** (Provides fit_binomial (fit_binomial.py) that fits the binomial model to normalized, natural-isotope-corrected fatty acid isotopologue distributions; includes Isocorrector example notebook and environment.yml) — https://github.com/Buglakova/13C_binomial
- **Scanpy** (Loads the pickled anndata objects storing per-cell normalized isotopologue distributions used as fitting input)
- **13C-SpaceM** (Upstream pipeline producing the quality-controlled single-cell anndata objects (sc_analysis.py, sciso fitting routine) whose palmitate isotopologue distributions are fitted) — https://github.com/Buglakova/13C-SpaceM
- **IsoCorrectoR** (External tool whose natural-isotope-corrected isotopologue output CSV can be analyzed via the provided example_isocorrector_analysis.ipynb notebook)

## Examples

```
from fit_binomial import fit_binomial; result = fit_binomial(normalized_corrected_intensities, model='C16')
```

## Evaluation signals

- fit_success == 1 in fit_binomial_result.csv for each optimized sample (optimization reached convergence)
- Histogram peaks of fitted p match reported values for the ACLYkd experiment: control ≈ 0.6, shRNA oligo 1 ≈ 0.35 (bimodal, range 0.2–0.65), oligo 2 ≈ 0.4
- For C18 fits, verify the model accounts for the M+2 peak coming from unlabeled palmitate + two 13C; p_mean should approximate p since mean = pn for a binomial
- fit_binomial_prediction.csv values should closely overlay the original isotopologue bars in the plotted prediction lines
- Fitted palmitate M+0 fraction should show only a minor increase under ACLY silencing even as p drops

## Limitations

- The binomial model cannot reliably estimate the uptake fraction when overall substrate usage is very low (e.g. hypoxia/normoxia); it was only applied to ACLYkd data
- Only relative, not absolute, substrate contribution to the lipogenic acetyl-CoA pool can be inferred, since pool size cannot be quantified
- Different fatty acid pools may reach isotopic steady state at different times; in the in vivo mouse model isotopic steady state was likely not reached within the 48-h labelling period, so labelling must be interpreted alongside pool size data
- Mixed Scanpy/anndata/pandas/pickle versions can cause errors reading the pickled anndata objects; consult environment_dump.yml for troubleshooting

## Evidence

- [readme] fit_binomial: [evidence span withheld — anchor status: external_doc]
- [results] acetyl-CoA histogram peaks: [evidence span withheld — anchor status: not_found]
- [discussion] only for ACLYkd: "if overall substrate usage is very low, such as in the hypoxia/normoxia experiment, it becomes impossible to reliably fit a binomial distribution and estimate the uptake fraction, therefore it was"
- [methods] ACLYkd workflow: "Fit binomial distribution to each fatty acid's isotopologue distribution"
- [discussion] ACLY silencing effect: "silencing of ACLY caused a reduction in the fractional labelling of acetyl-CoA from glucose but only a minor increase in the M + 0 fraction of palmitate."
