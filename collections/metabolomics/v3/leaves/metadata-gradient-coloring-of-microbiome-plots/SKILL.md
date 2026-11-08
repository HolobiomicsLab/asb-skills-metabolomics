---
name: metadata-gradient-coloring-of-microbiome-plots
description: Use when you have a QIIME principal coordinates file plus a metadata
  mapping file (e.g., thousands of samples with dozens of columns) and need exploratory
  3D ordination plots colored by a continuous metadata gradient or categorical variable.
when_to_use_negative:
- Input is not an ordination — e.g., you have only a raw feature table and no PCoA
  principal coordinates file; run dimensionality reduction (PCoA on beta diversity)
  first
- You need production-grade plotting in a generic language rather than interactive
  microbial-ecology exploration; generic packages like MATLAB, R, Excel or Matplotlib
  are mentioned as alternatives but are not metadata-tailored
- Your only browser is not Chrome or your OS lacks the required command line support
  (OS X 10.6+ and Linux only; Python 2.7, NumPy, BIOM 1.1.0, PyCogent and QIIME python
  libraries required)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EMPeror
  role: Generates the interactive WebGL 3D PCoA plot and provides the gradient-coloring
    and discrete/categorical metadata coloring controllers used to color samples by
    metadata
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Upstream source of the principal coordinates file (PCoA of beta-diversity
    distance matrices); EMPeror depends on the QIIME python libraries only
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:4e52e404565d21214116f938dafcfbf2a0748f93138496cfb40e4044333a099b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - QIIME
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# metadata-gradient-coloring-of-microbiome-plots

## Summary

Color 3D PCoA ordination points by sample metadata — as a continuous gradient (e.g., sample age) or discrete categories — using EMPeror's coloring controllers, so intrinsic patterns such as infant-versus-adult separation become visible in large microbiome datasets.

## When to use

Use when you have a QIIME principal coordinates file plus a metadata mapping file (e.g., thousands of samples with dozens of columns) and need exploratory 3D ordination plots colored by a continuous metadata gradient or categorical variable.

## When NOT to use

- Input is not an ordination — e.g., you have only a raw feature table and no PCoA principal coordinates file; run dimensionality reduction (PCoA on beta diversity) first
- You need production-grade plotting in a generic language rather than interactive microbial-ecology exploration; generic packages like MATLAB, R, Excel or Matplotlib are mentioned as alternatives but are not metadata-tailored
- Your only browser is not Chrome or your OS lacks the required command line support (OS X 10.6+ and Linux only; Python 2.7, NumPy, BIOM 1.1.0, PyCogent and QIIME python libraries required)

## Inputs

- QIIME principal coordinates file (ordination of a beta-diversity distance matrix)
- QIIME metadata mapping file with the columns to color by (e.g., age as a gradient, environment/host category as discrete)

## Outputs

- Interactive WebGL 3D PCoA visualization (HTML5/JavaScript) with gradient- or category-colored points
- Exported rendering image (e.g., figure_1b_age_coloring.png) showing metadata-driven sample separation

## How to apply

Run PCoA on a beta-diversity distance matrix (e.g., with QIIME) to obtain the principal coordinates file, then pair it with the sample metadata mapping file. Run EMPeror's command line interface on both files to generate the interactive WebGL 3D visualization, opened in Chrome. Apply EMPeror's gradient-coloring controller so each point's color is a function of its position along a metadata gradient (e.g., age), or apply the discrete/categorical coloring variant for non-numeric columns. Export a rendering showing the metadata-driven separation (e.g., infant time-series samples versus healthy adult samples) as a final image. This workflow scales to large datasets — EMPeror handles 5,740 samples with 120 metadata columns in a 26 MB file versus 1.85 GB in KiNG.

## Related tools

- **EMPeror** (Generates the interactive WebGL 3D PCoA plot and provides the gradient-coloring and discrete/categorical metadata coloring controllers used to color samples by metadata) — http://emperor.colorado.edu
- **QIIME** (Upstream source of the principal coordinates file (PCoA of beta-diversity distance matrices); EMPeror depends on the QIIME python libraries only)

## Evaluation signals

- The exported plot reproduces the reported result: metadata-integrated coloring clearly shows the age differences between infant samples and samples from healthy human adults (expert visual comparison against published Figure 1B panels)
- The exported output is a loadable PNG file
- Both coloring variants were exercised: gradient coloring (color as a function of position in the age metadata gradient) and the discrete/categorical variant
- Output size check: for 5,740 samples and 120 metadata columns, the EMPeror file is ~26 MB (1.3% of KiNG's 1.85 GB), confirming the skill scales rather than blowing up file size
- The visualization loads in seconds (not minutes as with KiNG) and opens in Chrome without extra plugins

## Limitations

- Command line interface is limited to OS X (10.6 and higher) and Linux only; no Windows support stated
- Requires Python 2.7 (end-of-life), Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent
- Graphical user interface requires the Chrome browser, suggesting untested compatibility with other browsers
- No version number, code repository URL (only project home page http://emperor.colorado.edu), or archival DOI/accession IDs for the example files are provided

## Evidence

- [abstract] EMPeror includes a rich set of controllers to modify features as a function of the metadata.: "EMPeror includes a rich set of controllers to modify features as a function of the metadata."
- [abstract] In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB.: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB"
- [abstract] ability to visualize gradients and categorical data: "ability to visualize gradients and categorical data"
- [discussion] The example files and additional data sets supporting the results of this article are available in the GigaScience Database: "The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25]."
- [discussion] Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent.: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
