---
name: binomial-fitting-of-fatty-acid-uptake-fraction
description: Use when per-cell (or per-pixel) 13C6-glucose isotopologue distributions
  of fatty acids (e.g. C16 palmitate, C18 oleate) are available and the goal is to
  quantify relative uptake vs de novo synthesis as a fitted uptake fraction.
when_to_use_negative:
- When overall substrate usage is very low, e.g. the hypoxia/normoxia experiment —
  the authors state it 'becomes impossible to reliably fit a binomial distribution
  and estimate the uptake fraction', so it was only used for ACLYkd data.
- When labelling data are not at isotopic steady state and absolute substrate contributions
  are required — the method provides only relative, not absolute, contribution to
  the lipogenic acetyl-CoA pool.
- For bulk MS data without natural isotope abundance correction and normalization
  of isotopologue distributions — the fit requires corrected, normalized intensities
  as input.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: 13C_binomial
  role: Standalone Python package providing fit_binomial.py (C16/C18 binomial models)
    and example notebooks; the easier, well-documented companion to 13C-SpaceM for
    fitting isotopologue distributions
  repo: https://github.com/Buglakova/13C_binomial
- name: 13C-SpaceM
  role: Single-cell pipeline whose anndata objects (after natural isotope abundance
    correction and isotopologue normalization in sc_analysis.py) supply the per-cell
    isotopologue distributions that are binomial-fitted
  repo: https://github.com/Buglakova/13C-SpaceM
- name: Scanpy
  role: Loads the pickled anndata objects holding per-cell, QC-filtered isotopologue
    distributions used as fitting inputs
  repo: https://github.com/Buglakova/13C-SpaceM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:e4d37d93573b7d32055e120e87a99adff3f078bf684858e68ff55eb6b0d53e8b
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
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# binomial-fitting-of-fatty-acid-uptake-fraction

## Summary

Fits a binomial model to natural-abundance-corrected, normalized isotopologue distributions of fatty acids to estimate the fraction of the fatty acid pool derived from uptake versus de novo synthesis, as used in 13C-SpaceM single-cell isotope tracing.

## When to use

Use when per-cell (or per-pixel) 13C6-glucose isotopologue distributions of fatty acids (e.g. C16 palmitate, C18 oleate) are available and the goal is to quantify relative uptake vs de novo synthesis as a fitted uptake fraction.

## When NOT to use

- When overall substrate usage is very low, e.g. the hypoxia/normoxia experiment — the authors state it 'becomes impossible to reliably fit a binomial distribution and estimate the uptake fraction', so it was only used for ACLYkd data.
- When labelling data are not at isotopic steady state and absolute substrate contributions are required — the method provides only relative, not absolute, contribution to the lipogenic acetyl-CoA pool.
- For bulk MS data without natural isotope abundance correction and normalization of isotopologue distributions — the fit requires corrected, normalized intensities as input.

## Inputs

- numpy array of normalized intensities for each isotopologue peak of the fatty acid, corrected for natural isotope abundance
- model type string: 'C16' or 'C18'
- optionally, Isocorrector software output (see example_isocorrector_analysis.ipynb)

## Outputs

- fit_binomial_result.csv: per-sample model (C16/C18), fit_success flag, uptake, uptake_palmitate (C18 only), p, mean, p_mean
- fit_binomial_prediction.csv: model-predicted isotopologue distributions in the same format as the input table
- plots showing original isotopologue data as bars with the fitted model prediction as a line

## How to apply

Take a numpy array of normalized intensities, corrected for natural isotope abundance, for each isotopologue peak of the fatty acid and call `fit_binomial` in `fit_binomial.py`, selecting model type C16 or C18; the C18 model accounts for the M+2 peak coming from unlabeled palmitate plus two 13C atoms. The optimizer returns the uptake parameter, the binomial parameter p, the distribution mean (sum(i * I(M+i))/sum(I(M+i)), with M+0 set to 0 and additionally M+2 set to 0 for C18), and p_mean (p estimated from the mean, since for a binomial mean = pn). In the single-cell ACLYkd workflow, per-cell isotopologue distributions from the Scanpy anndata objects are fitted per fatty acid and per condition (control vs shRNA group), then uptake fractions are plotted as histograms to compare population modes. Apply this only where overall substrate usage is sufficient — the authors state that if overall substrate usage is very low (hypoxia/normoxia experiment), reliable binomial fitting is impossible, so the model was only used for the ACLYkd data.

## Related tools

- **13C_binomial** (Standalone Python package providing fit_binomial.py (C16/C18 binomial models) and example notebooks; the easier, well-documented companion to 13C-SpaceM for fitting isotopologue distributions) — https://github.com/Buglakova/13C_binomial
- **13C-SpaceM** (Single-cell pipeline whose anndata objects (after natural isotope abundance correction and isotopologue normalization in sc_analysis.py) supply the per-cell isotopologue distributions that are binomial-fitted) — https://github.com/Buglakova/13C-SpaceM
- **Scanpy** (Loads the pickled anndata objects holding per-cell, QC-filtered isotopologue distributions used as fitting inputs) — https://github.com/Buglakova/13C-SpaceM

## Examples

```
python fit_binomial.py --model C18  # via function: from fit_binomial import fit_binomial; result = fit_binomial(iso_intensities_np_array, 'C16')
```

## Evaluation signals

- fit_success column in fit_binomial_result.csv equals 1 (optimization reached its convergence condition) for each sample
- For the ACLYkd single-cell data, fitted palmitate uptake histogram peaks reproduce ~0.55 (control) vs ~0.61 (ACLYkd oligo 2) and oleate ~0.85 (control) vs ~0.95 (ACLYkd oligo 2), with unimodal distributions
- Overlay of fit_binomial_prediction.csv (model line) against the original isotopologue bars in the output plots visually matches the data
- For C18 fits, p_mean (p estimated from the distribution mean, mean = pn) is consistent with the optimizer's p parameter
- Reported result: ACLY silencing reduced fractional labelling of acetyl-CoA from glucose with only a minor increase in palmitate M+0 fraction

## Limitations

- The binomial fit is only reliable when overall substrate usage is not very low; in the hypoxia/normoxia experiment it was impossible to fit reliably, so it was restricted to the ACLYkd dataset
- The method provides only relative, not absolute, substrate contribution to the lipogenic acetyl-CoA pool because pool size cannot be quantified
- Different fatty acid pools may reach isotopic steady state at different times; in the in vivo mouse model isotopic steady state was likely not reached within the 48-h labelling period, so labelling must be interpreted with pool size data
- Fatty acids analysed derive only from lipids effectively ionized in negative MALDI-AIF mode (primarily acidic glycerophospholipids); the uptake proportion may differ by lipid class

## Evidence

- [readme] Function `fit_binomial` in `fit_binomial.py` takes as input numpy array of normalized and corrected for natural isotope abundance intensities for each isotopologue peak of the fatty acid and the model type.: [evidence span withheld — anchor status: external_doc]
- [readme] Model can be either `C16` or `C18`. `C18` takes into account that M+2 peak can come from the unlabeled palmitate + $2^{13}C$.: [evidence span withheld — anchor status: external_doc]
- [readme] p_mean: parameter of binomial distribution estimated from the distribution mean (for binomial $mean = pn$): [evidence span withheld — anchor status: external_doc]
- [discussion] if overall substrate usage is very low, such as in the hypoxia/normoxia experiment, it becomes impossible to reliably fit a binomial distribution and estimate the uptake fraction: "if overall substrate usage is very low, such as in the hypoxia/normoxia experiment, it becomes impossible to reliably fit a binomial"
- [methods] Fit binomial distribution to each fatty acid's isotopologue distribution: "Fit binomial distribution to each fatty acid's isotopologue distribution"
