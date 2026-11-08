---
name: fatty-acid-isotopologue-distribution-fitting
description: Use when you have normalized, natural-isotope-corrected isotopologue
  intensities (M+0..M+16/M+18) for a fatty acid and need the glucose-derived acetyl-CoA
  labelling fraction p (e.g. per-cell p histograms for ACLYkd vs control).
when_to_use_negative:
- When overall substrate usage is very low, such as in the hypoxia/normoxia experiment,
  where it becomes impossible to reliably fit a binomial distribution and estimate
  the uptake fraction
- When the input intensities are not corrected for natural isotope abundance (e.g.
  raw METASPACE isotopologue distributions), as the fitted p would be biased
- When the goal is absolute rather than relative substrate contribution to the lipogenic
  acetyl-CoA pool, since the method cannot quantify pool size
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: 13C_binomial
  role: Standalone Python package implementing fit_binomial for C16/C18 models on
    corrected isotopologue intensities; recommended entry point for modelling bulk/MSI
    isotope tracing data
  repo: https://github.com/Buglakova/13C_binomial
- name: 13C-SpaceM
  role: Single-cell workflow whose sciso routines fit the binomial distribution per
    cell after natural isotope correction and normalization, producing per-cell p
    values
  repo: https://github.com/Buglakova/13C-SpaceM
- name: Scanpy
  role: Stores and reads the pickled anndata objects holding per-cell normalized isotopologue
    distributions used as fitting input
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:a7ca1aa41214ec7bfaab0cd534bfd5393bb923d5b663386a8d9b9567ad474b74
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Buglakova/13C_binomial
  tools:
  - 13C-SpaceM
  - 13C_binomial
  - Scanpy
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# fatty-acid-isotopologue-distribution-fitting

## Summary

Fits a 13C binomial model to the isotopologue distribution of a fatty acid (e.g. C16 palmitate, C18 stearate/oleate) after natural isotope abundance correction, to estimate the labelling degree p of the lipogenic acetyl-CoA pool and an uptake fraction. This is the core quantitative inference step of 13C-SpaceM.

## When to use

Use when you have normalized, natural-isotope-corrected isotopologue intensities (M+0..M+16/M+18) for a fatty acid and need the glucose-derived acetyl-CoA labelling fraction p (e.g. per-cell p histograms for ACLYkd vs control).

## When NOT to use

- When overall substrate usage is very low, such as in the hypoxia/normoxia experiment, where it becomes impossible to reliably fit a binomial distribution and estimate the uptake fraction
- When the input intensities are not corrected for natural isotope abundance (e.g. raw METASPACE isotopologue distributions), as the fitted p would be biased
- When the goal is absolute rather than relative substrate contribution to the lipogenic acetyl-CoA pool, since the method cannot quantify pool size

## Inputs

- numpy array of normalized, natural-isotope-abundance-corrected intensities per isotopologue peak (M+0..M+16 for C16; M+0..M+18 for C18)
- fatty acid chain-length/model type identifier ('C16' or 'C18')
- optional: CSV exported from Isocorrector software (e.g. IsoCorrectoR_ACLY.csv)
- per-cell isotopologue distributions from SpaceM/anndata objects (sc_analysis.py output)

## Outputs

- fit_binomial_result.csv with per-sample model (C16/C18), fit_success flag, uptake, uptake_palmitate, p, distribution mean, and p_mean
- fit_binomial_prediction.csv containing the fitted model prediction per sample in the same table format as the input
- plots overlaying the original isotopologue bars with the fitted model line
- per-cell acetyl-CoA labelling degree p values for condition-level histograms

## How to apply

Use the fit_binomial function from Buglakova/13C_binomial, passing a numpy array of normalized intensities corrected for natural isotope abundance for each isotopologue peak, plus a model type: 'C16' for palmitate or 'C18' for fatty acids where the M+2 peak can also come from unlabeled palmitate + 2 x 13C. The routine optimizes the binomial parameter p (acetyl-CoA labelling degree) and an uptake parameter; for C18 it additionally estimates palmitate uptake. The optimization output is cross-checked against a mean-based estimate p_mean, since for a binomial the distribution mean equals p*n. Group the fitted p values per condition to produce per-cell histograms and read off the histogram peaks (e.g. control p≈0.6, ACLYkd oligo 1 p≈0.35, oligo 2 p≈0.4).

## Related tools

- **13C_binomial** (Standalone Python package implementing fit_binomial for C16/C18 models on corrected isotopologue intensities; recommended entry point for modelling bulk/MSI isotope tracing data) — https://github.com/Buglakova/13C_binomial
- **13C-SpaceM** (Single-cell workflow whose sciso routines fit the binomial distribution per cell after natural isotope correction and normalization, producing per-cell p values) — https://github.com/Buglakova/13C-SpaceM
- **Scanpy** (Stores and reads the pickled anndata objects holding per-cell normalized isotopologue distributions used as fitting input)

## Examples

```
from fit_binomial import fit_binomial; result = fit_binomial(isotopologue_intensities, 'C16')
```

## Evaluation signals

- fit_success == 1 in fit_binomial_result.csv (optimization reached its convergence condition)
- p estimated by the optimizer agrees with p_mean derived from the distribution mean (binomial mean = p*n) for C16 data
- For C18, the M+2 peak is attributed consistently: model must account for unlabeled palmitate + 2 x 13C before inferring palmitate uptake
- Reproduced per-condition p histograms peak near reported values: control ≈0.6, ACLYkd oligo 1 ≈0.35 (bimodal, range 0.2–0.65), oligo 2 ≈0.4
- Plotted model prediction line follows the original isotopologue bar heights across M+i peaks

## Limitations

- If overall substrate usage is very low (e.g. hypoxia/normoxia data), the binomial fit cannot reliably estimate the uptake fraction; it was therefore only used for the ACLYkd data
- The method yields only relative, not absolute, substrate contribution to the lipogenic acetyl-CoA pool because pool size cannot be quantified
- Different fatty acid pools may reach isotopic steady state at different times; in the 48-h in vivo labelling steady state was likely not reached, so labelling data must be interpreted alongside pool size data
- The C18 model must decompose the M+2 peak (unlabeled palmitate + 2 x 13C), so model choice matters for chain length and co-eluting species

## Evidence

- [readme] Function fit_binomial in fit_binomial.py takes as input numpy array of normalized and corrected for natural isotope abundance intensities for each isotopologue peak of the fatty acid and the model type.: [evidence span withheld — anchor status: external_doc]
- [readme] Model can be either C16 or C18. C18 takes into account that M+2 peak can come from the unlabeled palmitate + 2 13C.: [evidence span withheld — anchor status: external_doc]
- [readme] fit_binomial_result.csv contains result of fit for each sample — model, fit_success, uptake, uptake_palmitate, p, mean, p_mean.: [evidence span withheld — anchor status: external_doc]
- [discussion] if overall substrate usage is very low, such as in the hypoxia/normoxia experiment, it becomes impossible to reliably fit a binomial distribution and estimate the uptake fraction: "if overall substrate usage is very low, such as in the hypoxia/normoxia experiment, it becomes impossible to reliably fit a binomial distribution and estimate the uptake fraction"
- [methods] Fit binomial distribution to each fatty acid's isotopologue distribution.: "Fit binomial distribution to each fatty acid's isotopologue distribution"
- [readme] p_mean: parameter of binomial distribution estimated from the distribution mean (for binomial mean = pn).: [evidence span withheld — anchor status: external_doc]
