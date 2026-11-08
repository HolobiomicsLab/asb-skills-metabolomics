---
name: isotope-tracing-uptake-vs-synthesis-interpretation
description: Use when you have normalized, natural-abundance-corrected isotopologue
  distributions of fatty acids from 13C6-glucose tracer experiments and need to decide
  whether apparent label reflects de novo synthesis from glucose-derived acetyl-CoA
  or uptake of pre-existing fatty acids.
when_to_use_negative:
- When overall substrate usage is very low (e.g. hypoxia/normoxia experiment) — the
  binomial distribution cannot be reliably fit to estimate uptake fraction, so fall
  back to M+0/mean labelling comparison instead of quantitative uptake estimates.
- When absolute contribution to the lipogenic acetyl-CoA pool is required — the method
  provides only relative substrate contribution because the pool size cannot be quantified.
- When precise de novo synthesis vs uptake proportions are needed per fatty acid in
  vivo without prior determination of time to isotopic steady state — different fatty
  acid pools may reach steady state at different times and labelling must be interpreted
  alongside pool size data.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: 13C_binomial
  role: Fits a binomial model (C16 or C18) to corrected isotopologue distributions
    to obtain uptake fraction and distribution parameters for fatty acids
  repo: https://github.com/Buglakova/13C_binomial
- name: 13C-SpaceM
  role: Produces the single-cell normalized isotopologue anndata objects that serve
    as input for interpreting synthesis vs uptake per fatty acid
  repo: https://github.com/Buglakova/13C-SpaceM
- name: Scanpy
  role: Handles the pickled anndata single-cell objects in which per-cell isotopologue
    distributions are stored for interpretation
- name: Isocorrector
  role: External software whose output can be analyzed with the fit_binomial workflow
    (example_isocorrector_analysis.ipynb) for bulk isotopologue correction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:b46e5170020993efcfd03105cae04ff86001eba669499f46404ab018ebbb7eeb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Buglakova/13C_binomial
  tools:
  - 13C-SpaceM
  - 13C_binomial
  - Isocorrector
  - Scanpy
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# isotope-tracing-uptake-vs-synthesis-interpretation

## Summary

Interprets 13C6-glucose labelling of esterified fatty acids (isotopologue distributions from 13C-SpaceM or bulk MSI/LC-MS) to separate de novo synthesis from uptake, using binomial fitting for parameters and mean-fraction-labelling for qualitative discrimination between conditions.

## When to use

Use when you have normalized, natural-abundance-corrected isotopologue distributions of fatty acids from 13C6-glucose tracer experiments and need to decide whether apparent label reflects de novo synthesis from glucose-derived acetyl-CoA or uptake of pre-existing fatty acids.

## When NOT to use

- When overall substrate usage is very low (e.g. hypoxia/normoxia experiment) — the binomial distribution cannot be reliably fit to estimate uptake fraction, so fall back to M+0/mean labelling comparison instead of quantitative uptake estimates.
- When absolute contribution to the lipogenic acetyl-CoA pool is required — the method provides only relative substrate contribution because the pool size cannot be quantified.
- When precise de novo synthesis vs uptake proportions are needed per fatty acid in vivo without prior determination of time to isotopic steady state — different fatty acid pools may reach steady state at different times and labelling must be interpreted alongside pool size data.

## Inputs

- Normalized, natural-isotope-abundance-corrected isotopologue distributions for fatty acids (C14:0 myristate, C16:0 palmitate, C16:1 palmitoleate, C18:0 stearate, C18:1 oleate) per cell from 13C-SpaceM anndata objects, or per-pixel from MALDI imaging, or bulk from LC-MS/MS
- Lipogenic acetyl-CoA fractional labelling per cell/condition (for ACLY knockdown or hypoxia/normoxia comparisons)
- Condition metadata (normoxic vs hypoxic populations, ACLY knockdown vs control, tumour vs surrounding brain)
- Fatty acid chain length information to select the C16 vs C18 binomial model
- For in vivo data: measured fatty acid pool sizes, since 48-h 13C6-glucose feeding likely does not reach isotopic steady state

## Outputs

- Per-fatty-acid qualitative call: fraction attributed to de novo glucose-dependent synthesis vs uptake
- Fitted binomial model parameters per fatty acid (uptake fraction, p parameter, mean isotopologue distribution, fit_success flag) when fitting is valid
- Condition-level comparison of mean fractional labelling and M+0 fractions (e.g. palmitate M+0 increase in hypoxia)
- Annotated interpretation caveats: relative-not-absolute substrate contribution, isotopic steady-state status, MALDI-AIF lipid-class selectivity

## How to apply

Load the per-cell or per-pixel anndata object produced by the 13C-SpaceM workflow and inspect the mean fractional labelling (weighted isotopologue mean with M+0 excluded) of each fatty acid under each condition. Treat increases in palmitate M+0 fraction as evidence of reduced de novo synthesis — as in hypoxia vs normoxia where palmitate M+0 clearly separates populations, while ACLY knockdown shows only a minor M+0 increase despite reduced acetyl-CoA fractional labelling. When overall substrate usage is sufficiently high (e.g. the ACLYkd experiment), fit a binomial model to the corrected isotopologue distribution (C16 or C18 model; for C18 account for the M+2 peak receiving contribution from unlabeled palmitate + 2x13C) using fit_binomial in 13C_binomial; do NOT attempt binomial fitting when substrate usage is very low, as in the hypoxia/normoxia experiment. Because the methodology measures only the relative — not absolute — contribution to the lipogenic acetyl-CoA pool, and uptake is inferred as label absent from de novo synthesis, interpret labelling data alongside pool-size data when isotopic steady state is uncertain.

## Related tools

- **13C_binomial** (Fits a binomial model (C16 or C18) to corrected isotopologue distributions to obtain uptake fraction and distribution parameters for fatty acids) — https://github.com/Buglakova/13C_binomial
- **13C-SpaceM** (Produces the single-cell normalized isotopologue anndata objects that serve as input for interpreting synthesis vs uptake per fatty acid) — https://github.com/Buglakova/13C-SpaceM
- **Scanpy** (Handles the pickled anndata single-cell objects in which per-cell isotopologue distributions are stored for interpretation)
- **Isocorrector** (External software whose output can be analyzed with the fit_binomial workflow (example_isocorrector_analysis.ipynb) for bulk isotopologue correction)

## Examples

```
python -c "from fit_binomial import fit_binomial; import numpy as np; I = np.loadtxt('data/IsoCorrectoR_ACLY.csv', delimiter=',', skiprows=1); res = fit_binomial(I, 'C16'); res.to_csv('fit_binomial_result.csv')"
```

## Evaluation signals

- For C18 model fits, the palmitate M+2 contribution from unlabeled C16:0 + 2x13C must be accounted for — verify fit residuals for the C18:1 oleate isotopologue pattern are acceptable before trusting uptake estimates.
- Hypoxia vs normoxia sanity check: palmitate M+0 fraction must show a strong increase in hypoxic cells and the two populations should separate on that metric, as reported in the manuscript discussion.
- ACLY knockdown sanity check: fractional labelling of lipogenic acetyl-CoA should decrease while palmitate M+0 increases only modestly — a large M+0 shift would indicate misinterpretation.
- fit_binomial_result.csv must record fit_success=1 for valid fits; exclude samples where optimization failed before interpreting uptake parameters.
- Confirm the interpretation is explicitly framed as relative (not absolute) substrate contribution — any output claiming absolute uptake/synthesis percentage exceeds the method's stated scope.

## Limitations

- If overall substrate usage is very low (e.g. hypoxia/normoxia experiment), it becomes impossible to reliably fit a binomial distribution and estimate the uptake fraction, so fitting was restricted to ACLYkd data.
- The methodology provides only relative, not absolute, substrate contribution to the lipogenic acetyl-CoA pool, as the pool size cannot be quantified.
- Different fatty acid pools may reach isotopic steady state at different times; accurate de novo synthesis vs uptake quantification may require prior determination of time to isotopic steady state.
- In the in vivo mouse model, isotopic steady state was likely not reached within the 48-h labelling period, so labelling data must be interpreted alongside pool size data.
- MALDI-AIF samples only a subset of lipids effectively ionized in negative mode (primarily acidic glycerophospholipids); abundant species like phosphatidylcholines, triglycerides and sphingomyelins are underrepresented, which must be considered when estimating fatty acid uptake as the uptake proportion may differ by lipid class.

## Evidence

- [readme] binomial fitting for uptake estimation: [evidence span withheld — anchor status: external_doc]
- [readme] C18 model M+2 correction: [evidence span withheld — anchor status: external_doc]
- [discussion] hypoxia palmitate M+0 separation: "Spatial single-cell isotope tracing by 13C-SpaceM clearly separated the two cell populations based on the strong increase in palmitate M + 0 fraction in hypoxic cells."
- [discussion] ACLYkd minor M+0 change: "silencing of ACLY caused a reduction in the fractional labelling of acetyl-CoA from glucose but only a minor increase in the M + 0 fraction of palmitate."
- [discussion] binomial fitting fails at low substrate usage: "if overall substrate usage is very low, such as in the hypoxia/normoxia experiment, it becomes impossible to reliably fit a binomial distribution and estimate the uptake fraction"
- [discussion] relative not absolute contribution: "our methodology can only provide information on relative substrate contribution to the lipogenic acetyl-CoA pool as opposed to absolute contribution, as the size of this pool cannot be quantified."
- [discussion] isotopic steady state caveat in vivo: [evidence span withheld — anchor status: not_found]
- [discussion] lipid-class selectivity caveat: "the selectivity of the MALDI-AIF method has to be considered for the estimation of fatty acid uptake, as the proportion of fatty acids derived from uptake may differ depending on lipid class."
