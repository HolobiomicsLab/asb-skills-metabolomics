---
name: binomial-labelling-model-inference
description: Use when you have normalized, natural-isotope-corrected isotopologue
  distributions for a fatty acid (e.g. palmitate M+0..M+16) and need the glucose-derived
  acetyl-CoA labelling degree p per cell, sample, or pixel.
when_to_use_negative:
- When overall substrate usage is very low, as in the hypoxia/normoxia experiment
  — the binomial distribution cannot be reliably fitted and the uptake fraction cannot
  be estimated; it was only used for ACLYkd data
- When isotopologue intensities are raw and not yet corrected for natural isotope
  abundance and normalized per cell/pixel — apply the sc_analysis.py correction and
  normalization steps first
- When labelling data must be interpreted against pool size (e.g. in vivo mouse tissue
  where isotopic steady state was likely not reached within the 48-h labelling period)
  — the model alone gives relative, not absolute, substrate contribution
edam_operation: http://edamontology.org/operation_3659
tools:
- name: 13C_binomial
  role: Standalone implementation (fit_binomial.py) of the binomial model with C16/C18
    model types and example IsoCorrectoR analysis notebook
  repo: https://github.com/Buglakova/13C_binomial
- name: 13C-SpaceM
  role: Single-cell pipeline whose sciso fitting routine applies the binomial model
    to per-cell palmitate isotopologue distributions from anndata objects
  repo: https://github.com/Buglakova/13C-SpaceM
- name: Scanpy
  role: Stores and loads the pickled anndata single-cell objects whose isotopologue
    distributions are the per-cell fitting inputs
  repo: https://github.com/Buglakova/13C-SpaceM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:8adbae151216e073b9154075d401ecb1d90c140ddc1151a68239dc36d3517dd5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Buglakova/13C_binomial
  tools:
  - 13C-SpaceM
  - 13C_binomial
  - Scanpy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# binomial-labelling-model-inference

## Summary

Fits a binomial distribution to natural-isotope-corrected, normalized fatty acid isotopologue intensities to estimate the labelling fraction p of the lipogenic acetyl-CoA pool (and uptake fraction) from 13C-glucose tracing data. This is the quantitative core of 13C-SpaceM's single-cell isotope tracing analysis.

## When to use

Use when you have normalized, natural-isotope-corrected isotopologue distributions for a fatty acid (e.g. palmitate M+0..M+16) and need the glucose-derived acetyl-CoA labelling degree p per cell, sample, or pixel.

## When NOT to use

- When overall substrate usage is very low, as in the hypoxia/normoxia experiment — the binomial distribution cannot be reliably fitted and the uptake fraction cannot be estimated; it was only used for ACLYkd data
- When isotopologue intensities are raw and not yet corrected for natural isotope abundance and normalized per cell/pixel — apply the sc_analysis.py correction and normalization steps first
- When labelling data must be interpreted against pool size (e.g. in vivo mouse tissue where isotopic steady state was likely not reached within the 48-h labelling period) — the model alone gives relative, not absolute, substrate contribution

## Inputs

- numpy array of normalized intensities per isotopologue peak (M+0..M+n) for a fatty acid, corrected for natural isotope abundance
- model type string: 'C16' or 'C18'
- optionally, IsoCorrectoR output CSV (e.g. data/IsoCorrectoR_ACLY.csv) processed via example_isocorrector_analysis.ipynb
- single-cell anndata objects with per-cell palmitate isotopologue distributions from the 13C-SpaceM workflow

## Outputs

- fit_binomial_result.csv with per-sample model (C16/C18), fit_success, uptake, uptake_palmitate (C18 only), p, mean, and p_mean
- fit_binomial_prediction.csv with model-predicted isotopologue distributions in the same format as the input
- plots showing original data as bars and fitted model prediction as a line
- per-cell p values grouped by condition for histogram analysis (control p≈0.6, ACLYkd oligo 1 p≈0.35, oligo 2 p≈0.4)

## How to apply

Take the numpy array of normalized intensities corrected for natural isotope abundance for each isotopologue peak of the fatty acid, and choose model type 'C16' (e.g. palmitate) or 'C18' (e.g. stearate/oleate), where C18 accounts for the M+2 peak coming from unlabeled palmitate + 2×13C. Call fit_binomial (fit_binomial.py in the 13C_binomial repo) to optimize the binomial parameters p and uptake; in the 13C-SpaceM single-cell workflow this is applied per cell to palmitate isotopologue distributions stored in Scanpy anndata objects produced by sc_analysis.py. The result file fit_binomial_result.csv reports model type, fit_success (1 if optimization converged), uptake, p, the distribution mean, and p_mean (p estimated from the binomial mean, mean = p·n). For ACLY knockdown data the fitted per-cell p values are grouped by condition and plotted as histograms, reproducing peaks at p≈0.6 (control), p≈0.35 (oligo 1), and p≈0.4 (oligo 2). Use the same per-pixel fitting for mouse brain tissue data after natural isotope abundance correction.

## Related tools

- **13C_binomial** (Standalone implementation (fit_binomial.py) of the binomial model with C16/C18 model types and example IsoCorrectoR analysis notebook) — https://github.com/Buglakova/13C_binomial
- **13C-SpaceM** (Single-cell pipeline whose sciso fitting routine applies the binomial model to per-cell palmitate isotopologue distributions from anndata objects) — https://github.com/Buglakova/13C-SpaceM
- **Scanpy** (Stores and loads the pickled anndata single-cell objects whose isotopologue distributions are the per-cell fitting inputs) — https://github.com/Buglakova/13C-SpaceM

## Examples

```
from fit_binomial import fit_binomial; result = fit_binomial(intensities_array, 'C16')
```

## Evaluation signals

- fit_success = 1 in fit_binomial_result.csv (optimization reached the convergence condition)
- Fitted p values reproduce expected histograms for ACLYkd data: peak at p≈0.6 in control, p≈0.35 (bimodal, range 0.2–0.65) in oligo 1, p≈0.4 in oligo 2
- For binomial distributions, p from the optimizer agrees with p_mean estimated from the distribution mean (mean = p·n), with M+0 (and M+2 for C18) set to 0
- Plotted model prediction line tracks the bar heights of the original isotopologue distribution

## Limitations

- If overall substrate usage is very low (hypoxia/normoxia experiment), a reliable binomial fit and uptake fraction estimate are impossible, so the method was only used for ACLYkd data
- The model yields relative substrate contribution to the lipogenic acetyl-CoA pool only, since pool size cannot be quantified
- For C18 fatty acids the M+2 peak conflates de novo labelling with unlabeled palmitate + 2×13C, requiring the C18 model variant
- Different fatty acid pools may reach isotopic steady state at different times, affecting accurate de novo synthesis vs uptake quantification

## Evidence

- [readme] takes as input numpy array of normalized and corrected for natural isotope abundance intensities for each isotopologue peak of the fatty acid and the model type: [evidence span withheld — anchor status: external_doc]
- [readme] Model can be either `C16` or `C18`. `C18` takes into account that M+2 peak can come from the unlabeled palmitate + $2^{13}C$.: [evidence span withheld — anchor status: external_doc]
- [methods] Fit binomial distribution to each fatty acid's isotopologue distribution: "Fit binomial distribution to each fatty acid's isotopologue distribution"
- [discussion] if overall substrate usage is very low, such as in the hypoxia/normoxia experiment, it becomes impossible to reliably fit a binomial distribution and estimate the uptake fraction: "if overall substrate usage is very low, such as in the hypoxia/normoxia experiment, it becomes impossible to reliably fit a binomial distribution and estimate the uptake fraction"
