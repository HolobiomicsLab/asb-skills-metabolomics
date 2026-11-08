---
name: de-novo-fatty-acid-synthesis-fraction-quantification
description: Use when analysing U-13C-glucose labelling imzML imaging data (isotopologue
  series M+0 to M+16 of fatty acids) to estimate de novo synthesis versus uptake fractions
  and acetyl-CoA pool labelling per pixel or cell.
when_to_use_negative:
- Overall substrate usage is very low (e.g. hypoxia/normoxia experiment) — the binomial
  fit cannot reliably estimate the uptake fraction, as reported for the hypoxia data
  where the model was only applied to ACLYkd data
- Isotopic steady state was not reached within the labelling window (e.g. 48-h in
  vivo 13C6-glucose diet) — labelling fractions then need joint interpretation with
  pool size data and are not absolute substrate contributions
- Input is already aggregated bulk isotope-tracing data without spatial/pixel or single-cell
  resolution — the single-cell heterogeneity signal this skill targets is hidden in
  bulk analyses
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: pyimzML
  role: Parses imzML files to extract ion images for all isotopologues of fatty acids
    from the molecule list
  repo: https://github.com/alexandrovteam/pyimzML
- name: 13C-SpaceM
  role: Repository with the full pixel- and single-cell-level isotopologue correction,
    binomial fitting, and figure-reproduction workflow
  repo: https://github.com/Buglakova/13C-SpaceM
- name: 13C_binomial
  role: Standalone fit_binomial.py that fits the binomial model (C16 or C18) to corrected,
    normalized isotopologue distributions
  repo: https://github.com/Buglakova/13C_binomial
- name: METASPACE
  role: Hosts the mouse brain MALDI imaging data project for download and annotation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:bb9bcbec0ac141e66ac44b94914967b0046ad83c04ce93c8da3bd5d8885ffa20
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexandrovteam/pyimzML
  tools:
  - 13C-SpaceM
  - 13C_binomial
  - METASPACE
  - pyimzML
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# de-novo-fatty-acid-synthesis-fraction-quantification

## Summary

Quantifies the fraction of esterified fatty acids (e.g. palmitate, stearate) produced by de novo synthesis from glucose-derived 13C by correcting MALDI-imaging isotopologue distributions for natural isotope abundance and computing 1-(M+0)/Σ(M+n) per pixel or per cell, as in 13C-SpaceM glioma tumour analysis.

## When to use

Use when analysing U-13C-glucose labelling imzML imaging data (isotopologue series M+0 to M+16 of fatty acids) to estimate de novo synthesis versus uptake fractions and acetyl-CoA pool labelling per pixel or cell.

## When NOT to use

- Overall substrate usage is very low (e.g. hypoxia/normoxia experiment) — the binomial fit cannot reliably estimate the uptake fraction, as reported for the hypoxia data where the model was only applied to ACLYkd data
- Isotopic steady state was not reached within the labelling window (e.g. 48-h in vivo 13C6-glucose diet) — labelling fractions then need joint interpretation with pool size data and are not absolute substrate contributions
- Input is already aggregated bulk isotope-tracing data without spatial/pixel or single-cell resolution — the single-cell heterogeneity signal this skill targets is hidden in bulk analyses

## Inputs

- imzML MALDI imaging file (e.g. AIF 600–1600 m/z, 10 µm pixel size, 285x230 pixels)
- Ion/isotopologue list CSV (e.g. ion_list_fa.csv with fatty acids and M+0 to M+16 isotopologues)
- Isocorrector-output normalized, natural-abundance-corrected isotopologue intensities (for 13C_binomial fit_binomial)
- Tumour/region annotation or segmentation masks for spatial aggregation

## Outputs

- Per-pixel or per-cell de novo synthesis fractions 1-(M+0)/Σ(M+n) for each fatty acid
- Fitted binomial parameters (uptake, p, mean, fit_success) per sample/fatty acid
- Spatial maps of de novo fractions (manuscript Figure 5 panels c–d, Figure 6 panel a)
- Per-fatty-acid fraction table CSV (brain_denovo_fractions.csv)

## How to apply

Parse the imzML file with pyimzML (e.g. via extract_isotopologue_ion_images.py or analysis/brain/00_extract_ion_images.py with an ion list such as ion_list_fa.csv) to extract ion images for all isotopologues (M+0 to M+16+) of each fatty acid. Apply natural isotope abundance correction to each fatty acid's isotopologue distribution per pixel or ablation mark, then normalize the distributions. Fit a binomial distribution to the corrected isotopologue distribution — using the C16 model for palmitate/palmitoleate and the C18 model for stearate/oleate, since the C18 model accounts for the M+2 peak arising from unlabeled palmitate + 2×13C via elongation — to estimate the fractional labelling of the lipogenic acetyl-CoA pool. Compute the de novo synthesis fraction 1-(M+0)/Σ(M+n) per fatty acid per pixel, derive M+2-enrichment metrics for C18 fatty acids as elongation readouts, map fractions spatially, and aggregate over regions of interest (tumour vs surrounding brain). Export per-fatty-acid fractions to CSV (e.g. brain_denovo_fractions.csv).

## Related tools

- **pyimzML** (Parses imzML files to extract ion images for all isotopologues of fatty acids from the molecule list) — https://github.com/alexandrovteam/pyimzML
- **13C-SpaceM** (Repository with the full pixel- and single-cell-level isotopologue correction, binomial fitting, and figure-reproduction workflow) — https://github.com/Buglakova/13C-SpaceM
- **13C_binomial** (Standalone fit_binomial.py that fits the binomial model (C16 or C18) to corrected, normalized isotopologue distributions) — https://github.com/Buglakova/13C_binomial
- **METASPACE** (Hosts the mouse brain MALDI imaging data project for download and annotation)

## Examples

```
python analysis/brain/00_extract_ion_images.py analysis/brain/ion_list_fa.csv 2023-09-25_ME_PattiLab_MouseBrain_4-4_010G_AIF_600-1600_30NCE_100-600_10umss_285x230_32at
```

## Evaluation signals

- Reproduced panels show strong induction of de novo synthesis of SFAs (palmitate, stearate) in tumour tissue versus surrounding brain
- MUFAs (palmitoleate, oleate) show a higher M+0 (uptake) proportion than palmitate, consistent with high MUFA demand
- M+2 enrichment is observed in stearate and oleate, consistent with elongation of palmitate/palmitoleate chains
- fit_success = 1 (optimization converged) for binomial fits; fitted p from optimization agrees with p_mean estimated from the distribution mean (mean = pn for binomial)
- Predicted isotopologue distributions from fit_binomial_prediction.csv overlay the observed bar distributions in the diagnostic plots

## Limitations

- MALDI-AIF in negative mode only samples fatty acids from a subset of effectively ionized lipid classes (primarily acidic glycerophospholipids); abundant species like phosphatidylcholines, triglycerides and sphingomyelins are underrepresented, so uptake proportions may differ by lipid class
- The method gives only relative, not absolute, substrate contribution to the lipogenic acetyl-CoA pool because pool size cannot be quantified
- Isotopic steady state was likely not reached within the 48-h labelling period, so labelling data must be interpreted alongside pool size data
- If overall substrate usage is very low (hypoxia/normoxia), a binomial distribution cannot be reliably fit to estimate the uptake fraction

## Evidence

- [methods] Compute the de novo synthesis fraction 1-(M+0)/Σ(M+n) for each fatty acid per pixel: [evidence span withheld — anchor status: not_found]
- [readme] readme binomial fitting: [evidence span withheld — anchor status: external_doc]
- [readme] readme C18 model rationale: [evidence span withheld — anchor status: external_doc]
- [methods] pyimzML extraction: "used `extract_isotopologue_ion_images.py` script to parse each `imzML` file"
- [discussion] binomial fit limitation: "if overall substrate usage is very low, such as in the hypoxia/normoxia experiment, it becomes impossible to reliably fit a binomial distribution and estimate the uptake fraction"
