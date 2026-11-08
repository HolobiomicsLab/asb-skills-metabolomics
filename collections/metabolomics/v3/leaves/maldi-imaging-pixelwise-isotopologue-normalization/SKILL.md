---
name: maldi-imaging-pixelwise-isotopologue-normalization
description: Use when processing MALDI imaging mass spectrometry isotope-tracing data
  (imzML files, e.g. 13C6-glucose-labelled tissue) at pixel rather than single-cell
  level, to obtain corrected, normalized isotopologue distributions for fatty acids
  per pixel.
when_to_use_negative:
- When analysing single-cell (per-cell) data from SpaceM output rather than raw tissue
  MALDI images — use the per-cell sc_analysis.py workflow with natural abundance correction,
  median signal averaging per cell, and per-cell normalization instead.
- When overall substrate usage is very low (e.g. hypoxia/normoxia experiments), where
  the article states it becomes impossible to reliably fit a binomial distribution
  and estimate the uptake fraction.
- When absolute substrate contributions to the lipogenic acetyl-CoA pool are needed
  — the method provides only relative contributions since the pool size cannot be
  quantified.
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: 13C-SpaceM
  role: Provides extract_isotopologue_ion_images.py (pyimzML-based) to parse imzML
    files and extract ion images for all isotopologues of ions from the molecule list
  repo: https://github.com/Buglakova/13C-SpaceM
- name: pyimzML
  role: Parser used to extract ion images from imzML files in the format expected
    by SpaceM
  repo: https://github.com/alexandrovteam/pyimzML
- name: 13C_binomial
  role: fit_binomial function fits a C16 or C18 binomial model to each fatty acid's
    natural-abundance-corrected, normalized isotopologue distribution
  repo: https://github.com/Buglakova/13C_binomial
- name: METASPACE
  role: Source of the mouse brain MALDI imaging dataset (project buglakova-2024) and
    peak annotations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:bb3c6ba501d9f64d26dfcdb50d43419cfe9eafce535758242a55a03157dd0df5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Buglakova/13C-SpaceM
  tools:
  - 13C-SpaceM
  - 13C_binomial
  - METASPACE
  - Scanpy
  - conda
  - mamba
  - pyimzML
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# maldi-imaging-pixelwise-isotopologue-normalization

## Summary

Per-pixel analysis of MALDI imaging isotope-tracing data: natural isotope abundance correction and normalization of fatty acid isotopologue distributions at each pixel of an imzML image, enabling spatial comparison of 13C-glucose-derived labelling between tumour and surrounding brain tissue.

## When to use

Use when processing MALDI imaging mass spectrometry isotope-tracing data (imzML files, e.g. 13C6-glucose-labelled tissue) at pixel rather than single-cell level, to obtain corrected, normalized isotopologue distributions for fatty acids per pixel.

## When NOT to use

- When analysing single-cell (per-cell) data from SpaceM output rather than raw tissue MALDI images — use the per-cell sc_analysis.py workflow with natural abundance correction, median signal averaging per cell, and per-cell normalization instead.
- When overall substrate usage is very low (e.g. hypoxia/normoxia experiments), where the article states it becomes impossible to reliably fit a binomial distribution and estimate the uptake fraction.
- When absolute substrate contributions to the lipogenic acetyl-CoA pool are needed — the method provides only relative contributions since the pool size cannot be quantified.

## Inputs

- imzML MALDI imaging files (e.g. mouse brain dataset from METASPACE project buglakova-2024)
- Ion list CSV (e.g. ion_list_fa.csv) specifying target fatty acids and their isotopologues (e.g. FA 20:3, 20:4, 20:5, 22:4, 22:5)
- Python environment from 13C-SpaceM environment.yml (sciso_env, with pyimzML, Scanpy, numpy)

## Outputs

- Per-pixel natural-abundance-corrected and normalized isotopologue distributions for each fatty acid
- Per-pixel binomial fit results (uptake, p, mean; CSV tables such as fit_binomial_result.csv and fit_binomial_prediction.csv)
- CSV results table comparing isotopologue fractions (e.g. M+2) between tumour and surrounding brain regions

## How to apply

For tissue imaging, run the analysis per pixel of the MALDI image instead of per cell, applying the same steps as the single-cell workflow: parse each imzML file (e.g. with the 13C-SpaceM extract_isotopologue_ion_images.py script built on pyimzML) to extract ion images for all isotopologues of the target fatty acids from the ion list CSV; apply natural isotope abundance correction to each ion's isotopologue distribution in each pixel; then normalize the isotopologue distributions for each fatty acid. Optionally fit a binomial distribution (C16 or C18 model from the 13C_binomial fit_binomial function) to each fatty acid's corrected distribution — the C18 model accounts for the M+2 peak arising from unlabeled palmitate + 2×13C. Finally compare fractional isotopologue abundances (e.g. M+2, reflecting glucose-derived two-carbon elongation) between regions of interest such as tumour versus surrounding brain and write results to a CSV table.

## Related tools

- **13C-SpaceM** (Provides extract_isotopologue_ion_images.py (pyimzML-based) to parse imzML files and extract ion images for all isotopologues of ions from the molecule list) — https://github.com/Buglakova/13C-SpaceM
- **pyimzML** (Parser used to extract ion images from imzML files in the format expected by SpaceM) — https://github.com/alexandrovteam/pyimzML
- **13C_binomial** (fit_binomial function fits a C16 or C18 binomial model to each fatty acid's natural-abundance-corrected, normalized isotopologue distribution) — https://github.com/Buglakova/13C_binomial
- **METASPACE** (Source of the mouse brain MALDI imaging dataset (project buglakova-2024) and peak annotations)

## Examples

```
python sciso/extract_isotopologue_ion_images.py analysis/ion_list_fa.csv 20220322_AB_DKFZHypoxia/W2
```

## Evaluation signals

- Natural isotope abundance correction is applied to every ion's isotopologue distribution in every pixel before normalization — normalized distributions should sum to 1 per fatty acid per pixel.
- M+2 enrichment appears for elongation/desaturation products (e.g. 20:3, 20:4, 22:4, 22:5) in tumour regions, while 20:5 (omega-3 branch PUFA) shows no label incorporation, matching the reported finding.
- Binomial fit reports fit_success=1 (convergence) for fitted samples; the C18 model is used for fatty acids where the M+2 peak can come from unlabeled palmitate + 2×13C.
- Corrected per-pixel mean of the isotopologue distribution equals sum(i×I(M+i))/sum(I(M+i)) with M+0 (and M+2 for C18) set to 0, matching the 13C_binomial output definition.
- Downstream tumour-vs-surrounding-brain comparison reproduces higher glucose-dependent elongation of essential fatty acids in tumours than in healthy brains.

## Limitations

- MALDI-AIF only samples a subset of lipids ionized in negative mode — abundant species (phosphatidylcholines, triglycerides, sphingomyelins) are underrepresented, and analysis is restricted mainly to fatty acids from acidic glycerophospholipids.
- Binomial fitting cannot reliably estimate uptake fraction when overall substrate usage is very low.
- In the 48-h in vivo 13C6-glucose labelling, isotopic steady state was likely not reached for fatty acids, so labelling data must be interpreted alongside pool size data.
- Lipid-class selectivity of MALDI-AIF means the proportion of fatty acids derived from uptake may differ depending on lipid class.

## Evidence

- [readme] For the tissue imaging analysis the analysis was done per pixel of the MALDI image instead of per cell.: "For the tissue imaging analysis the analysis was done per pixel of the MALDI image instead of per cell."
- [readme] In each pixel we do the same analysis steps: "In each pixel we do the same analysis steps as for the single-cell data:
- Perform natural isotope abundance correction for each ion's isotopologue distribution
- Fit binomial distribution to each"
- [readme] C18 takes into account: [evidence span withheld — anchor status: external_doc]
- [methods] pyimzML is used to extract ion images: "here after generating all possible isotopologues for a given molecule, [pyimzML](https://github.com/alexandrovteam/pyimzML) is used to extract ion images and store in the format expected by `SpaceM`."
- [methods] Data for mouse brain experiments: "Data for mouse brain experiments can be downloaded from [Metaspace project](https://metaspace2020.eu/project/buglakova-2024)."
