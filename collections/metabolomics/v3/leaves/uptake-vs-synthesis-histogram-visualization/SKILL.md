---
name: uptake-vs-synthesis-histogram-visualization
description: Use when single-cell isotopologue data (e.g. Scanpy anndata objects from
  13C-SpaceM) have been fitted with a binomial uptake model and you need per-cell
  relative uptake fraction histograms per fatty acid, with modes marked and compared
  across conditions.
when_to_use_negative:
- When overall substrate usage is very low (e.g. the hypoxia/normoxia experiment),
  the binomial distribution cannot be reliably fitted and no uptake fraction estimate
  or histogram should be produced
- When the goal is absolute (not relative) substrate contribution to the lipogenic
  acetyl-CoA pool — the method only provides relative contributions because pool size
  cannot be quantified
- When fatty acid pools have not reached isotopic steady state (e.g. the 48-h in vivo
  labelling), labelling-based uptake histograms must be interpreted alongside pool
  size data rather than reported directly
edam_operation: http://edamontology.org/operation_2928
tools:
- name: 13C-SpaceM
  role: Source of the QC-filtered single-cell anndata objects and manuscript_figures
    notebooks that generate the per-fatty-acid uptake histograms
  repo: https://github.com/Buglakova/13C-SpaceM
- name: 13C_binomial
  role: Provides fit_binomial (C16/C18 models) that estimates the uptake parameter
    from corrected, normalized isotopologue intensities before histogram plotting
  repo: https://github.com/Buglakova/13C_binomial
- name: Scanpy
  role: Loads the pickled anndata objects holding per-cell isotopologue distributions
    and condition labels
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:3454c368a6fdeb2eddb6bf23f7fe452156a778e84974121d5296c7dda61c17e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Buglakova/13C-SpaceM
  tools:
  - 13C-SpaceM
  - 13C_binomial
  - Scanpy
  edam_operation: http://edamontology.org/operation_2928
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# uptake-vs-synthesis-histogram-visualization

## Summary

Visualize single-cell fatty-acid uptake-versus-de novo-synthesis fractions as per-fatty-acid histograms, computed by binomial-model fitting of isotopologue distributions, to reveal population-level heterogeneity that bulk analyses hide.

## When to use

Use when single-cell isotopologue data (e.g. Scanpy anndata objects from 13C-SpaceM) have been fitted with a binomial uptake model and you need per-cell relative uptake fraction histograms per fatty acid, with modes marked and compared across conditions.

## When NOT to use

- When overall substrate usage is very low (e.g. the hypoxia/normoxia experiment), the binomial distribution cannot be reliably fitted and no uptake fraction estimate or histogram should be produced
- When the goal is absolute (not relative) substrate contribution to the lipogenic acetyl-CoA pool — the method only provides relative contributions because pool size cannot be quantified
- When fatty acid pools have not reached isotopic steady state (e.g. the 48-h in vivo labelling), labelling-based uptake histograms must be interpreted alongside pool size data rather than reported directly

## Inputs

- QC-filtered, aggregated single-cell anndata object (Scanpy pickled anndata) with median-aggregated, natural-isotope-abundance-corrected, normalized isotopologue distributions per fatty acid per cell
- Per-cell condition labels from normalized GFP-intensity classification (control vs hypoxic vs ACLYkd oligo 2)
- Binomial model type per fatty acid (C16 or C18)

## Outputs

- Results table of per-cell relative uptake fractions for each fatty acid (e.g. fit_binomial_result.csv with model, fit_success, uptake, p, mean)
- Per-fatty-acid histograms of fitted relative uptake fractions for each condition, with distribution modes marked
- Overlay of original isotopologue data (bars) and fitted model prediction (line) per sample

## How to apply

Load the QC-filtered, aggregated single-cell anndata objects (produced by the analysis/hypoxia and analysis/ACLYkd notebooks) with Scanpy, and split cells into groups (e.g. normoxic control, hypoxic, ACLYkd oligo 2) using the normalized GFP-intensity classification stored per cell. For each fatty acid (myristate, palmitate, palmitoleate, stearate, oleate) and each cell, fit a binomial distribution to the natural-isotope-abundance-corrected, normalized isotopologue distribution to estimate the fractional labelling of the lipogenic acetyl-CoA pool and the relative uptake fraction, using the fit_binomial function with model type C16 or C18 (C18 accounts for the M+2 peak coming from unlabeled palmitate + 2×13C). Store per-cell uptake fractions in a results table, then plot per-fatty-acid histograms for each condition with matplotlib, marking distribution modes. Validate the figures against reported values: control uptake fractions fall at 60–90% with palmitate peaking at ~0.55 and palmitoleate/oleate peaking higher; for ACLYkd oligo 2, palmitate peaks ~0.61 and oleate ~0.95 with a skewed unimodal distribution.

## Related tools

- **13C-SpaceM** (Source of the QC-filtered single-cell anndata objects and manuscript_figures notebooks that generate the per-fatty-acid uptake histograms) — https://github.com/Buglakova/13C-SpaceM
- **13C_binomial** (Provides fit_binomial (C16/C18 models) that estimates the uptake parameter from corrected, normalized isotopologue intensities before histogram plotting) — https://github.com/Buglakova/13C_binomial
- **Scanpy** (Loads the pickled anndata objects holding per-cell isotopologue distributions and condition labels)

## Examples

```
python fit_binomial.py --input data/IsoCorrectoR_ACLY.csv --outdir data/acly_results  # then plot per-cell uptake histograms with matplotlib from manuscript_figures notebooks
```

## Evaluation signals

- Control-cell uptake histograms fall within the reported 60–90% uptake range (10–40% de novo synthesis)
- Palmitate histogram peak reproduces ~0.55 in control and ~0.61 in ACLYkd oligo 2 cells; oleate ~0.85 (control) vs ~0.95 (ACLYkd oligo 2), with ACLYkd oleate distribution skewed and unimodal
- Palmitoleate/oleate histograms peak at higher uptake fractions than palmitate, matching the reported MUFA-demand finding
- fit_binomial_result.csv shows fit_success = 1 for samples whose histograms are included in the figures
- Overlay plots show fitted model prediction (line) closely following the original isotopologue bars for each sample

## Limitations

- Binomial fitting and uptake estimation become impossible when overall substrate usage is very low, such as in the hypoxia/normoxia experiment
- The methodology provides only relative, not absolute, substrate contribution to the lipogenic acetyl-CoA pool because the pool size cannot be quantified
- MALDI-AIF restricts the analysis to fatty acids from a subset of lipids effectively ionized in negative mode (primarily acidic glycerophospholipids), and uptake proportion may differ by lipid class

## Evidence

- [readme] fit_binomial input and model types: [evidence span withheld — anchor status: external_doc]
- [readme] C18 model accounts for M+2 contribution: [evidence span withheld — anchor status: external_doc]
- [readme] manuscript_figures notebooks reproduce plots from anndata objects: "`manuscript_figures` contains notebooks that allow to reproduce plots for each figure panel based on the `anndata` objects produced in the previous step"
- [methods] isotopologue distributions normalized per fatty acid per cell: "Normalizing isotopologue distributions for each fatty acid for each cell"
- [discussion] higher uptake for palmitoleate/oleate than palmitate: "with a higher proportion of uptake for palmitoleate and oleate compared with palmitate, suggesting high demand for MUFAs."
