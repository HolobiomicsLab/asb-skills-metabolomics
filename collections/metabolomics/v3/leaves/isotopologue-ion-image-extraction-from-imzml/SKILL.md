---
name: isotopologue-ion-image-extraction-from-imzml
description: Use when MALDI imaging data are stored as imzML and you need ion images
  for all isotopologues of specific molecules (e.g. fatty acids 20:3, 20:4, 20:5,
  22:4, 22:5) from an ion list, rather than only the annotated natural-abundance peaks
  from METASPACE.
when_to_use_negative:
- If you already have the shared SpaceM output (ion images and single-cell data),
  re-extracting from raw imzML is unnecessary per the README
- If you need only natural-abundance METASPACE annotations, since METASPACE already
  generates those without this extraction step
- If the aim is bulk/MSI isotopologue modelling without imaging — use the simpler
  13C_binomial repository instead
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: pyimzML
  role: Python parser used by extract_isotopologue_ion_images.py to read each imzML
    file and extract ion images for all isotopologues of the ions from the molecule
    list
  repo: https://github.com/alexandrovteam/pyimzML
- name: 13C-SpaceM
  role: Provides the extract_isotopologue_ion_images.py script, the ion list CSV,
    and the conda environment (sciso_env) in which the extraction runs
  repo: https://github.com/Buglakova/13C-SpaceM
- name: METASPACE
  role: Source of the imzML datasets (project buglakova-2024) and usual peak annotations;
    cannot generate labelled-isotopologue distributions, which motivates this extraction
    step
- name: SpaceM
  role: Consumes the extracted isotopologue ion images in its expected format after
    image registration and cell segmentation
- name: conda/mamba
  role: Environment management for the sciso_env environment; mamba solver recommended
    when conda solving is too slow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:ad2e98abc17ad47181f287add8147c0f352cc1d584f6baa6be0efb72abe0187f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexandrovteam/pyimzML
  tools:
  - 13C-SpaceM
  - METASPACE
  - Scanpy
  - SpaceM
  - conda
  - conda/mamba
  - mamba
  - pyimzML
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Isotopologue ion image extraction from imzML

## Summary

Parses imzML MALDI imaging files with pyimzML to extract per-pixel ion images for every isotopologue (M+0, M+1, M+2, ...) of target fatty acids, because METASPACE annotations only cover natural-abundance isotopologue distributions and cannot supply 13C-labelled peaks. It is the first step of the 13C-SpaceM isotopologue workflow before natural isotope abundance correction and binomial fitting.

## When to use

Use when MALDI imaging data are stored as imzML and you need ion images for all isotopologues of specific molecules (e.g. fatty acids 20:3, 20:4, 20:5, 22:4, 22:5) from an ion list, rather than only the annotated natural-abundance peaks from METASPACE.

## When NOT to use

- If you already have the shared SpaceM output (ion images and single-cell data), re-extracting from raw imzML is unnecessary per the README
- If you need only natural-abundance METASPACE annotations, since METASPACE already generates those without this extraction step
- If the aim is bulk/MSI isotopologue modelling without imaging — use the simpler 13C_binomial repository instead

## Inputs

- imzML MALDI imaging files (version 1.1.0) from the METASPACE project buglakova-2024 (mouse brain experiments)
- Ion list CSV specifying target fatty acids and their isotopologues (e.g. analysis/ion_list_fa.csv)
- SpaceM working directory per well (e.g. 20220322_AB_DKFZHypoxia/W2) containing registration/segmentation outputs

## Outputs

- Per-isotopologue ion images for every molecule in the ion list, stored in the format expected by SpaceM
- Downstream input for sc_analysis.py producing per-slide anndata objects with normalized, natural-abundance-corrected isotopologue distributions

## How to apply

Install the 13C-SpaceM conda environment (preferably with the mamba solver) and clone the repository. First run SpaceM (image registration and segmentation) up to the METASPACE annotation step as usual; since METASPACE only generates isotopologue distributions based on natural isotope abundance, generate all possible isotopologues for each molecule in the ion list CSV and then run extract_isotopologue_ion_images.py, which parses each imzML file with pyimzML and produces ion images in the format expected by SpaceM. For mouse brain tissue analysis, the extraction is done per pixel of the MALDI image and an alternative entry point is analysis/brain/00_extract_ion_images.py. The extracted isotopologue ion images are then handed to sc_analysis.py (natural isotope abundance correction, per-cell median averaging, isotopologue distribution normalization). Note that raw imzML files need not be reprocessed if the SpaceM output is shared.

## Related tools

- **pyimzML** (Python parser used by extract_isotopologue_ion_images.py to read each imzML file and extract ion images for all isotopologues of the ions from the molecule list) — https://github.com/alexandrovteam/pyimzML
- **13C-SpaceM** (Provides the extract_isotopologue_ion_images.py script, the ion list CSV, and the conda environment (sciso_env) in which the extraction runs) — https://github.com/Buglakova/13C-SpaceM
- **METASPACE** (Source of the imzML datasets (project buglakova-2024) and usual peak annotations; cannot generate labelled-isotopologue distributions, which motivates this extraction step)
- **SpaceM** (Consumes the extracted isotopologue ion images in its expected format after image registration and cell segmentation)
- **conda/mamba** (Environment management for the sciso_env environment; mamba solver recommended when conda solving is too slow)

## Examples

```
python sciso/extract_isotopologue_ion_images.py analysis/ion_list_fa.csv 20220322_AB_DKFZHypoxia/W2
```

## Evaluation signals

- The extraction produces ion images for all possible isotopologues of every ion in the ion list CSV, not just natural-abundance peaks
- Downstream isotopologue distributions can be normalized per fatty acid after natural isotope abundance correction, indicating the images were parsed correctly
- For the glioma dataset, M+2 fractional abundance should be enriched in tumour pixels for 20:3, 20:4, 22:4 and 22:5, but 20:5 (omega-3 branch) should show no label incorporation
- If SpaceM output is shared, the extracted images should match analysis/00_extract_ion_images.ipynb examples without needing raw imzML re-parsing

## Limitations

- METASPACE only generates isotopologue distributions based on natural isotope abundance, so labelled isotopologues must be extracted manually via pyimzML
- MALDI-AIF only samples a subset of lipids ionized in negative mode; abundant species such as phosphatidylcholines, triglycerides and sphingomyelins are underrepresented, restricting analysis primarily to fatty acids from acidic glycerophospholipids
- Environment version conflicts with Scanpy pickled anndata objects (pandas/anndata/pickle/scanpy combinations) can break downstream steps; environment_dump.yml is provided for troubleshooting

## Evidence

- [readme] normally mass spec peak annotations downloaded from Metaspace are used in SpaceM pipeline, but Metaspace only generates isotopologue distributions based on natural isotope abundance: "Normally mass spec peak annotations downloaded from [Metaspace](https://metaspace2020.eu/) are used in `SpaceM` pipeline, but Metaspace only generates isotopologue distributions based on natural"
- [readme] pyimzML is used to extract ion images and store in the format expected by SpaceM: "here after generating all possible isotopologues for a given molecule, [pyimzML](https://github.com/alexandrovteam/pyimzML) is used to extract ion images and store in the format expected by `SpaceM`"
- [methods] then used extract_isotopologue_ion_images.py script to parse each imzML file: "then used `extract_isotopologue_ion_images.py` script to parse each `imzML` file"
- [readme] Data for mouse brain experiments can be downloaded from Metaspace project buglakova-2024: "Data for mouse brain experiments can be downloaded from [Metaspace project](https://metaspace2020.eu/project/buglakova-2024)."
- [discussion] Fatty acids analysed using the MALDI-AIF method applied here are derived from a subset of lipids that are effectively ionized in the negative mode: "Fatty acids analysed using the MALDI-AIF method applied here are derived from a subset of lipids that are effectively ionized in the negative mode. Some abundant lipid species, including"
