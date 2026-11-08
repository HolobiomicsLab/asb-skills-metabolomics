---
name: spatial-heatmap-visualization
description: Use when you have per-pixel or per-cell fitted model parameters (e.g.
  binomial-fit p values from palmitate isotopologue distributions in imzML data) and
  need to visualise their spatial distribution across a tissue section with a TIC
  control panel.
when_to_use_negative:
- Data are already aggregated to single-cell readouts (spatiomolecular_adata.h5ad)
  rather than per-pixel MALDI intensities; single-cell plotting uses the anndata/Scanpy
  workflow instead
- Overall substrate usage is very low (e.g. hypoxia/normoxia experiment), so a binomial
  fit cannot be reliably obtained and no p heatmap can be produced
- The goal is absolute substrate contribution rather than a relative spatial map,
  since the method cannot quantify pool size
edam_operation: http://edamontology.org/operation_0526
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: pyimzML
  role: Parses the imzML file to extract per-pixel ion images for fatty acid isotopologues
    used to build the pixel grid and heatmap
  repo: https://github.com/alexandrovteam/pyimzML
- name: 13C-SpaceM
  role: Repository containing 00_extract_ion_images.py and the brain tissue pixel-wise
    analysis pipeline that produces the Figure 5 p heatmap
  repo: https://github.com/Buglakova/13C-SpaceM
- name: 13C_binomial
  role: Provides fit_binomial that fits the binomial distribution (C16/C18 models)
    to corrected isotopologue intensities, yielding the p values mapped onto the heatmap
  repo: https://github.com/Buglakova/13C_binomial
- name: METASPACE
  role: Source of the mouse brain MALDI-AIF dataset downloaded for pixel-wise heatmap
    rendering
  repo: https://metaspace2020.eu/project/buglakova-2024
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:485b4dc78afb0fcdcad65a6e40636552474a4bea9147855846b87c1dabe73d1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexandrovteam/pyimzML
  tools:
  - 13C-SpaceM
  - 13C_binomial
  - METASPACE
  - pyimzML
  edam_operation: http://edamontology.org/operation_0526
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# spatial-heatmap-visualization

## Summary

Render pixel-wise fitted model parameters (e.g. glucose labelling probability p of the cytosolic acetyl-CoA pool) as a spatial heatmap over a MALDI imaging tissue section, paired with a uniform total-ion-count panel to attribute spatial variation to biology rather than signal intensity.

## When to use

Use when you have per-pixel or per-cell fitted model parameters (e.g. binomial-fit p values from palmitate isotopologue distributions in imzML data) and need to visualise their spatial distribution across a tissue section with a TIC control panel.

## When NOT to use

- Data are already aggregated to single-cell readouts (spatiomolecular_adata.h5ad) rather than per-pixel MALDI intensities; single-cell plotting uses the anndata/Scanpy workflow instead
- Overall substrate usage is very low (e.g. hypoxia/normoxia experiment), so a binomial fit cannot be reliably obtained and no p heatmap can be produced
- The goal is absolute substrate contribution rather than a relative spatial map, since the method cannot quantify pool size

## Inputs

- imzML MALDI imaging file (e.g. 2023-09-25_ME_PattiLab_MouseBrain_4-4_010G_AIF_600-1600_30NCE_100-600_10umss_285x230_32at from Metaspace project buglakova-2024)
- ion_list_fa.csv listing fatty acid isotopologue m/z values
- Per-pixel normalized and natural-isotope-abundance-corrected isotopologue intensities for the fatty acid (e.g. palmitate)
- Per-pixel fitted binomial model parameters (p, model type C16 or C18) from fit_binomial

## Outputs

- Spatial heatmap (PNG) of fitted labelling probability p across the tissue section, e.g. Figure 5c
- Paired spatially uniform total ion count map on the same pixel grid, e.g. Figure 5d

## How to apply

After natural isotope abundance correction and normalization of the palmitate isotopologue distributions, fit the binomial distribution to every pixel's isotopologue intensities (using the fit_binomial function with C16 or C18 model depending on fatty acid chain length) to obtain the labelling probability p. Map the fitted p values back onto the MALDI image pixel grid using the imzML pixel coordinates obtained via pyimzML, and render them as a heatmap overlaying the tissue section; for the glioma xenograft this reproduces Figure 5 panels c and d showing p in the 0.45–0.70 range over tumour tissue. Alongside the p heatmap, produce a spatially uniform total ion count (TIC) map on the same pixel grid so that spatial heterogeneity in p can be attributed to biology rather than variation in signal intensity. Confirm that the layout mirrors the published panels: p map showing the 0.45–0.70 range within the tumour region and a uniform TIC panel.

## Related tools

- **pyimzML** (Parses the imzML file to extract per-pixel ion images for fatty acid isotopologues used to build the pixel grid and heatmap) — https://github.com/alexandrovteam/pyimzML
- **13C-SpaceM** (Repository containing 00_extract_ion_images.py and the brain tissue pixel-wise analysis pipeline that produces the Figure 5 p heatmap) — https://github.com/Buglakova/13C-SpaceM
- **13C_binomial** (Provides fit_binomial that fits the binomial distribution (C16/C18 models) to corrected isotopologue intensities, yielding the p values mapped onto the heatmap) — https://github.com/Buglakova/13C_binomial
- **METASPACE** (Source of the mouse brain MALDI-AIF dataset downloaded for pixel-wise heatmap rendering) — https://metaspace2020.eu/project/buglakova-2024

## Examples

```
python analysis/brain/00_extract_ion_images.py --raw path/to/2023-09-25_ME_PattiLab_MouseBrain_4-4_010G_AIF_600-1600_30NCE_100-600_10umss_285x230_32at.imzML --ion_list analysis/brain/ion_list_fa.csv --out outdir
```

## Evaluation signals

- Output heatmap file exists as a PNG at the expected path (Figure 5c equivalent)
- Fitted p values over the tumour region fall within the reported 0.45–0.70 range
- The TIC panel on the same pixel grid is spatially uniform, confirming heterogeneity is not driven by signal intensity
- Pixel grid dimensions match the imzML metadata (285x230 pixels at 10 µm spacing for the cited dataset)
- Overall layout visually matches the published Figure 5 panels c and d

## Limitations

- Only relative, not absolute, substrate contribution can be visualised because the lipogenic acetyl-CoA pool size cannot be quantified
- AIF-based release restricts analysis to fatty acids from lipid classes effectively ionized in negative mode (primarily acidic glycerophospholipids), so the heatmap covers only that subset
- The binomial fit and hence the p heatmap cannot be reliably produced when overall substrate usage is very low (e.g. hypoxia/normoxia experiment)

## Evidence

- [results] Pixel-wise binomial modelling: [evidence span withheld — anchor status: not_found]
- [readme] MalDI image per pixel analysis: "For the tissue imaging analysis the analysis was done per pixel of the MALDI image instead of per cell."
- [methods] Fit binomial distribution to each fatty acid: "Fit binomial distribution to each fatty acid's isotopologue distribution"
- [methods] pyimzML is used to extract ion images: [evidence span withheld — anchor status: not_found]
- [results] TIC uniform across sample: [evidence span withheld — anchor status: not_found]
