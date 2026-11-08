---
name: beta-diversity-pcoa-ordination
description: Use when QIIME beta diversity distance matrices have been ordinated to
  principal coordinates (PCoA) and you need interactive 3D exploration of sample clustering
  colored by metadata, or jackknifed confidence display.
when_to_use_negative:
- Input is raw sequencing data or a feature/Biom table rather than already-computed
  principal coordinates of a beta diversity distance matrix
- The command line interface is needed on Windows; the article states the CLI supports
  OS X (10.6 and higher) and Linux only
- Only generic 2D plotting is needed and no ordination coordinates or metadata mapping
  file exist
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0091
tools:
- name: EMPeror
  role: Renders the PCoA coordinates as an interactive 3D WebGL/HTML5 scene with metadata
    controllers and jackknifed confidence regions
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Upstream framework that generates the beta diversity distance matrices and
    principal coordinates files consumed by the visualization
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:93d230bbca389263d576647ea74cc489b21ba60a84d5659d99ec64f09ac81045
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - PyCogent
  - QIIME
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# beta-diversity-pcoa-ordination

## Summary

Reduces beta diversity distance matrices to principal coordinates (PCoA) and renders the resulting ordination as an interactive, self-contained 3D WebGL/HTML5 visualization via EMPeror, optionally with jackknifed per-sample confidence regions.

## When to use

Use when QIIME beta diversity distance matrices have been ordinated to principal coordinates (PCoA) and you need interactive 3D exploration of sample clustering colored by metadata, or jackknifed confidence display.

## When NOT to use

- Input is raw sequencing data or a feature/Biom table rather than already-computed principal coordinates of a beta diversity distance matrix
- The command line interface is needed on Windows; the article states the CLI supports OS X (10.6 and higher) and Linux only
- Only generic 2D plotting is needed and no ordination coordinates or metadata mapping file exist

## Inputs

- QIIME principal coordinates file (PCoA of a beta diversity distance matrix)
- QIIME metadata mapping file
- jackknifed replicate principal coordinates files (for confidence-region display)

## Outputs

- Self-contained WebGL/HTML5 web page with interactive 3D PCoA sample scatter plot
- Per-sample confidence regions (spheroids/ellipsoids) rendered from jackknifed replicate coordinates
- Metadata-driven UI controllers for visibility, transparency, and recoloring of points

## How to apply

Run principal coordinates analysis (PCoA) on beta diversity distance matrices from microbial community samples to obtain a QIIME principal coordinates file. Feed that PCoA file plus the metadata mapping file to the EMPeror Python command line interface (Python 2.7, QIIME python libraries, NumPy, PyCogent, BIOM 1.1.0). For jackknifed beta diversity, parse the per-replicate PCoA files and aggregate each sample's replicate coordinates so the resample dispersion defines a per-sample confidence region (spheroid/ellipsoid). Use metadata columns to drive coloring and visibility categories, then emit a self-contained web page deliverable whose UI controllers toggle visibility, transparency, and recoloring independent of the command line tool.

## Related tools

- **EMPeror** (Renders the PCoA coordinates as an interactive 3D WebGL/HTML5 scene with metadata controllers and jackknifed confidence regions) — http://emperor.colorado.edu
- **QIIME** (Upstream framework that generates the beta diversity distance matrices and principal coordinates files consumed by the visualization)

## Evaluation signals

- Generated visualization opens in Chrome as a self-contained page and displays samples at their PCoA coordinates without extra plugins
- Jackknifed-resampled data appear as per-sample confidence regions (spheroid/ellipsoid) whose visibility, color, and transparency can be modified from the metadata controllers
- Output file size stays far below legacy KiNG equivalents (paper reports 26 MB vs 1.85 GB for 5,740 samples and 120 metadata columns, i.e. 1.3%)
- Thousands of points load in seconds, not minutes, when the page is opened
- Recoloring/visibility toggles correctly reflect categories from the supplied metadata mapping file

## Limitations

- Command line interface is limited to OS X (10.6 and higher) and Linux; no Windows support stated
- Requires Python 2.7 (an end-of-life Python version), Chrome for the GUI, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent
- No explicit limitations section is given in the paper; confidence regions depend on the availability of jackknifed replicate PCoA files

## Evidence

- [abstract] PCoA dimensionality reduction: "Ordination methods, such as Principal Coordinates Analysis (PCoA) [6] are useful for dimensionality reduction and widely used in different fields to conceptualize distance matrices"
- [abstract] jackknifed clustering confidence: "display jackknifed-resampled data to assess statistical confidence in clustering"
- [discussion] software requirements: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
- [abstract] metadata-driven controllers: "EMPeror includes a rich set of controllers to modify features as a function of the metadata."
