---
name: essential-fatty-acid-elongation-analysis
description: Use when analysing imzML MALDI imaging data from 13C6-glucose tracer
  studies of brain/tumour tissue to measure elongation/desaturation of essential fatty
  acids (20:3, 20:4, 20:5, 22:4, 22:5) and detect branch-specific (omega-3 vs omega-6)
  labelling.
when_to_use_negative:
- The analysis goal is absolute substrate contribution to the lipogenic acetyl-CoA
  pool — the method provides only relative contributions because the pool size cannot
  be quantified
- Overall substrate usage is very low (e.g. the hypoxia/normoxia setting) — the binomial
  fit cannot reliably estimate uptake fractions in that regime
- Data are single-cell culture data rather than tissue — then the per-cell SpaceM
  single-cell workflow (sc_analysis.py) is appropriate instead of the per-pixel tissue
  analysis
edam_operation: http://edamontology.org/operation_3632
tools:
- name: 13C-SpaceM
  role: Provides the extract_isotopologue_ion_images.py script and the sciso workflow
    for parsing imzML files, isotope correction, normalization, and per-pixel tissue
    analysis
  repo: https://github.com/Buglakova/13C-SpaceM
- name: pyimzML
  role: Parses imzML files to extract ion images for all isotopologues of the fatty
    acids from the molecule list
  repo: https://github.com/alexandrovteam/pyimzML
- name: 13C_binomial
  role: Fits a binomial model (C16 or C18) to the normalized, natural-isotope-abundance-corrected
    isotopologue distribution of each fatty acid
  repo: https://github.com/Buglakova/13C_binomial
- name: METASPACE
  role: Hosts the mouse brain experimental imzML data for download (project buglakova-2024)
  repo: https://metaspace2020.eu/project/buglakova-2024
- name: Scanpy
  role: Stores and manages single-cell/pixel results as pickled anndata objects within
    the 13C-SpaceM workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:058f82b5027d709d83e94fe1877f3ad2b80f8f63a6f55e2a07f8a32ef595dcd4
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
  edam_operation: http://edamontology.org/operation_3632
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# essential-fatty-acid-elongation-analysis

## Summary

Per-pixel isotopologue analysis of 13C6-glucose-labelled essential fatty acid derivatives in mouse brain MALDI imaging data, quantifying M+2 (glucose-derived two-carbon elongation) enrichment to compare omega-3 versus omega-6 essential fatty acid processing in tumours versus surrounding brain.

## When to use

Use when analysing imzML MALDI imaging data from 13C6-glucose tracer studies of brain/tumour tissue to measure elongation/desaturation of essential fatty acids (20:3, 20:4, 20:5, 22:4, 22:5) and detect branch-specific (omega-3 vs omega-6) labelling.

## When NOT to use

- The analysis goal is absolute substrate contribution to the lipogenic acetyl-CoA pool — the method provides only relative contributions because the pool size cannot be quantified
- Overall substrate usage is very low (e.g. the hypoxia/normoxia setting) — the binomial fit cannot reliably estimate uptake fractions in that regime
- Data are single-cell culture data rather than tissue — then the per-cell SpaceM single-cell workflow (sc_analysis.py) is appropriate instead of the per-pixel tissue analysis

## Inputs

- Mouse brain imzML MALDI imaging files from the Metaspace project (https://metaspace2020.eu/project/buglakova-2024)
- Ion list CSV of fatty acid isotopologues (e.g. analysis/ion_list_fa.csv) covering FA 20:3, 20:4, 20:5, 22:4, 22:5
- Tumour-vs-surrounding-brain region annotation per pixel

## Outputs

- Per-pixel, natural-isotope-corrected, normalized isotopologue distributions for each target fatty acid
- M+2 fractional abundance per fatty acid per region (tumour vs surrounding brain)
- CSV results table with isotopologue distributions and tumour-vs-brain comparison
- Optional binomial fit results (uptake, p, mean; fit_success flag) per fatty acid

## How to apply

Download the mouse brain imzML files from the METASPACE project (buglakova-2024) listed in the paper's methods. Parse each imzML file with the 13C-SpaceM `extract_isotopologue_ion_images.py` script (built on pyimzML), extracting ion images for all isotopologues of the target fatty acids 20:3, 20:4, 20:5, 22:4 and 22:5 from the ion list CSV (e.g. `analysis/ion_list_fa.csv`). Work per pixel of the MALDI image rather than per cell, applying natural isotope abundance correction to each fatty acid's isotopologue distribution, then normalize the isotopologue distributions for each fatty acid in tumour and surrounding brain regions. Optionally fit a binomial model (C16 or C18; C18 accounts for M+2 coming from unlabeled palmitate + 2x13C) using the 13C_binomial `fit_binomial` function on the normalized, corrected intensities. Finally compare M+2 fractional abundance between tumour and surrounding brain, expecting M+2 enrichment for 20:3, 20:4, 22:4 and 22:5 but not for 20:5, and write isotopologue distributions plus the tumour-vs-brain comparison to a CSV results table.

## Related tools

- **13C-SpaceM** (Provides the extract_isotopologue_ion_images.py script and the sciso workflow for parsing imzML files, isotope correction, normalization, and per-pixel tissue analysis) — https://github.com/Buglakova/13C-SpaceM
- **pyimzML** (Parses imzML files to extract ion images for all isotopologues of the fatty acids from the molecule list) — https://github.com/alexandrovteam/pyimzML
- **13C_binomial** (Fits a binomial model (C16 or C18) to the normalized, natural-isotope-abundance-corrected isotopologue distribution of each fatty acid) — https://github.com/Buglakova/13C_binomial
- **METASPACE** (Hosts the mouse brain experimental imzML data for download (project buglakova-2024)) — https://metaspace2020.eu/project/buglakova-2024
- **Scanpy** (Stores and manages single-cell/pixel results as pickled anndata objects within the 13C-SpaceM workflow)

## Examples

```
python sciso/extract_isotopologue_ion_images.py analysis/ion_list_fa.csv 20220322_AB_DKFZHypoxia/W2
```

## Evaluation signals

- M+2 isotopologue fractions of 20:3, 20:4, 22:4 and 22:5 are enriched in tumour tissue relative to surrounding brain, while 20:5 (PUFA exclusive to the omega-3 branch) shows no label incorporation
- Natural isotope abundance correction applied to every fatty acid's isotopologue distribution in every pixel before normalization
- Normalized isotopologue distributions sum to 1 per fatty acid per pixel
- Binomial fits report fit_success = 1 and model type (C16 or C18) recorded per sample; for C18 the M+2 peak contribution from unlabeled palmitate + 2x13C is modelled
- CSV results table contains both isotopologue distributions and the tumour-vs-brain M+2 comparison

## Limitations

- MALDI-AIF restricts the analysis to fatty acids from lipid classes effectively ionized in negative mode, primarily acidic glycerophospholipids; phosphatidylcholines, triglycerides and sphingomyelins are underrepresented
- Isotopic steady state was likely not reached for fatty acids within the 48-h labelling period, so labelling data must be interpreted alongside pool size data
- Uptake proportion estimates may differ depending on lipid class due to MALDI-AIF lipid-class selectivity

## Evidence

- [abstract] abstract finding: "we found higher glucose-dependent synthesis of saturated fatty acids and increased elongation of essential fatty acids in tumours compared with healthy brains."
- [methods] workflow parsing step: "then used `extract_isotopologue_ion_images.py` script to parse each `imzML` file"
- [readme] tissue per-pixel analysis: "For the tissue imaging analysis the analysis was done per pixel of the MALDI image instead of per cell."
- [readme] data source: "Data for mouse brain experiments can be downloaded from [Metaspace project](https://metaspace2020.eu/project/buglakova-2024)."
- [discussion] tumour SFA induction: "Isotopologue analysis revealed a strong induction of synthesis of SFAs, myristate, palmitate and stearate, in tumour tissue compared with the surrounding brain"
