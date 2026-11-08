---
name: isotopologue-distribution-normalization
description: Use when single-cell isotope-tracing data (SpaceM spatiomolecular anndata
  objects, per-cell ion intensities for fatty acids like palmitate C16:0 and oleate
  C18:1) must be converted to corrected, normalized isotopologue fractions before
  binomial-fitting uptake estimates.
when_to_use_negative:
- When the input isotopologue table has already been natural-isotope-corrected and
  normalized to fractional abundances (e.g. fit_binomial_prediction.csv or IsoCorrector
  output) — re-normalization risks double-correcting natural-abundance contributions
- When the goal is absolute rather than relative substrate contribution — this normalization
  only supports relative quantification, since the lipogenic acetyl-CoA pool size
  cannot be quantified
- When processing raw imzML for METASPACE-style natural-abundance-only annotations
  — METASPACE isotopologue distributions assume natural isotope abundance and are
  not intended for labeled samples
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: 13C-SpaceM
  role: Repository providing sciso/sc_analysis.py, which performs natural isotope
    abundance correction, per-cell median signal averaging, and isotopologue distribution
    normalization
  repo: https://github.com/Buglakova/13C-SpaceM
- name: Scanpy
  role: Stores and reloads the normalized per-cell isotopologue distributions as pickled
    anndata objects per slide
- name: 13C-binomial
  role: 'Downstream consumer: fit_binomial requires normalized and natural-abundance-corrected
    intensities per isotopologue peak (C16/C18 models)'
  repo: https://github.com/Buglakova/13C_binomial
- name: pyimzML
  role: Parses imzML files to extract ion images for all isotopologues of the fatty
    acids in the molecule list
  repo: https://github.com/alexandrovteam/pyimzML
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:6ba7f3aaf01c999b417317586b417dcfe9d582b55c143233505461a5311faa14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Buglakova/13C-SpaceM
  tools:
  - 13C-SpaceM
  - 13C-binomial
  - Scanpy
  - SpaceM
  - pyimzML
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# isotopologue-distribution-normalization

## Summary

Converts raw per-cell (or per-ablation-mark / per-pixel) fatty-acid isotopologue ion intensities from 13C-SpaceM MALDI-AIF data into fractional isotopologue distributions by first correcting for natural isotope abundance and then normalizing each fatty acid's M+0...M+16 peaks per cell, so that uptake/synthesis fractions can be fitted downstream.

## When to use

Use when single-cell isotope-tracing data (SpaceM spatiomolecular anndata objects, per-cell ion intensities for fatty acids like palmitate C16:0 and oleate C18:1) must be converted to corrected, normalized isotopologue fractions before binomial-fitting uptake estimates.

## When NOT to use

- When the input isotopologue table has already been natural-isotope-corrected and normalized to fractional abundances (e.g. fit_binomial_prediction.csv or IsoCorrector output) — re-normalization risks double-correcting natural-abundance contributions
- When the goal is absolute rather than relative substrate contribution — this normalization only supports relative quantification, since the lipogenic acetyl-CoA pool size cannot be quantified
- When processing raw imzML for METASPACE-style natural-abundance-only annotations — METASPACE isotopologue distributions assume natural isotope abundance and are not intended for labeled samples

## Inputs

- SpaceM spatiomolecular_adata.h5ad / spatiomolecular_matrix.csv per well (output of SpaceM pipeline)
- Per-ablation-mark isotopologue ion intensities for fatty acids extracted from imzML (via extract_isotopologue_ion_images.py and pyimzML)
- Molecule list (e.g. analysis/ion_list_fa.csv) defining the annotated m/z values and fatty-acid isotopologues

## Outputs

- Per-cell normalized isotopologue distributions (fractional abundances M+0 ... M+16) per fatty acid, stored in a slide-level Scanpy anndata object
- Corrected, normalized numpy arrays suitable as input to 13C_binomial fit_binomial (C16/C18 models)

## How to apply

Within the 13C-SpaceM single-cell workflow, sc_analysis.py first applies natural isotope abundance correction for each fatty acid in each ablation mark, then averages signal per cell by taking the median for each annotated m/z value, and finally normalizes the isotopologue distribution for each fatty acid for each cell. Run this per slide (e.g. 20220411_AB_DKFZACLYac slides 1-3) to keep anndata file sizes manageable. The corrected/normalized intensities are the required input format for the downstream binomial model: fit_binomial in 13C_binomial takes a numpy array of normalized and natural-isotope-abundance-corrected intensities for each isotopologue peak (model C16 or C18). Order matters: correction precedes median aggregation, which precedes normalization; skipping the natural-abundance correction would inflate M+1/M+2 peaks from natural isotopes rather than 13C6-glucose-derived label.

## Related tools

- **13C-SpaceM** (Repository providing sciso/sc_analysis.py, which performs natural isotope abundance correction, per-cell median signal averaging, and isotopologue distribution normalization) — https://github.com/Buglakova/13C-SpaceM
- **Scanpy** (Stores and reloads the normalized per-cell isotopologue distributions as pickled anndata objects per slide)
- **13C-binomial** (Downstream consumer: fit_binomial requires normalized and natural-abundance-corrected intensities per isotopologue peak (C16/C18 models)) — https://github.com/Buglakova/13C_binomial
- **pyimzML** (Parses imzML files to extract ion images for all isotopologues of the fatty acids in the molecule list) — https://github.com/alexandrovteam/pyimzML

## Examples

```
python sciso/sc_analysis.py $data_dir/20220411_AB_DKFZACLYac/slide1/spacem_data $data_dir/20220411_AB_DKFZACLYac/slide1/anndata
```

## Evaluation signals

- Downstream fit_binomial runs must succeed and converge (fit_success = 1) — the README states fit_binomial expects normalized, corrected intensities, so uncorrected input typically breaks convergence or inflates M+1/M+2
- Normalized isotopologue fractions for each fatty acid in each cell must sum to 1 across M+0 ... M+16
- The palmitate pseudo-bulk distribution should reproduce the article's reported values, e.g. control M+0 fraction ~0.6 vs bulk LC-MS ~0.3
- Fitted uptake histograms should match reported peaks: palmitate ~0.55 (control) vs ~0.61 (ACLYkd oligo 2); oleate ~0.85 (control) vs ~0.95 (ACLYkd oligo 2)
- Median-based per-cell aggregation must occur after natural-abundance correction and before normalization, per the documented sc_analysis.py step order

## Limitations

- MALDI-AIF only captures fatty acids from lipids effectively ionized in negative mode (primarily acidic glycerophospholipids); abundant species like phosphatidylcholines, triglycerides and sphingomyelins are underrepresented, and uptake proportions may differ by lipid class
- The method yields only relative, not absolute, substrate contributions to the lipogenic acetyl-CoA pool
- Conda environment/version issues with pickled anndata (pandas, anndata, pickle, scanpy combinations) can prevent reading the normalized objects; environment_dump.yml documents the working versions
- Isotopic steady state may not be reached (e.g. 48-h in vivo labelling), affecting interpretation of normalized labelling fractions

## Evidence

- [readme] Natural isotope abundance correction for each fatty acid in each ablation mark: "- Natural isotope abundance correction for each fatty acid in each ablation mark"
- [methods] Normalizing isotopologue distributions for each fatty acid for each cell: "- Normalizing isotopologue distributions for each fatty acid for each cell"
- [readme] fit_binomial takes normalized and corrected intensities: [evidence span withheld — anchor status: external_doc]
- [readme] Per-cell median aggregation of signal: "- Averaging of signal in each cell by taking a median for each annotated m/z value"
