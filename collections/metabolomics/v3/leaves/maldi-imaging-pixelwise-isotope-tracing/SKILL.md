---
name: maldi-imaging-pixelwise-isotope-tracing
description: Use when MALDI imaging data from 13C6-glucose-labeled tissue must be
  analyzed per pixel instead of per cell, to recover spatial maps of isotopologue
  distributions and binomial-fitted labelling probability p for fatty acids.
when_to_use_negative:
- When single-cell segmentation from co-registered microscopy is available — use the
  SpaceM single-cell workflow instead of pixel-wise analysis.
- 'When overall substrate usage is very low (e.g. hypoxia/normoxia experiments): ''it
  becomes impossible to reliably fit a binomial distribution and estimate the uptake
  fraction''.'
- When the goal is absolute quantification of substrate contribution — the method
  only provides relative contribution because the size of the lipogenic acetyl-CoA
  pool cannot be quantified.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: pyimzML
  role: parses each imzML file to extract ion images for all isotopologues of the
    ions from the molecule list, storing them in the format expected downstream
  repo: https://github.com/alexandrovteam/pyimzML
- name: 13C-SpaceM
  role: provides the analysis/brain/00_extract_ion_images.py script and the pixel-level
    workflow (isotope correction, normalization, binomial fitting) for tissue imaging
    data
  repo: https://github.com/Buglakova/13C-SpaceM
- name: 13C_binomial
  role: fit_binomial function fits the binomial model (C16 or C18) to normalized,
    corrected isotopologue intensities per pixel to estimate p and uptake
  repo: https://github.com/Buglakova/13C_binomial
- name: METASPACE
  role: hosting the mouse brain MALDI imaging datasets downloaded for pixel-wise analysis
  repo: https://metaspace2020.eu/project/buglakova-2024
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:f494799d803f04fd6203183c1766a1ae4fa0edccd3cfb33a53cefd34d11de6d8
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# maldi-imaging-pixelwise-isotope-tracing

## Summary

Analyzes MALDI imaging isotope-tracing data (MALDI-AIF after 13C6-glucose labeling) on a per-pixel basis: natural isotope abundance correction of each ion's isotopologue distribution followed by binomial-model fitting to estimate substrate contribution to the lipogenic acetyl-CoA pool, producing spatial maps of heterogeneity in fatty acid synthesis.

## When to use

Use when MALDI imaging data from 13C6-glucose-labeled tissue must be analyzed per pixel instead of per cell, to recover spatial maps of isotopologue distributions and binomial-fitted labelling probability p for fatty acids.

## When NOT to use

- When single-cell segmentation from co-registered microscopy is available — use the SpaceM single-cell workflow instead of pixel-wise analysis.
- When overall substrate usage is very low (e.g. hypoxia/normoxia experiments): 'it becomes impossible to reliably fit a binomial distribution and estimate the uptake fraction'.
- When the goal is absolute quantification of substrate contribution — the method only provides relative contribution because the size of the lipogenic acetyl-CoA pool cannot be quantified.

## Inputs

- raw imzML file from MALDI-AIF imaging of 13C6-glucose-labeled tissue (e.g. dataset 2023-09-25_ME_PattiLab_MouseBrain_4-4_010G_AIF_600-1600_30NCE_100-600_10umss_285x230_32at from Metaspace project buglakova-2024)
- ion list CSV (e.g. analysis/brain/ion_list_fa.csv) enumerating fatty acid isotopologue m/z values

## Outputs

- per-pixel ion images for all fatty acid isotopologues
- natural isotope abundance-corrected and normalized isotopologue distributions per pixel
- per-pixel binomial fit results (labelling probability p, fit success) for each fatty acid
- spatial heatmap of fitted p values across the tissue section (e.g. Figure 5 panel c) and accompanying total ion count map (panel d)

## How to apply

Parse the raw imzML file with pyimzML (analysis/brain/00_extract_ion_images.py) to extract per-pixel ion images for all fatty acid isotopologues listed in ion_list_fa.csv. For every pixel, perform natural isotope abundance correction on each ion's isotopologue distribution, then normalize the corrected distributions for each fatty acid. Fit a binomial distribution to each fatty acid's normalized isotopologue distribution per pixel — using the C16 or C18 model from the 13C_binomial code, where the C18 model accounts for the M+2 peak coming from unlabeled palmitate plus two 13C — to estimate the labelling probability p. The choice of pixel-level analysis over single-cell analysis is deliberate for tissue sections where single-cell segmentation is not available at the current MALDI spatial resolution. Finally, render spatial heatmaps of the fitted p values (e.g. glucose contribution to cytosolic acetyl-CoA ranging 0.45–0.70 in tumour regions) alongside a total ion count map to confirm that heterogeneity is not an artifact of signal intensity.

## Related tools

- **pyimzML** (parses each imzML file to extract ion images for all isotopologues of the ions from the molecule list, storing them in the format expected downstream) — https://github.com/alexandrovteam/pyimzML
- **13C-SpaceM** (provides the analysis/brain/00_extract_ion_images.py script and the pixel-level workflow (isotope correction, normalization, binomial fitting) for tissue imaging data) — https://github.com/Buglakova/13C-SpaceM
- **13C_binomial** (fit_binomial function fits the binomial model (C16 or C18) to normalized, corrected isotopologue intensities per pixel to estimate p and uptake) — https://github.com/Buglakova/13C_binomial
- **METASPACE** (hosting the mouse brain MALDI imaging datasets downloaded for pixel-wise analysis) — https://metaspace2020.eu/project/buglakova-2024

## Examples

```
python analysis/brain/00_extract_ion_images.py <raw_imzml> analysis/brain/ion_list_fa.csv <output_dir>
```

## Evaluation signals

- Output heatmap PNG exists and the tumour-region p values fall within the reported 0.45–0.70 range.
- Spatially uniform total ion count map accompanies the p heatmap, confirming heterogeneity is not driven by TIC differences.
- fit_binomial_result.csv reports fit_success = 1 (convergence reached) for each pixel/fatty acid fit.
- Model type recorded in fit_binomial_result.csv (C16 vs C18) matches the fatty acid chain length being fit.
- Mean of the isotopologue distribution reported by the fit (sum(i*I(M+i))/sum(I(M+i))) is consistent with the fitted p (p_mean = p*n for binomial).

## Limitations

- Isotopic steady state was likely not reached for fatty acids within the 48-h labelling period in the in vivo mouse model, so labelling data must be interpreted alongside pool size data.
- The method provides only relative, not absolute, substrate contribution to the lipogenic acetyl-CoA pool because the pool size cannot be quantified.
- MALDI-AIF restricts analysis to fatty acids from a subset of lipid classes effectively ionized in negative mode, primarily acidic glycerophospholipids.
- Spatial resolution of current MALDI imaging limits extension to single-cell isotope tracing in tissue sections; would require increased spatial resolution and single-cell segmentation.

## Evidence

- [readme] For the tissue imaging analysis the analysis was done per pixel of the MALDI image instead of per cell. In each pixel we do the same analysis steps as for the single-cell data: Perform natural isotope abundance correction for each ion's isotopologue distribution; Fit binomial distribution to each fatty acid's isotopologue distribution: "For the tissue imaging analysis the analysis was done per pixel of the MALDI image instead of per cell.
In each pixel we do the same analysis steps as for the single-cell data:
- Perform natural"
- [results] Pixel-wise binomial modelling of palmitate isotopologues revealed significant spatial intra-tumoural heterogeneity, with glucose contribution to the cytosolic acetyl-CoA pool (p) ranging from 0.45 to 0.70: [evidence span withheld — anchor status: not_found]
- [readme] Function fit_binomial in fit_binomial.py takes as input numpy array of normalized and corrected for natural isotope abundance intensities for each isotopologue peak of the fatty acid and the model type: [evidence span withheld — anchor status: external_doc]
- [discussion] our methodology can only provide information on relative substrate contribution to the lipogenic acetyl-CoA pool as opposed to absolute contribution, as the size of this pool cannot be quantified: "our methodology can only provide information on relative substrate contribution to the lipogenic acetyl-CoA pool as opposed to absolute contribution, as the size of this pool cannot be quantified."
- [discussion] isotopic steady state was likely not reached for fatty acids within the 48-h labelling period38. Thus, labelling data need to be interpreted under consideration of pool size data.: "isotopic steady state was likely not reached for fatty acids within the 48-h labelling period38. Thus, labelling data need to be interpreted under consideration of pool size data."
