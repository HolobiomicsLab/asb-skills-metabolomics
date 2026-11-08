---
name: pseudo-bulk-aggregation-of-single-cell-maldi-data
description: Use when single-cell MALDI-MS isotope tracing data (e.g. 13C-SpaceM anndata
  objects with per-cell isotopologue fractions M+0..M+16) must be collapsed per condition
  to compare with bulk LC-MS or to visualize condition-level shifts.
when_to_use_negative:
- When overall substrate usage is very low (e.g. hypoxia/normoxia experiment), the
  binomial fit cannot reliably estimate uptake fraction — the model was only used
  for ACLYkd data.
- When per-cell heterogeneity is the target (pseudo-bulking intentionally obscures
  single-cell heterogeneity in acetyl-CoA labelling that bulk analysis hides).
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: 13C-SpaceM
  role: Provides the single-cell anndata objects of per-cell isotopologue distributions
    that are pseudo-bulked
  repo: https://github.com/Buglakova/13C-SpaceM
- name: Scanpy
  role: Stores and reads the pickled anndata single-cell objects used for aggregation
- name: 13C-binomial
  role: Fits binomial model to pseudo-bulk isotopologue distributions (fit_binomial
    in fit_binomial.py, models C16/C18)
  repo: https://github.com/Buglakova/13C_binomial
- name: SpaceM
  role: Upstream segmentation/registration producing the single-cell data prior to
    aggregation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:7e91b6673482d77c9e9642a6047976a3d2415d3e9ab569875aa18fe7b7f293cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Buglakova/13C-SpaceM
  tools:
  - 13C-SpaceM
  - 13C-binomial
  - Scanpy
  - SpaceM
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# pseudo-bulk-aggregation-of-single-cell-maldi-data

## Summary

Aggregates quality-controlled single-cell isotopologue distributions from MALDI imaging (13C-SpaceM anndata objects) into pseudo-bulk per-condition distributions by median-averaging, enabling comparison with bulk LC-MS measurements. In the ACLYkd experiment this reveals the shift of the palmitate isotopologue distribution toward lower mass isotopologues after ACLY silencing.

## When to use

Use when single-cell MALDI-MS isotope tracing data (e.g. 13C-SpaceM anndata objects with per-cell isotopologue fractions M+0..M+16) must be collapsed per condition to compare with bulk LC-MS or to visualize condition-level shifts.

## When NOT to use

- When overall substrate usage is very low (e.g. hypoxia/normoxia experiment), the binomial fit cannot reliably estimate uptake fraction — the model was only used for ACLYkd data.
- When per-cell heterogeneity is the target (pseudo-bulking intentionally obscures single-cell heterogeneity in acetyl-CoA labelling that bulk analysis hides).

## Inputs

- Scanpy pickled anndata objects with per-cell isotopologue distributions for fatty acids (output of sciso/sc_analysis.py per slide/well)
- Cell-level metadata for condition assignment (control, ACLYkd oligo 1, ACLYkd oligo 2)
- QC thresholds for cell size, number of detected ions, and total ion intensity

## Outputs

- Pseudo-bulk isotopologue distribution table per condition (e.g. palmitate_isotopologue_distributions_aclykd.csv)
- Condition-level comparison plot of isotopologue shifts (e.g. palmitate_isotopologue_shift_aclykd.png)
- Optionally, 13C-binomial fit parameters per condition (uptake, p, mean) from fit_binomial_result.csv

## How to apply

Load the Scanpy pickled anndata objects produced by sciso/sc_analysis.py for each well of the slide (e.g. dataset 20220411_AB_DKFZACLYac, control and ACLYkd oligo 1/2 wells). Filter cells by size, number of detected ions and total ion intensity (quality control as in the analysis/ACLYkd notebooks). For each fatty acid in each cell use the naturally-isotope-corrected, normalized isotopologue distribution (M+0 ... M+16 fractional abundances), then aggregate cells per condition (control, ACLYkd oligo 1, ACLYkd oligo 2) by median-averaging the fractional abundances. Optionally fit the 13C-binomial model (C16/C18 from Buglakova/13C_binomial) to each condition's distribution to quantify labelled acetyl-CoA fraction and uptake. Compare pseudo-bulk distributions across conditions and validate against expected values, e.g. control pseudo-bulk palmitate M+0 fraction of ~0.6 versus bulk ~0.3.

## Related tools

- **13C-SpaceM** (Provides the single-cell anndata objects of per-cell isotopologue distributions that are pseudo-bulked) — https://github.com/Buglakova/13C-SpaceM
- **Scanpy** (Stores and reads the pickled anndata single-cell objects used for aggregation)
- **13C-binomial** (Fits binomial model to pseudo-bulk isotopologue distributions (fit_binomial in fit_binomial.py, models C16/C18)) — https://github.com/Buglakova/13C_binomial
- **SpaceM** (Upstream segmentation/registration producing the single-cell data prior to aggregation)

## Examples

```
python sciso/sc_analysis.py $data_dir/20220411_AB_DKFZACLYac/slide1/spacem_data $data_dir/20220411_AB_DKFZACLYac/slide1/anndata
```

## Evaluation signals

- Control pseudo-bulk palmitate M+0 fraction ≈ 0.6, consistent with bulk LC-MS control value of ~0.3 discrepancy check described in the manuscript
- ACLYkd oligo 1 and 2 pseudo-bulk distributions show a clear shift toward lower mass (lower M+i) isotopologues relative to control
- Median (not mean) aggregation over QC-passing cells yields stable fractional abundances summing to 1 across M+0..M+16 peaks
- 13C-binomial fit converges (fit_success=1) for ACLYkd pseudo-bulk distributions with sensible p and uptake parameters

## Limitations

- Pseudo-bulking hides substantial single-cell heterogeneity in fractional labelling of lipogenic acetyl-CoA (e.g. differing shRNA efficiency within ACLY-silenced populations) that the method otherwise reveals
- MALDI-AIF only captures fatty acids from lipid classes effectively ionized in negative mode (primarily acidic glycerophospholipids); uptake proportions may differ by lipid class
- Only relative, not absolute, substrate contribution to the lipogenic acetyl-CoA pool is obtainable since pool size cannot be quantified
- Different fatty acid pools may reach isotopic steady state at different times; in the in vivo mouse model steady state was likely not reached in 48 h

## Evidence

- [methods] QC filtering and median aggregation: "After single-cell analysis data from all slides is aggregated together and quality control is performed, filtering out cells by size and number and total intensity of detected ions."
- [methods] Normalization step: "Normalizing isotopologue distributions for each fatty acid for each cell"
- [methods] Natural abundance correction: "Natural isotope abundance correction for each fatty acid in each ablation mark"
- [abstract] Heterogeneity hidden by bulk: "This revealed substantial single-cell heterogeneity in labelling of the lipogenic acetyl-CoA pool and in relative fatty acid uptake versus synthesis hidden in bulk analyses."
- [discussion] ACLYkd shift finding: "silencing of ACLY caused a reduction in the fractional labelling of acetyl-CoA from glucose but only a minor increase in the M + 0 fraction of palmitate."
- [readme] Binomial fit usage: [evidence span withheld — anchor status: external_doc]
- [readme] Fit outputs: [evidence span withheld — anchor status: external_doc]
- [discussion] Heterogeneity in ACLYkd: "We also revealed substantial heterogeneity in fractional labelling of lipogenic acetyl-CoA within the ACLY-silenced cell populations, suggesting differences in efficiency of the shRNA sequences"
- [readme] sc_analysis invocation: "python sciso/sc_analysis.py $data_dir/20220411_AB_DKFZACLYac/slide1/spacem_data $data_dir/20220411_AB_DKFZACLYac/slide1/anndata"
