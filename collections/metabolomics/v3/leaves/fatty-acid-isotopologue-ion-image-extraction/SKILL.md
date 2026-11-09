---
name: fatty-acid-isotopologue-ion-image-extraction
description: Use when MALDI-AIF imzML datasets contain 13C-labelled fatty acid isotopologues
  that METASPACE annotations miss, and ion images must be extracted per pixel or ablation
  mark before isotope-abundance correction and binomial fitting.
when_to_use_negative:
- When SpaceM output (spatiomolecular_adata.h5ad or spatiomolecular_matrix.csv) is
  already shared — the README explicitly says 'You don't need to download raw `imzML`
  files and rerun this step'
- When the analysis targets unlabelled fatty acids and only natural-isotope distributions
  are needed, since METASPACE annotations alone suffice
- When overall substrate usage is very low (e.g. the hypoxia/normoxia experiment),
  where the article states 'it becomes impossible to reliably fit a binomial distribution
  and estimate the uptake fraction'
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0610
tools:
- name: pyimzML
  role: Parses each imzML file and extracts ion images for all isotopologues of ions
    from the molecule list
  repo: https://github.com/alexandrovteam/pyimzML
- name: SpaceM
  role: Consumes the extracted ion images for image registration, microscopy correlation
    and ablation-mark attribution to segmented cells
- name: METASPACE
  role: Hosts the source datasets (project buglakova-2024); annotations are not used
    directly because they only cover natural isotope abundance
- name: 13C-SpaceM
  role: Provides the extract_isotopologue_ion_images.py script and downstream analysis
    scripts
  repo: https://github.com/Buglakova/13C-SpaceM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:f1704b592ec363ab43c86d1d4fe9981189375d665e2cfd82715fb6df8db8ca11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexandrovteam/pyimzML
  tools:
  - 13C-SpaceM
  - METASPACE
  - SpaceM
  - pyimzML
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# fatty-acid-isotopologue-ion-image-extraction

## Summary

Extracts per-pixel or per-ablation-mark ion images for all isotopologues of fatty acids from raw imzML MALDI imaging files using pyimzML, producing image matrices expected by the SpaceM pipeline or downstream binomial isotope-tracing models.

## When to use

Use when MALDI-AIF imzML datasets contain 13C-labelled fatty acid isotopologues that METASPACE annotations miss, and ion images must be extracted per pixel or ablation mark before isotope-abundance correction and binomial fitting.

## When NOT to use

- When SpaceM output (spatiomolecular_adata.h5ad or spatiomolecular_matrix.csv) is already shared — the README explicitly says 'You don't need to download raw `imzML` files and rerun this step'
- When the analysis targets unlabelled fatty acids and only natural-isotope distributions are needed, since METASPACE annotations alone suffice
- When overall substrate usage is very low (e.g. the hypoxia/normoxia experiment), where the article states 'it becomes impossible to reliably fit a binomial distribution and estimate the uptake fraction'

## Inputs

- Raw imzML imaging file (version 1.1.0) containing 13C6-glucose-labelled fatty acid isotopologues
- CSV molecule list of fatty acid isotopologues and their m/z values (e.g. analysis/ion_list_fa.csv)
- Output directory for extracted ion images

## Outputs

- Ion images (one per fatty acid isotopologue m/z) stored in the format expected by SpaceM
- Per-pixel isotopologue intensity matrices for downstream isotope-abundance correction and binomial fitting
- Spatial heatmap of fitted p values (labelling degree of cytosolic acetyl-CoA) when followed by the per-pixel pipeline

## How to apply

Parse each imzML file with pyimzML (version 1.1.0 format) rather than relying on METASPACE annotations, because METASPACE only generates isotopologue distributions based on natural isotope abundance and cannot supply labelled isotopologue peaks. Supply the script with a CSV molecule list such as `analysis/ion_list_fa.csv` enumerating all fatty acid isotopologues and their m/z values, together with the imzML file path and an output directory. The script extracts an ion image for every m/z of every listed isotopologue, storing images in the format expected by SpaceM's registration and segmentation steps. For mouse-brain tissue analysis (e.g. dataset 2023-09-25_ME_PattiLab_MouseBrain_4-4_010G_AIF_600-1600_30NCE_100-600_10umss_285x230_32at from the METASPACE project buglakova-2024), the same extraction feeds a per-pixel workflow: after extraction, apply natural isotope abundance correction, normalize each palmitate isotopologue distribution, and fit a binomial model (C16 or C18) per pixel to estimate the labelling probability p. Note the README states this step can be skipped when SpaceM outputs are shared.

## Related tools

- **pyimzML** (Parses each imzML file and extracts ion images for all isotopologues of ions from the molecule list) — https://github.com/alexandrovteam/pyimzML
- **SpaceM** (Consumes the extracted ion images for image registration, microscopy correlation and ablation-mark attribution to segmented cells)
- **METASPACE** (Hosts the source datasets (project buglakova-2024); annotations are not used directly because they only cover natural isotope abundance)
- **13C-SpaceM** (Provides the extract_isotopologue_ion_images.py script and downstream analysis scripts) — https://github.com/Buglakova/13C-SpaceM

## Examples

```
python analysis/brain/00_extract_ion_images.py --imzml raw.imzML --ion_list analysis/ion_list_fa.csv --out out_dir
```

## Evaluation signals

- The script runs without error on each imzML file and produces ion images for every m/z in the molecule list (e.g. analysis/ion_list_fa.csv)
- For brain-tissue data, pixel-wise palmitate binomial fits yield p values within the reported 0.45–0.70 range in tumour regions
- Resulting heatmap (PNG) and a spatially uniform TIC panel reproduce Figure 5 panels c and d
- Downstream natural isotope abundance correction and normalization run on the extracted matrices without missing isotopologue entries
- If SpaceM outputs are present, skipping this step yields identical downstream single-cell results

## Limitations

- The step must be rerun only if SpaceM outputs are absent; raw imzML download plus pyimzML extraction is otherwise redundant
- METASPACE-only annotations cannot provide labelled isotopologue distributions, so extraction from imzML is mandatory for 13C-tracing
- Per-pixel (tissue) analysis differs from per-cell (single-cell) analysis; downstream scripts differ (00_extract_ion_images.py vs extract_isotopologue_ion_images.py)
- In the in vivo mouse model, isotopic steady state was likely not reached within the 48-h labelling period, so labelling data need to be interpreted alongside pool size data

## Evidence

- [methods] pyimzML is used to extract ion images: "Normally mass spec peak annotations downloaded from [Metaspace](https://metaspace2020.eu/) are used in `SpaceM` pipeline, but Metaspace only generates isotopologue distributions based on natural"
- [methods] extract_isotopologue_ion_images.py is the extraction script: "used `extract_isotopologue_ion_images.py` script to parse each `imzML` file and produce ion images for all isotopologues of the ions from molecule list"
- [readme] Molecule list argument: "python sciso/extract_isotopologue_ion_images.py analysis/ion_list_fa.csv 20220322_AB_DKFZHypoxia/W2"
- [methods] Per-pixel workflow description: "For the tissue imaging analysis the analysis was done per pixel of the MALDI image instead of per cell. In each pixel we do the same analysis steps as for the single-cell data"
- [results] Spatial heterogeneity finding: [evidence span withheld — anchor status: not_found]
