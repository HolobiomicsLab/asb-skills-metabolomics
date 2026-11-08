---
name: single-cell-isotope-tracing-data-analysis
description: Use when MALDI imaging MS data paired with microscopy segmentation has
  been acquired after 13C6-glucose labelling and the goal is per-cell fatty-acid isotopologue
  distributions and uptake/synthesis fractions.
when_to_use_negative:
- When overall substrate usage is very low, e.g. the hypoxia/normoxia experiment —
  the binomial fit cannot reliably estimate the uptake fraction and was only applied
  to ACLYkd data
- When input is bulk isotope-tracing data rather than single-cell/segmented MALDI
  data — use the simpler 13C_binomial companion workflow instead of the per-cell SpaceM
  pipeline
- When aiming for absolute substrate contribution to the lipogenic acetyl-CoA pool
  — the method only provides relative contributions because pool size cannot be quantified
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: 13C-SpaceM
  role: Spatial single-cell isotope tracing method and repository whose scripts (extract_isotopologue_ion_images.py,
    sc_analysis.py) perform isotopologue extraction, correction, normalization, and
    QC aggregation
  repo: https://github.com/Buglakova/13C-SpaceM
- name: 13C_binomial
  role: Companion repository providing fit_binomial (C16/C18 models) to fit a binomial
    distribution to normalized, natural-abundance-corrected isotopologue intensities
    and estimate uptake fractions
  repo: https://github.com/Buglakova/13C_binomial
- name: pyimzML
  role: Parser used to extract ion images from imzML files for all isotopologues of
    ions in the molecule list before SpaceM processing
  repo: https://github.com/alexandrovteam/pyimzML
- name: Scanpy
  role: Loads and manipulates the pickled anndata single-cell objects for group splitting,
    QC, and figure generation
  repo: https://github.com/Buglakova/13C-SpaceM
- name: SpaceM
  role: Closed-source software used upstream for image registration, segmentation,
    and overlaying MALDI ablation marks with microscopy cell masks; its outputs are
    the inputs to this skill
- name: conda/mamba
  role: Environment creation via environment.yml (conda env create -f environment.yml;
    mamba solver recommended for slow solves)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:d463e10ab66d3681e06b0752af51e18b4c1d805f8bdb797b051f90514854dcbc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Buglakova/13C-SpaceM
  tools:
  - 13C-SpaceM
  - 13C_binomial
  - Scanpy
  - SpaceM
  - conda/mamba
  - pyimzML
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# single-cell-isotope-tracing-data-analysis

## Summary

End-to-end single-cell isotope tracing analysis for 13C-SpaceM data: extracting fatty-acid isotopologue ion images, correcting natural isotope abundance, normalizing isotopologue distributions per cell, and fitting binomial models to estimate relative fatty-acid uptake versus de novo synthesis at single-cell resolution.

## When to use

Use when MALDI imaging MS data paired with microscopy segmentation has been acquired after 13C6-glucose labelling and the goal is per-cell fatty-acid isotopologue distributions and uptake/synthesis fractions.

## When NOT to use

- When overall substrate usage is very low, e.g. the hypoxia/normoxia experiment — the binomial fit cannot reliably estimate the uptake fraction and was only applied to ACLYkd data
- When input is bulk isotope-tracing data rather than single-cell/segmented MALDI data — use the simpler 13C_binomial companion workflow instead of the per-cell SpaceM pipeline
- When aiming for absolute substrate contribution to the lipogenic acetyl-CoA pool — the method only provides relative contributions because pool size cannot be quantified

## Inputs

- SpaceM output directories per well (spatiomolecular_adata.h5ad, ablation_mark_analysis, cell_segmentation_external, overlap_analysis2, ion_image_registration)
- Raw imzML files with MALDI imaging MS data from 13C6-glucose-labelled cells
- Molecule/ion list CSV (e.g. analysis/ion_list_fa.csv) with fatty-acid isotopologues
- QC-filtered, aggregated single-cell anndata objects (Biostudies downloads for Hypoxia/Normoxia and ACLYkd experiments)
- Normalized GFP-intensity classification per cell (control vs knockdown groups)

## Outputs

- Per-cell, per-fatty-acid normalized isotopologue distributions stored in anndata objects
- Fitted binomial model results per cell (fit_binomial_result.csv: model C16/C18, fit_success, uptake, p, mean, p_mean)
- Binomial predictions per sample (fit_binomial_prediction.csv)
- Uptake-fraction histograms per fatty acid per condition with marked distribution modes
- Manuscript-figure plots for Figures 2-4 from single-cell anndata objects

## How to apply

Start from SpaceM output directories (per-well structures with ablation_mark_analysis and cell_segmentation_external subfolders); because Metaspace only annotates natural-abundance isotopologues, use pyimzML via extract_isotopologue_ion_images.py to parse each imzML file and generate ion images for all isotopologues of fatty acids in the molecule list. Run python sciso/sc_analysis.py per slide, which performs natural isotope abundance correction for each fatty acid in each ablation mark, averages signal per cell by taking the median for each annotated m/z value, and normalizes isotopologue distributions per fatty acid per cell, saving an anndata object per slide. Aggregate slides and perform quality control, filtering cells by size, number of detected ions, and total ion intensity, then classify cells by normalized GFP intensity (e.g. control vs ACLYkd oligo 2). Load the QC-filtered aggregated anndata objects with Scanpy and fit a binomial distribution (C16 or C18 model) to each fatty acid's normalized, corrected isotopologue distribution to estimate the uptake fraction, using the fit_binomial function from the 13C_binomial repository; visualize per-group uptake histograms and verify reported peak values.

## Related tools

- **13C-SpaceM** (Spatial single-cell isotope tracing method and repository whose scripts (extract_isotopologue_ion_images.py, sc_analysis.py) perform isotopologue extraction, correction, normalization, and QC aggregation) — https://github.com/Buglakova/13C-SpaceM
- **13C_binomial** (Companion repository providing fit_binomial (C16/C18 models) to fit a binomial distribution to normalized, natural-abundance-corrected isotopologue intensities and estimate uptake fractions) — https://github.com/Buglakova/13C_binomial
- **pyimzML** (Parser used to extract ion images from imzML files for all isotopologues of ions in the molecule list before SpaceM processing) — https://github.com/alexandrovteam/pyimzML
- **Scanpy** (Loads and manipulates the pickled anndata single-cell objects for group splitting, QC, and figure generation) — https://github.com/Buglakova/13C-SpaceM
- **SpaceM** (Closed-source software used upstream for image registration, segmentation, and overlaying MALDI ablation marks with microscopy cell masks; its outputs are the inputs to this skill)
- **conda/mamba** (Environment creation via environment.yml (conda env create -f environment.yml; mamba solver recommended for slow solves))

## Examples

```
python sciso/sc_analysis.py $data_dir/20220411_AB_DKFZACLYac/slide1/spacem_data $data_dir/20220411_AB_DKFZACLYac/slide1/anndata
```

## Evaluation signals

- Palmitate uptake histogram peak reproduces ~0.55 in control vs ~0.61 in ACLYkd oligo 2; oleate ~0.85 vs ~0.95, with ACLYkd oleate distribution skewed and unimodal
- fit_binomial_result.csv reports fit_success = 1 (optimization convergence) and the correct model type (C16 or C18) for each sample
- Uptake histograms for control and ACLYkd palmitate populations are unimodal and symmetric as reported
- sc_analysis.py output anndata objects are produced per slide (one joined object for all wells on a slide) before aggregation and QC
- For C18, the fit accounts for the M+2 peak coming from unlabeled palmitate + 2 x 13C, and mean = p*n invariant holds for the binomial parameter estimated from the distribution mean

## Limitations

- Binomial fitting cannot reliably estimate uptake fraction when overall substrate usage is very low (e.g. hypoxia/normoxia); it was only used for the ACLYkd data
- MALDI-AIF only detects fatty acids from lipids effectively ionized in negative mode (primarily acidic glycerophospholipids); phosphatidylcholines, triglycerides, and sphingomyelins are underrepresented, and uptake proportion may differ by lipid class
- Method provides only relative, not absolute, substrate contribution to the lipogenic acetyl-CoA pool because pool size cannot be quantified
- Different fatty acid pools may reach isotopic steady state at different times; in the in vivo mouse model, isotopic steady state was likely not reached within the 48-h labelling period
- Pickled anndata objects may fail to load with mismatched pandas/anndata/pickle/Scanpy versions — consult environment_dump.yml

## Evidence

- [readme] sc_analysis performs natural isotope abundance correction, median per-cell averaging, and isotopologue normalization: "This script performs the following steps: - Natural isotope abundance correction for each fatty acid in each ablation mark - Averaging of signal in each cell by taking a median for each annotated m/z"
- [readme] extract_isotopologue_ion_images.py parses imzML files for all isotopologues: "then used `extract_isotopologue_ion_images.py` script to parse each `imzML` file and produce ion images for all isotopologues of the ions from molecule list"
- [readme] Metaspace only generates natural-abundance isotopologue distributions, motivating custom extraction: "but Metaspace only generates isotopologue distributions based on natural isotope abundance, therefore here after generating all possible isotopologues for a given molecule, [pyimzML] is used to"
- [methods] QC filters cells by size, ion count, and total intensity, then GFP-based classification: "After single-cell analysis data from all slides is aggregated together and quality control is performed, filtering out cells by size and number and total intensity of detected ions. Subsequently the"
- [readme] fit_binomial takes normalized, corrected intensities and C16/C18 model type: [evidence span withheld — anchor status: external_doc]
- [abstract] 13C-SpaceM revealed single-cell heterogeneity in uptake versus synthesis hidden in bulk analyses: "This revealed substantial single-cell heterogeneity in labelling of the lipogenic acetyl-CoA pool and in relative fatty acid uptake versus synthesis hidden in bulk analyses."
- [discussion] Binomial fitting only applied to ACLYkd, not hypoxia/normoxia: "if overall substrate usage is very low, such as in the hypoxia/normoxia experiment, it becomes impossible to reliably fit a binomial distribution and estimate the uptake fraction, therefore it was"
