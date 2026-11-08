---
name: binomial-isotopologue-distribution-fitting
description: Use when input consists of isotope-tracing (13C6-glucose) isotopologue
  distributions of fatty acids (e.g. palmitate, stearate, oleate) from MALDI-MSI or
  single-cell SpaceM data that are already corrected for natural isotope abundance
  and normalized, and you need p and uptake fractions.
when_to_use_negative:
- Overall substrate usage is very low, e.g. the hypoxia/normoxia experiment, where
  it becomes impossible to reliably fit a binomial distribution and estimate the uptake
  fraction
- Input isotopologue distributions have not yet been corrected for natural 13C abundance
  and normalized — apply isotope correction first or the fit is invalid
- Absolute (rather than relative) substrate contribution to the lipogenic acetyl-CoA
  pool is required — the method only provides relative contributions
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: 13C-binomial
  role: Standalone repo with fit_binomial.py implementing the C16/C18 binomial models
    fitted to corrected, normalized isotopologue distributions
  repo: https://github.com/Buglakova/13C_binomial
- name: 13C-SpaceM
  role: Produces the single-cell anndata objects (after isotope correction, median
    aggregation, and normalization) and per-pixel brain tissue ion images that feed
    the binomial fits
  repo: https://github.com/Buglakova/13C-SpaceM
- name: Scanpy
  role: Reads the pickled anndata objects containing per-cell isotopologue distributions
    used as fitting inputs
  repo: https://github.com/Buglakova/13C-SpaceM
- name: pyimzML
  role: Parses imzML files to extract per-pixel ion images for all fatty acid isotopologues
    (M+0 to M+16+) prior to correction and fitting
  repo: https://github.com/alexandrovteam/pyimzML
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_007
  - task_002
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:97e351a12f7a72bdb8f5b97044ee8d3544fdbf4000274be33dbf9a2793a05329
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Buglakova/13C_binomial
  tools:
  - 13C-SpaceM
  - 13C-binomial
  - METASPACE
  - Scanpy
  - SpaceM
  - pyimzML
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# binomial-isotopologue-distribution-fitting

## Summary

Fits a 13C-binomial model to natural-isotope-abundance-corrected, normalized fatty acid isotopologue distributions (M+0...M+16) to estimate the fractional labelling p of the lipogenic acetyl-CoA pool and the relative uptake vs de novo synthesis fraction, per cell or per MALDI-MSI pixel.

## When to use

Use when input consists of isotope-tracing (13C6-glucose) isotopologue distributions of fatty acids (e.g. palmitate, stearate, oleate) from MALDI-MSI or single-cell SpaceM data that are already corrected for natural isotope abundance and normalized, and you need p and uptake fractions.

## When NOT to use

- Overall substrate usage is very low, e.g. the hypoxia/normoxia experiment, where it becomes impossible to reliably fit a binomial distribution and estimate the uptake fraction
- Input isotopologue distributions have not yet been corrected for natural 13C abundance and normalized — apply isotope correction first or the fit is invalid
- Absolute (rather than relative) substrate contribution to the lipogenic acetyl-CoA pool is required — the method only provides relative contributions

## Inputs

- Natural-isotope-abundance-corrected, normalized isotopologue intensity vector per fatty acid per cell or pixel (numpy array; e.g. from Isocorrector output or 13C-SpaceM anndata objects)
- Pseudo-bulk isotopologue distributions per condition (median-averaged single-cell fractions)
- Model type identifier: 'C16' or 'C18'

## Outputs

- fit_binomial_result.csv: per-sample fitted parameters (model, fit_success, uptake, uptake_palmitate, p, mean, p_mean)
- fit_binomial_prediction.csv: model-predicted isotopologue distributions matching the input table format
- Per-cell/per-pixel relative uptake fraction tables (e.g. brain_denovo_fractions.csv)
- Spatial heatmaps of fitted p values and histograms of per-cell uptake fractions

## How to apply

First perform natural isotope abundance correction and normalize each fatty acid's isotopologue distribution (M+0 ... M+16) to fractional abundances, per cell (median-aggregated across ablation marks in an anndata object) or per pixel of an imzML-derived ion image stack. Use the 13C_binomial repo's fit_binomial function (fit_binomial.py), selecting model 'C16' for C16 fatty acids like palmitate and 'C18' for C18 fatty acids, since the C18 model accounts for the M+2 peak coming from unlabeled palmitate carrying two 13C (chain elongation). The fit returns p (binomial parameter = fractional glucose-derived acetyl-CoA labelling), uptake (relative uptake fraction, 1-p related de novo contribution), fit_success, distribution mean, and p_mean estimated from the distribution mean (mean = p*n). Aggregate per-pixel or per-cell results to produce spatial heatmaps (e.g. glioma tumour p in 0.45–0.70) or per-cell histograms (control liver cancer cells at 60–90% uptake) and validate against reported peaks, e.g. control palmitate histogram peaking at ~0.55 with palmitoleate/oleate at higher uptake.

## Related tools

- **13C-binomial** (Standalone repo with fit_binomial.py implementing the C16/C18 binomial models fitted to corrected, normalized isotopologue distributions) — https://github.com/Buglakova/13C_binomial
- **13C-SpaceM** (Produces the single-cell anndata objects (after isotope correction, median aggregation, and normalization) and per-pixel brain tissue ion images that feed the binomial fits) — https://github.com/Buglakova/13C-SpaceM
- **Scanpy** (Reads the pickled anndata objects containing per-cell isotopologue distributions used as fitting inputs) — https://github.com/Buglakova/13C-SpaceM
- **pyimzML** (Parses imzML files to extract per-pixel ion images for all fatty acid isotopologues (M+0 to M+16+) prior to correction and fitting) — https://github.com/alexandrovteam/pyimzML

## Examples

```
from fit_binomial import fit_binomial; result = fit_binomial(corrected_normalized_intensities, model='C16')
```

## Evaluation signals

- fit_success = 1 (optimization converged) in fit_binomial_result.csv for the fitted samples
- Control liver cancer cells: fitted relative uptake fractions fall in the reported 60–90% range, with palmitate histogram peaking at ~0.55 and palmitoleate/oleate peaking at higher uptake than palmitate
- Glioma tumour pixels: fitted palmitate p values fall in the 0.45–0.70 range while total ion count is spatially uniform
- Predicted isotopologue distributions in fit_binomial_prediction.csv overlay the original bar distributions (bars = data, line = fitted model)
- For C18 fatty acids, the C18 model must be used and the M+2 enrichment in stearate/oleate is consistent with elongation of palmitate/palmitoleate chains

## Limitations

- If overall substrate usage is very low (e.g. the hypoxia/normoxia experiment), it becomes impossible to reliably fit a binomial distribution and estimate the uptake fraction; the model was only used for ACLYkd data in that context
- The method provides only relative, not absolute, substrate contribution to the lipogenic acetyl-CoA pool, since the pool size cannot be quantified
- In the in vivo mouse model, isotopic steady state was likely not reached for fatty acids within the 48-h 13C6-glucose labelling period, so fitted fractions must be interpreted alongside pool size data

## Evidence

- [readme] C18 model accounts for elongation: [evidence span withheld — anchor status: external_doc]
- [methods] binomial fit on brain pixels: "Perform natural isotope abundance correction for each ion's isotopologue distribution - Fit binomial distribution to each fatty acid's isotopologue distribution"
- [readme] input requirements: [evidence span withheld — anchor status: external_doc]
- [discussion] fit fails at low substrate usage: "if overall substrate usage is very low, such as in the hypoxia/normoxia experiment, it becomes impossible to reliably fit a binomial distribution and estimate the uptake fraction"
- [abstract] revealed heterogeneity: "This revealed substantial single-cell heterogeneity in labelling of the lipogenic acetyl-CoA pool and in relative fatty acid uptake versus synthesis hidden in bulk analyses."
